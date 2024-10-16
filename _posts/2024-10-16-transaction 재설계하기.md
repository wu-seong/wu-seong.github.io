---
title: 정보의 일관성을 고려하여 Transaction 재설계하기
date: 2024-10-16 00:00:00 +/- TTTT
categories: [project, 탐식당]
tags: [공부 정리, 트러블 슈팅]	# TAG는 반드시 소문자로 이루어져야함!
---
<style>
  figcaption {
    font-size: 14px;
    color: #555;
    font-style: italic;
  }
</style>

## 서론
이번 글에서는 오류를 해결하면서 문제의 원인을 파악하고 해당 문제를 해결하기 위해서 트랜잭션을 재설계하는 과정을 정리했습니다. 좋은 트랜잭션을 설계하기 위해서 고민한 과정까지 모두 기록했으니 이를 집중해서 봐주시면 감사하겠습니다. 🙂

## 배경
운영중이던 탐식당 어플에서 한번씩 식단 이미지가 제대로 업로드 되지 않는 상황이 종종 일어났습니다.   
원인을 찾기 위해서 서버 로그, DB, S3에 업로드된 이미지를 확인하는 과정에서 문제가 있음을 인지했습니다.


10월 7일 9:46분에 이미지 업로드가 API가 호출되었고
<figure>
    <img src="/assets/img/2024-10-11/img2.png" width="100%" alt="서버 로그">
    <figcaption>서버 로그</figcaption>
</figure>

URL이 무사히 저장되었습니다.
<figure>
    <img src="/assets/img/2024-10-11/img1.png" width="100%" alt="DB">
    <figcaption>DB 이미지 기록</figcaption>
</figure>

하지만 S3를 보니 9:46분에 업로드된 이미지가 없음을 발견했고 
<figure>
    <img src="/assets/img/2024-10-11/img3.png" width="70%" alt="S3">
    <figcaption>S3 이미지 목록(제대로 업로드 되지 않아 몇번 더 시도한 것으로 보인다.. 하지만 그래도 어플에서 이미지가 보이지 않는 것으로 보아 재업로드에도 문제가 있음을 발견..)</figcaption>
</figure>


이미지가 처음부터 업로드 되지 않아서 재업로드시에도 문제가 된건지,   
처음에는 문제가 없었는데 재업로드 과정에서 문제가 생기면서 DB에 수정된 URL이 반영되지 않은건지 확실치는 않지만

**DB와 S3 둘 사이에 정보의 일관성을 잃었다**는 것은 알 수 있었습니다.   
따라서 근본적으로 설계에 문제가 있음을 인지하고 이미지 업로드 과정에서 트랜잭션 단위를 재설계 하고자 했습니다.

## 원래 코드 

### 업로드 

이미지 업로드 시에 호출되는 메서드 입니다.

```java
    @Override
    @Transactional
    public DietPhoto uploadDietPhoto(DietRequestDTO.DietQueryDTO dietQueryDTO, MultipartFile multipartFile) {
        Diet diet = dietQueryService.getDiet(dietQueryDTO.getCafeteriaId(), dietQueryDTO.getLocalDate(), dietQueryDTO.getMeals());
        DietPhoto dietPhoto = DietPhoto.builder()
                .diet(diet)
                .imageKey(uploadImage(multipartFile))
                .build();
        return dietPhotoRepository.save(dietPhoto);
    }
```

식단 정보를 가져와서 참조해서 식단 이미지 객체를 만들어 저장합니다.
객체를 build하는 과정에서 imageKey를 set하기 위해 uploadImage 메서드를 호출합니다.

```java
    public String uploadImage(MultipartFile multipartFile) {
        String fileName = path + "/" + createFileName(multipartFile.getOriginalFilename());

        ObjectMetadata objectMetadata = new ObjectMetadata();
        objectMetadata.setContentLength(multipartFile.getSize());
        objectMetadata.setContentType(multipartFile.getContentType());

        try (InputStream inputStream = multipartFile.getInputStream()) {
            amazonS3Client.putObject(new PutObjectRequest(bucket, fileName, inputStream, objectMetadata)
                    .withCannedAcl(CannedAccessControlList.Private));
        } catch (IOException e) {
            throw new GeneralException(ErrorStatus._INTERNAL_SERVER_ERROR);
        }

        return fileName;
    }
        private String createFileName(String fileName) {
        return UUID.randomUUID().toString().concat(getFileExtension(fileName));
    }

    private String getFileExtension(String fileName) {
        return fileName.substring(fileName.lastIndexOf("."));
    }
```
s3 메서드를 사용하여 실제로 고유한 URL을 생성하고 이미지를 업로드 하는 부분입니다.
randomUUID를 이용하여 고유한 URL을 만들고 Multipart 객체를 S3에 업로드 시키고 실패시에는 일단 예외를 발생시킵니다.   

