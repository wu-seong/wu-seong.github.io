---
title: "삭제를 Delete 쿼리 한 줄로 끝낼 수 없는 이유"
date: 2026-04-22 00:00:00 +/- TTTT
categories: [백엔드]
tags: [jpa, spring, database, cascade]	# TAG는 반드시 소문자로 이루어져야함!
image: /assets/img/2026-04-22/thumbnail.png
published: true

---
<style>
  figcaption {
    font-size: 14px;
    color: #555;
    font-style: italic;
  }
</style>

---

> *삭제는 단순히 DELETE 쿼리 하나가 아니다. 복구 가능성, 연쇄 삭제, 구현 방법 선택까지 — 데이터를 지운다는 것의 의미를 설계 관점에서 정리했다.*

---

## 서론

<figure>
    <img src="/assets/img/2026-04-22/img1.png" width="50%" alt="soft delete vs hard delete">
    <figcaption></figcaption>
</figure>

특정 서비스에서 삭제 기능을 넣는다고 하면 필연적으로 고려해야할 것들이 있다.

- 첫번째로는, 이 삭제의 **복구 가능성**을 염두에 둘 것인가? (softDelete vs HardDelete)
- 두번째로는, 삭제 후에 이 정보를 **참조**하고 있던 곳에선 어떻게 처리를 할 것인가?

이외에도 복구 가능성을 둔다면 복구 가능 기간을 최대 며칠로 둘 것인지? 여러 고민점이 생길 것이다.

이번 글에서는 프로젝트를 진행하면서 이러한 삭제에 대한 고민을 마주했던 상황에 대해서 얘기하고,
어떤 방식으로의 삭제가 각 상황에 적절한지를 살펴볼 것이다.

---

## 개념

### 복구 가능성 - Soft Delete vs Hard Delete

복구 가능성을 어떻게 구현할 지에 대해서 간단히 살펴보자.
복구 가능성을 위해선 **물리적**으로 데이터를 지워선 안된다.

<figure>
    <img src="/assets/img/2026-04-22/img2.png" width="100%" alt="soft delete vs hard delete">
    <figcaption></figcaption>
</figure>

물리적으로 데이터를 지운다는 건 보통 저장된 데이터들이 장기 저장 장치(SSD, HDD)에 저장되어 있는데,
휴지통을 비운다던지, DB에 Delete 쿼리를 날린다던지 하여 실제 물리적인 저장 장치에서 삭제되는 것이다.
일반적으로 이런 경우에는 **복구가 불가능**하다. 이 경우를 **Hard Delete** 라고 한다.

<figure>
    <img src="/assets/img/2026-04-22/img3.png" width="100%" alt="soft delete vs hard delete">
    <figcaption>soft delete는 deleted_at만 찍고 row를 남기기 때문에, DB CASCADE(hard delete)와 충돌한다</figcaption>
</figure>

따라서 우리는 데이터를 **논리적**으로 삭제해야한다.
간단하게 말해서 '삭제됨' 꼬리표를 정보에 추가하는거다.
그러면 우리는 이 정보는 삭제됐다고 취급을 하고 조회 시에 꼬리표가 붙은 정보는 없는 정보 취급을 함으로서 실제로 삭제가 된 것과 동일하게 느낄 수 있다.

이 경우를 **Soft Delete**라고 한다.

---

### 삭제된 데이터의 참조 정책

삭제된 데이터를 참조하고 있어서 보여줘야하는 경우들이 있다.
대표적인 예로 댓글/대댓글 기능에서 댓글이 삭제 된다고 하더라도 대댓글에서는 맥락 파악을 위해 댓글을 참조한다.

1. 이 때 삭제된 댓글을 어떻게 표시할 지?
2. 혹은 댓글 삭제 시 하위 대댓글도 모두 삭제할 지?

삭제를 실제 처리하는 관점에서 본다면, **어플리케이션 로직**에서 이러한 정책들을 구현할 수 있고
혹은, **DB 레이어**에서 이러한 정책을 그대로 따를 수 있는 규칙을 추가할 수 있다.

이러한 부분들을 정책적으로 결정해야한다.

---

### 실제로는 어떻게?

앞서 말했던 복구 가능성과 삭제된 데이터에 대한 참조 정책은 동시에 고려해야한다.
복구 가능성을 염두에 두면서 데이터 삭제 시에 이를 외부에서 어떻게 처리할 지를 같이 고민해야하는 것이다.

아래에선 실제 프로젝트를 진행하면서 어떤 고민을하고 어떤 결정을 내렸는지를 설명할 것이다.

---

## 프로젝트 설계 결정

