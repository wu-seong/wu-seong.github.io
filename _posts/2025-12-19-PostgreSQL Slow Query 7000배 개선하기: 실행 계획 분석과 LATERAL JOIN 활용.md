---
title: "Swagger 작동원리 모르면 들어오세요"
date: 2025-01-17 00:00:00 +/- TTTT
slug: swagger-internals
categories: [백엔드, 트러블슈팅]
tags: [postgresql, performance]	# TAG는 반드시 소문자로 이루어져야함!
image: /assets/img/2025-12-19/img0.png
published: false

---
<style>
  figcaption {
    font-size: 14px;
    color: #555;
    font-style: italic;
  }
</style>


---

> *Swagger 문서화 과정에서 겪은 문제해결 과정과 인사이트를 공유합니다*

---

## 배경

프로젝트를 진행하던 중 
온보딩 과정 중 에러 처리가 제대로 되지 않는 부분이 있었다.

백엔드에서 에러 케이스를 예상하고 대응했지만, 프론트쪽에 공유가 되지 않은 것
Swagger를 사용하여 API 명세에 대한 부분을 공유하고 있었지만, 성공 케이스에 대한 부분만 공유가 되고 있었고 에러 케이스는 직접 전달을 해야하는 구조였다.(이런 방식이다 보면 놓치는 부분이 분명 생길 것)
이런 상황 속에서 예외 처리에 대한 명세 자동화의 필요성을 느꼈다.


기준 구조 ~
Claude 추천 구조 기반으로 시도
Swagger에 대한 원리? 설명

어노테이션에 대한 문제 겪음
메타 어노테이션? 으로 해결

어노테이션에 대한 재고찰
JLS 규칙
어노테이션이 인터페이스가 불가능한 이유
"형태"가 아닌 "값"을 저장하기 때문
그저 메타데이터임

[ .class file ]
      ↓ (1회)
[ JVM Metaspace ]
      ↓                ↘
 getAnnotation()      Spring scan
      ↓                ↓
[ Annotation Proxy ]  [ 해석 결과 캐시 ]



다형성 불가 -> 인터페이스로 다형성에 대한 처리 불가
컴파일 시점의 메타데이터에 불가함
다시 생각해보면 문서화라는 것 자체가 컴파일 시점에 결정되지, 런타임에 결정되는 것이 아니기 때문에
어찌보면 당연하고 더 맞는 것임 Errorcode로 받으면 MembeController에 막 다른 에러가 가능

## 참고자료
[Swagger의 사실과 오해: API-First Development](https://dev.to/headf1rst/swaggeryi-sasilgwa-ohae-api-first-development-3kcc)