@Transactional이 붙은 uploadDietPhoto 메서드에서 uploadImage를 호출하고 실패 시에 예외를 발생시키는 것으로 보아 업로드 실패 시에 롤백되어 일관성이 유지될 것으로 보입니다. 

여기서 예외 발생 시에 로그가 남아야 하는데   
로그가 없는 것을 보니 아무래도 첫 업로드 시에는 문제가 없이 동작했던 것 같습니다. (업로드 API가 문제가 아니라 재업로드 API가 문제!!)


### 재업로드

s3를 보면 10:06에 이미지가 업로드 된 것을 볼 수 있는데 이 때 로그를 보니
<figure>
    <img src="/assets/img/2024-10-11/img4.png" width="100%" alt="10:06 로그">
    <figcaption>10:06 서버 로그(업로드와 재업로드 API가 구분이 안된다 🥲 로그에 HTTP 메서드도 기록하도록 변경해야겠다.. )   </figcaption>
</figure>

재업로드 API를 호출한 것 같습니다.
이 부분에서 key가 중복됐다는 SQL 로그를 발견하였고 조금 더 자세히 보기 위해 코드를 보겠습니다.

<figure>
    <img src="/assets/img/2024-10-11/img5.png" width="100%" alt="문제의 재업로드 API">
    <figcaption>문제의 재업로드 API</figcaption>
</figure>

재업로드 API를 보니 벌써부터 잘못된 것이 보입니다.
Delete를 하고 다시 업로드를 하도록 설계했는데 메서드 각각마다 하나의 트랜잭션으로 동작하여 '재업로드' 라는 하나의 단위로 동작하지 못한 것이였습니다. 당시에는 있는 메서드를 재사용하자는 취지에서 저렇게 구현을했는데 이게 데이터의 일관성을 해칠 수 있다는 생각은 못했었던 것 같습니다.   

업로드 동작은 위에서 봤으니 삭제 동작을 한번 보겠습니다.

```java
@Transactional
    public DietPhoto deleteDietPhoto(DietRequestDTO.DietQueryDTO dietQueryDTO) {
        Diet diet = dietQueryService.getDiet(dietQueryDTO.getCafeteriaId(), dietQueryDTO.getLocalDate(), dietQueryDTO.getMeals());
        DietPhoto dietPhoto = dietPhotoRepository.findByDiet(diet);
        deleteImage(dietPhoto.getImageKey());
        dietPhotoRepository.delete(dietPhoto);
        return dietPhoto;
    }
    public void deleteImage(String fileName) {
      amazonS3Client.deleteObject(new DeleteObjectRequest(bucket, fileName));
    }
```

결과적으로 재업로드 API 동작을 간략하게 정리하면

> **트랜잭션1**
1. 식단 및 식단 이미지 조회
2. S3 이미지 삭제
3. dietPhoto 삭제

> **트랜잭션2**
1. 식단 조회 
2. 이미지 업로드
3. DietPhoto 저장

코드 순서상으로는 트랜잭션1 -> 트랜잭션2 이렇게 수행되는 것을 기대했지만
실제로 **하나의 트랜잭션으로 묶여 있지 않기 때문에 DB에서 순서를 보장해주지 않습니다.**

따라서 트랜잭션 2의 DietPhoto 저장 쿼리가 트랜잭션 1의 DietPhoto 삭제보다 먼저 실행될 때,   
OneToOne으로 설정되어 생성된 Unique_key 제약 조건에 위배되기 때문에 나타났던 결과입니다.

따라서 오류의 원인은 재업로드 API에 있었지만, 업로드/재업로드 API 모두 트랜잭션 설계에 문제가 있었기에 둘 다 트랜잭션을 재설계 하기로 했습니다.

## 트랜잭션 재설계

### 업로드 트랜잭션 재설계

#### 1차 설계
<figure>
    <img src="/assets/img/2024-10-11/img7.png" width="50%" alt="업로드 로직 및 복구동작">
    <figcaption>업로드 로직 및 복구동작</figcaption>
</figure>

S3에 **이미지를 업로드**하는 동작과 **DB에** URL정보를 담은 DietPhoto를 **저장**하는 동작을 **하나의 트랜잭션**으로 설계하여,   

만약 이미지 업로드 시에 실패 한다면 로그를 보고 개발자가 직접 처리하고   
DB 저장시에 실패한다면 업로드한 이미지를 삭제하고 로그를 남기고 롤백하도록 했습니다. 

이처럼 동작에 실패하더라도 **정보의 일관성을 유지**하고, 문제의 **원인을 빠르게 찾을 수 있도록** 하였습니다.