<figure>
    <img src="/assets/img/2026-04-22/img5.png" width="100%" alt="soft delete vs hard delete">
    <figcaption></figcaption>
</figure>

### 기본 삭제 정책

기본적으로 휘발적인 정보를 제외하면 우리 프로젝트에서는 이후 복구 가능성이나 추적을 위해 모든 엔티티에 소프트 딜리트를 적용하기로 했다.

기본적으로 Hard Delete를 하고 필요한 부분에 대해서만 Soft Delete를 하는 것은 어떨까 싶었지만,
Soft Delete와 Hard Delete가 섞이면 오히려 관리가 힘들어질 수 있다는 우려와 운영을 하면서 모든 컨텐츠에 대해 복구 가능 혹은 추적이 있는 것이 좋을 것 같다는 의견이 있어서 해당 정책으로 결정됐다.

---

### 유저 삭제시 동작

가장 중요한 유저 삭제 작업을 설계해보려고 한다.
유저는 보통 서비스의 대부분의 컨텐츠에 참여를 하며, 유저가 삭제될 시에 이 유저가 생성했던 컨텐츠를 어떻게 할 지 결정한다.

예를 들어 유저가 생성한 게시글이 있다면 이 게시글도 모두 삭제할 것인지?
혹은 삭제하지 않는다면 게시글에서 작성자 정보를 보여줄 때는 어떻게 나타낼지?

우리는 기본적으로 소프트 딜리트를 적용하여 필요 시 복구가 가능하기 때문에, 유저 삭제 시 **연관된 모든 정보를 삭제**하기로 했다. (이 글에서는 1뎁스의 연쇄삭제만 고려한다)

상위 데이터의 삭제가 하위 데이터까지 연쇄적으로 영향을 미쳐 모두 함께 삭제되는 모습을 폭포에 비유하여 **cascade**라고 한다.

---

### cascade 작업

이 cascade 작업을 구현하는 데에는 크게 3가지 방법이 있다.

1. DBMS On Delete 제약조건
2. JPA @OneToMany CascadeType.REMOVE 옵션
3. 어플리케이션에서 Cascade 직접 구현

각각 상황에 따라 쓰임이 조금씩 다르고 장단점이 있다.

| 비교 항목 | DBMS ON DELETE CASCADE | JPA CascadeType.REMOVE | 애플리케이션 직접 구현 |
|---|---|---|---|
| **동작 위치** | DB 엔진 (DDL 레벨) | Hibernate (EntityManager) | Service / Repository 레이어 |
| **트랜잭션 참여** | DB 내부 트랜잭션 자동 처리 | JPA 트랜잭션 내 처리 | 명시적 `@Transactional` 필요 |
| **정합성** | DB가 보장, 누락 불가 | 영속성 컨텍스트 flush 시점에 의존 | 구현 실수 시 정합성 깨짐 위험 |
| **성능** | 가장 빠름 (DB 내부 처리) | N+1 SELECT 후 DELETE 발생 가능 | 배치 처리로 최적화 가능 |
| **소프트 딜리트 호환** | ❌ 불가 (물리 삭제만) | △ 자식에도 `@SQLDelete` 달면 가능, 단 PC 로드 보장 필요 | ✅ 가능 (`deleted_at` 직접 세팅) |
| **부모만 삭제 시** | 자식 자동 삭제 | 자식 자동 삭제 (PC에 로드된 경우) | 명시적으로 자식 먼저 처리 |
| **orphanRemoval 연동** | 무관 | `orphanRemoval=true` 함께 쓰면 관계 해제도 커버 | 무관 |
| **Lazy 로딩 주의** | 없음 | 자식이 PC에 없으면 삭제 누락 가능 ⚠️ | 직접 조회하므로 무관 |
| **이벤트/훅 발생** | ❌ JPA `@PreRemove` 미발생 | ✅ `@PreRemove` 발생 | ✅ 원하는 시점에 이벤트 발행 가능 |
| **유지보수성** | 스키마 변경 시 DDL 추적 필요 | 엔티티 코드로 추적 가능 | 로직 분산 가능, 테스트 용이 |
| **감사 로그 (Audit)** | ❌ 어려움 | △ `@PreRemove`로 가능 | ✅ 자유롭게 처리 가능 |
| **멀티 DB 이식성** | ❌ DB 벤더 의존 | ✅ 벤더 독립 | ✅ 벤더 독립 |
| **주요 적합 케이스** | 소프트 딜리트 없는 단순 도메인, 데이터 무결성 최우선 | 소규모 연관관계, 빠른 개발 | 소프트 딜리트, 복잡한 비즈니스 로직, 이벤트 발행 필요 시 |

