---
title: 운영체제 정리 - OS Structures
date: 2023-06-24 00:00:00 +/- TTTT
categories: [CS, 운영체제]
tags: [공부 정리]		# TAG는 반드시 소문자로 이루어져야함!
---


-------
## 운영체제 서비스
![](https://velog.velcdn.com/images/jws1228/post/38431adb-2e96-41af-8618-ecb24be717a1/image.png)

- 운영체제는 **system call**을 통해 프로그램과 유저에게 service를 제공한다

### 서비스 종류
for 유저 - UI	제공, 프로그램 실행, I/O 동작, 파일 시스템 관리, 소통(between process), 에러 감지

for OS(itself) - 자원 관리, 기록, 보호 및 보안 

-----

## System call

* System call은 유저에게 서비스를 사용할 인터페이스를 제공한다
* 이때 직접 System call을 호출하도록 하는 것이 아니라 API를 제공하여 간접적으로 호출하게 한다.
![](https://velog.velcdn.com/images/jws1228/post/ba692d52-eaef-4a05-b1f3-c9ed4d5e4495/image.png)
위는 유닉스와 리눅스에서 이용되는 헤더 파일의 API의 예시이다 

![](https://velog.velcdn.com/images/jws1228/post/26c1417a-9a9d-4cd4-a7c6-e11f87215072/image.png)

운영체제 서비스를 이용하는 과정을 정리하자면 

1.어떤 응용프로그램을 사용할 때, 유저는 RTE(Run-Time Environment)에서 제공하는 API를 이용하여 system call을 간접호출 하고
2. 커널에서는 이 호출에 따른 적절한 동작을 수행한 후 반환 값을 다시 돌려준다.
3. 유저는 다시 API를 통해 반환 값을 얻을 수 있다.

### System call 파라미터 전달
- 가장 기본적으로 **register**를 통해 전달된다
- **메모리**에 블럭 혹은 테이블로 **저장**되고 이 **주소**를 **가르키는 register**로 전달되기도 한다
- **stack공간**을 통해 push와 pop을 이용하여 전달하기도 한다

-----

## 링커와 로더


링커와 로더의 역할을 알기 위해서는 우리가 작성하는 소스코드가 어떻게 실행되는지까지를 알면 좋다.

<image src="https://velog.velcdn.com/images/jws1228/post/44d9567f-e4e2-4f3b-ad74-75ccb74aafde/image.png" width="60%" height="60%" ></image>

1. 우리가 작성한 소스코드가 컴파일러에 의해서 이진 바이너리 파일로(relocatable object file) 바뀐다.
2. 링커는 이 파일들을 하나의 실행가능한 파일로 합친다.
3. 로더는 이 파일을 메모리에 올려 cpu가 작업할 수 있도록한다.(이때, DLL이 같이 메모리에 올라가 실행되기도 한다)

#### DLL(Dynamically linked library)
- 동적으로 로드되는 라이브러리 실행파일(executable file)
- 동적으로 연결되어 사용될 수 있으며, 실행파일이기 때문에 절대 주소를 가짐

예시) Windows API DLL, 그래픽 라이브러리 DLL, 데이터베이스 접근 DLL..


-------

## OS 구조

### Monolithic 구조

<image src="https://velog.velcdn.com/images/jws1228/post/1772bb3e-c7cf-485d-bb7d-44047b860aa1/image.png" width="40%" height="40%" alt="Linux monolithic 구조예시" style="margin:0px auto"></image> <figcaption style="text-align:center; font-size:15px; color:#808080">리눅스 monolithic 구조 예시</figcaption>
- 모든 기능이 하나의 파일에 결합되어 있음
- 속도는 빠르나, 구현이 어렵고 확장성이 떨어짐
  
### 계층적(Layrered) 구조
<p style="display:block; width:200px; height:200px; margin:0px auto;">
<image src="https://velog.velcdn.com/images/jws1228/post/0a1da3ea-463e-4aae-8ab4-f10bfd4193b5/image.png"></image><figcaption style="text-align:center; font-size:15px; color:#808080">OS 계층적 구조 </figcaption>
  </p>

- 계층적으로 이루어진 구조, 가장 바깥은 **UI** 가장 안쪽은 **hardware**
- 구조가 단순해지고 사용자는 최상위의 UI만 다루면 됨
- 다단계 구조로 인한 비효율이 발생할 수 있음
  
### Microkernel 구조
  
<image src="https://velog.velcdn.com/images/jws1228/post/4307beb2-84c0-428d-a752-2c95d77a7292/image.png"  width="60%" height="60%"></image>
  <figcaption style="text-align:center; font-size:15px; color:#808080">OS microkernel 구조 </figcaption>

- 가장 핵심적인 기능만 커널에 남기고 나머지는 user-level에서 구현하도록 함
- 구조가 유연하고 보안에 강하다
- 사용자와 커널간 통신이 잦기 때문에 이에 따른 성능 저하 있을 수도 있음
  
### Module 구조

- 커널에 핵심 요소가 있고 나머지 요소는 동적으로 연결 될 수 있음(부팅이나, 실행 시)
- 계층 구조와 비슷하지만 더 유연함(어떤 모듈이든 호출가능)
- microkernel 구조와 비슷하지만 통신지연이 없어 더 빠름
- 현재 많은 OS가 이 구조를 채택함