또한 이미지 업로드 메서드를 DietPhoto메서드에서 호출하는 것이 아니라 각각을 의존관계가 없는 독립된 메서드로 만들어
**이미지 업로드 동작시에는 불필요하게 DB Connection을 가지지 않도록** 하였습니다.

#### 2차 설계(복구 과정 실패를 고려한)
그런데 사실, 복구 과정역시도 실패할 수 있기 때문에 안정성을 더 높이기 위해서는 이에 고려도 해야합니다.    


보통 실패한다면 트래픽이 몰려 I/O 자원이 부족해서거나, 아예 외부 네트워크 연결이 원할하지 않은 경우라고 가정하여
만약 **복구 로직에 실패 시에 이미지 삭제 리스트를 로컬에 캐싱**해두고 나중에 환경이 안정적일 때 주기적으로 삭제시키도록 설계했습니다.

<figure>
    <img src="/assets/img/2024-10-11/img8.png" width="50%" alt="업로드 로직 및 복구동작(수정)">
    <figcaption>업로드 로직 및 복구동작(수정)</figcaption>
</figure>

#### 3차 설계(순서를 고려한)
그런데 더 고민하다보니 DB저장 로직을 앞에 두는 것이 낫다는 생각이 들었습니다.   
DB 저장 로직을 앞에 둔다면 스프링에서 지원하는 *Transactionl* 어노테이션을 이용하여 더 **간단하게 구현**하고 실제 DB에서 사용하는 **검증된 트랜잭션을 이용**할 수 있어서 제가 직접 롤백 로직을 구현하는 것보다 이점이 많기 때문입니다.

최종적으로 아래의 로직으로 파일 업로드의 트랙잭션을 보장하도록 하였습니다.
<figure>
    <img src="/assets/img/2024-10-11/img10.png" width="50%" alt="업로드 로직 및 복구동작(최종)">
    <figcaption>업로드 로직 및 복구동작(최종)</figcaption>
</figure>

이처럼 동작의 순서에 따라서 롤백 로직이 달라지기 때문에,   
**롤백동작의 구현 난이도, 걸리는 시간, 안정성 등을 고려하여 더 효율적인 롤백 과정을 거치도록 순서를 정해야 한다**는 사실을 알았습니다!



### 재업로드 트랜잭션 재설계

#### 1차 설계
재업로드 로직은 업로드 로직과 유사하지만, 이전 이미지를 삭제한다는 추가적인 로직이 필요합니다.   
따라서 업로드 동작 뒤에 삭제 동작을 붙여 트랜잭션을 설계했습니다. 
<figure>
    <img src="/assets/img/2024-10-11/img11.png" width="50%" alt="재업로드 로직 및 복구동작">
    <figcaption>재업로드 로직 및 복구동작</figcaption>
</figure>

음.. 뭔가 이상하지 않은가요? 🤨    
사실 객체의 URL과 새 이미지를 업로드하는 것에 성공을 하면 재업로드는 어느정도 성공했다고 볼 수 있습니다. 적어도 사용자 관점에서는 말이죠   

여기서 S3 이미지를 삭제하는 동작은 '재업로드'와는 성격이 다르다고 느껴집니다.   
물론, 재업로드 시 이전 이미지는 삭제를 해줘야겠지만 그것이 실패했을 때 이전 과정을 롤백할 만큼 중요할까요?

#### 2차 설계
저는 아니라고 판단했기 때문에 S3 이전 이미지를 삭제하는 것은 트랜잭션에 포함시키지 않았습니다.  
대신 이전에 설계했던 것 처럼 실패 시에 이미지 삭제 리스트에 추가하도록 했습니다.

<figure>
    <img src="/assets/img/2024-10-11/img12.png" width="50%" alt="재업로드 로직 및 복구동작(최종)">
    <figcaption>재업로드 로직 및 복구동작(최종)</figcaption>
</figure>

트랜잭션에 어떤 동작을 포함시킨다는 것은 그만큼 트랜잭션을 무겁게 하기 때문에 트랜잭션의 성공 가능성 역시 낮추기에 **정말 필요한 최소한의 동작만으로 트랜잭션을 구성해야 한다**는 사실을 다시 한번 체감했습니다!

---
## 마무리
이로써 업로드와 재업로드 동작에 대해 트랜잭션을 설계하는 과정을 마쳤습니다. 다음 글에서는 설계한 트랜잭션을 코드로서 구현하는 과정을 이어서 진행하도록 하겠습니다. 지금까지 긴 글 읽어주셔서 감사합니다 ☺️

## 참고자료

책 - RealMySQL   
[티스토리 - AWS S3 이미지 저장 및 삭제와 DB로직 트랜잭션 분리](https://jinmook.tistory.com/24)