우리는 어차피 소프트 딜리트 정책을 전반적으로 가져가기로 했으니, 소프트 딜리트 호환 부분만 뽑아서 보면:

| 비교 항목 | DBMS ON DELETE CASCADE | JPA CascadeType.REMOVE | 애플리케이션 직접 구현 |
|---|---|---|---|
| **소프트 딜리트 호환** | ❌ 불가 (물리 삭제만) | △ 자식에도 `@SQLDelete` 달면 가능, 단 PC 로드 보장 필요 | ✅ 가능 (`deleted_at` 직접 세팅) |

소프트 딜리트가 가능한 JPA Cascade 옵션, 직접 구현 두 방법 중 선택해야한다.
사실 부모에서 Cascade를 사용한다는것 자체가 사실상 **양방향**을 적용하는 것이기 때문에 (단방향 `@OneToMany`는 별도의 단점이 많아 실무에서 거의 사용하지 않으므로 고려하지 않음),
현재 프로젝트에선 양방향 매핑을 지양하고 있어 현재 기준으로는 **직접 구현**하는 선택지 밖에 없긴 하다.

그럼에도 cascade 옵션을 활용하는 방안을 한번 더 살펴보자면, 사용을 위한 몇 가지 조건이 있다.

1. 부모 엔티티에 `@OneToMany`/`@OneToOne` 양방향 매핑을 해야한다.
2. 양방향 매핑에 `cascade.remove` 옵션을 적용해야한다.
3. 삭제 시점에 자식이 Load되어 있음을 보장해야한다. (Eager loading을 쓰거나 Lazy Loading시 이를 철저히 관리)
4. 자식 엔티티에 `@SQLDelete` 옵션을 적용해야한다.

<figure>
    <img src="/assets/img/2026-04-22/img4.svg" width="100%" alt="soft delete vs hard delete">
    <figcaption>JPA 동작 구조(feat.claude)</figcaption>
</figure>

이렇게 양방향 매핑을 기반으로한 기술이기 때문에 자체에 고려할 점이 많아진다.
결국 우린 수동으로 작업을 해야겠다.

다음 글에서는 실제로 이러한 설계를 프로젝트에 어떻게 적용시켰는지를 살펴볼 것이다.

---

## 정리

삭제 기능을 설계할 때는 **복구 가능성**과 **참조 정책** 두 가지를 함께 고려해야 한다.

복구 가능성을 염두에 둔다면 물리적으로 데이터를 지우는 Hard Delete 대신, 논리적으로만 삭제 표시를 남기는 Soft Delete를 선택해야 한다.

연쇄 삭제(cascade)를 구현하는 방법은 세 가지가 있다.

| 방법 | 소프트 딜리트 호환 | 특징 |
|---|---|---|
| DBMS ON DELETE CASCADE | ❌ | 성능 최우선, 물리 삭제만 가능 |
| JPA CascadeType.REMOVE | △ | 양방향 매핑 + `@SQLDelete` 조합 필요 |
| 애플리케이션 직접 구현 | ✅ | 유연하지만 정합성은 직접 보장해야 함 |

소프트 딜리트 정책을 전체 적용하는 경우, DBMS ON DELETE CASCADE는 물리 삭제만 지원하기 때문에 사용할 수 없다. JPA Cascade는 양방향 매핑을 전제로 하기 때문에 단방향 매핑만 사용하는 프로젝트라면 결국 **애플리케이션에서 직접 구현**하는 것이 현실적인 선택이다.

cascade 방법을 선택할 때는 단순히 편의성만 볼 것이 아니라, 프로젝트의 **삭제 정책**과 **연관관계 매핑 방향**을 먼저 파악한 뒤 결정해야 한다.

---

## 참고자료

- [Baeldung - JPA Cascade Types](https://www.baeldung.com/jpa-cascade-types#2-cascadetypepersist)
- [Thorben Janssen - Avoid CascadeType.REMOVE on Many Associations](https://thorben-janssen.com/avoid-cascadetype-delete-many-assocations/)
- [codestudy - orphanRemoval vs ON DELETE CASCADE](https://www.codestudy.net/blog/how-does-jpa-orphanremoval-true-differ-from-the-on-delete-cascade-dml-clause/)
- [w3tutorials - JPA Unidirectional ManyToOne and Cascading Delete](https://www.w3tutorials.net/blog/jpa-unidirectional-many-to-one-and-cascading-delete/)
- [gilssang97 - JPA Cascade vs DB ON DELETE CASCADE](https://gilssang97.tistory.com/71)
