---
title: 운영체제 정리 - Synchronization Tool
date: 2023-07-08 00:00:00 +/- TTTT
categories: [CS, 운영체제]
tags: [공부 정리]		# TAG는 반드시 소문자로 이루어져야함!
---

---

## Background
\- 데이터를 concurent하게 처리하면서 공유되는 데이터에 동시에 접근하게 되는데 이때, 데이터의 **불일치**가 나타날 수 있다.

=> 이를 해결하기 위해서 실행 순서를 정해 **배타적으로 접근**하는 것이 필요하다.
<br><br>
### Race Condition
\- Producer-Consumer Problem에서 Producer는** counter++**을 하고 Consumer는 **counter--**를 하여 동시에 counter에 접근하여 값을 변경시킬 수 있는데 이때 **데이터의 불일치**가 나타난다

![](https://velog.velcdn.com/images/jws1228/post/b55d2ca8-9412-4f7d-9407-7d46d941d79d/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">counter++의 동작</p>

![](https://velog.velcdn.com/images/jws1228/post/0787470b-20c8-4bb2-a715-c899d32d500a/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">counter--의 동작</p>

![](https://velog.velcdn.com/images/jws1228/post/e2570777-68d6-404f-99bb-78d44d32b4f6/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">동시에 일어났을 때의 데이터 불일치 문제</p>

=> 연산의 **원자성**이 보장되지 않기 때문에 생김
<br><br>

---
## Critical Section 문제

**critical section**: shared data가 존재하는 코드 부분

**critical section problem**: 여러 프로세스 또는 스레드가 shared data에 동시에 접근하는 상황에서의 동기화 문제

### Solution to Critical-Section Problem
\- 다음 세가지를 만족함으로써 Critical Section문제를 해결할 수 있다.

**Mutual Exclusion(상호 배제)**: 동시에 critical section을 진입할 수 없음



**Progress(진행)**: 필요없는 지연없이 critical section이 비었다면 진입하는 것

**Bounded Waiting(한정된 대기)**: 무한정 대기 하지 않는 것, 진입 요청 사이에 다른 프로세스가 진입하는 횟수에 대한 제한
<br><br>

---
## Peterson's Solution
- 고전적인 Critical-section 문제 해결방법
- flag[]: 진입 요청 플래그
- turn: 현재 진입할 프로세스

![](https://velog.velcdn.com/images/jws1228/post/1caecad8-0332-4dce-8502-140a62ba457f/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">Peterson's Solution</p>

- 요청을 하고 차례를 양보한다
- 상대가 진입 요청을 하지 않았거나 턴이 끝났거나 혹은, 상대가 다시 양보를 하면 critical section에 진입
- critical section 나올 때는 반드시 flag를 거두어 progress되도록 한다.

### 문제점
\- 현대 컴퓨터에서는 성능을 위해 코드의 순서가 바뀔 수 있기 때문에 동시에 진입하여 **mutual exclusion이 보장되지 않는다.**
![](https://velog.velcdn.com/images/jws1228/post/68ae187b-8569-4822-9900-db6e3b7a0143/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">Peterson's Solution의 순서가 바뀐 예시: cs에 동시에 진입</p>
<br><br>

---
## Hardware Support for Synchronization


\- 앞에서 봤듯 코드의 현대의 컴퓨터는 순서가 보장되지 않기 때문에 **이 순서를 보장해줄 3가지 하드웨어적인 해결방법**이 있다.

### Memory Barries
![](https://velog.velcdn.com/images/jws1228/post/68c4d5d0-bd7f-4386-8435-bfd25e7a6fae/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">Memory Barries</p>

\- 명령어를 이용하여 명령어 기준 이후의 코드가 이전의 코드보다 나중에 실행되도록 보장
<br><br>
### Hardware Instruction

\- 하드웨어의 지원으로 원자적으로 실행되도록 하는 lock 명령어들
#### test_and_set(TAS)
![](https://velog.velcdn.com/images/jws1228/post/d21901ce-98a1-4cf3-8efc-a12606259e49/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">TAS 코드</p>

- target이 false 일때 들어오면서 target= true로 바꿈 반환은 false
- target이 true라면 true를 유지

![](https://velog.velcdn.com/images/jws1228/post/631af6bc-f10f-49ce-bef4-8e2ab811f3ed/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">TAS를 이용한 Lock</p>

- lock이 True일 때는 True가 유지되어 CS에 진입을 못함
- lock이 false가 되는 순간 한번만 false를 반환화면서 CS에 진입
- 그 다음부터는 다시 True가 유지 되어 lock


#### compare_and_swap(CAS)

![](https://velog.velcdn.com/images/jws1228/post/79587c48-0df9-4f02-ae5a-b4f6fbe161f2/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">CAS 코드</p>

- TAS 유사하지만 target이 boolean이 아니라 int임
![](https://velog.velcdn.com/images/jws1228/post/088aa704-e726-4613-ba7c-04bead77b8b9/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">CAS를 이용한 Lock</p>
- lock이 not expect이면 CS에 진입을 못함
- lock이 expect가 되는 순간 한번만 expect를 반환하면서 조건문이 false가 되어 CS에 진입
- 그 다음부터는 다시 not expect가 유지 되어 lock

<br><br>
### Atomic Variables


![](https://velog.velcdn.com/images/jws1228/post/8cdf951a-b817-4cf8-8e4b-5ffc354cfc38/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">Atomic variable</p>

- Atomic한 연산을 제공하는 변수, 자바의 integer, boolean도 아토믹 변수임
- 내부적으로 CAS로 구현되어 있는 경우가 많음
<br><br>
---

## Mutext Locks
- 앞의 하드웨어 기반 솔루션을 **응용프로그램 개발자가 직접 사용하기에는 복잡하고 어려움**
- 그래서 간편하게 사용할 수 있는 것이 **mutex(mutual exclusion) lock**
- **acquire()과 release()** 명령어를 사용하여 lock을 제어한다
- 이 명령어들은 **atomic**해야하고 보통 내부적으로 **CAS**로 구현되어있다

![](https://velog.velcdn.com/images/jws1228/post/6f4695c2-cd84-42db-bf2e-ef09eb88e2ad/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">acquire, release 코드</p>
<br><br>

---

## Semaphore
- mutex lock보다 더 정교한 방법을 제공하는 동기화 도구 
- integer 변수이고 오직 아래 두 명령어를 이용하여 접근할 수 잇다
- wait()와 signal() 명령어를 사용하여 제어

![](https://velog.velcdn.com/images/jws1228/post/0e4cc154-19ba-48ab-887e-e0a9d89775b7/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">wait와 signal 코드</p>

**Counting Semaphore**: 모든 범위의 정수를 사용, 여러 프로세스가 접근할 때 유용
**Binary Semaphore**: 0과1만 사용, mutex lock과 같다.
<br><br>

### with no busy waiting
- 원래 waiting을 하면서 조건을 계속해서 확인해야 했었다.
- 프로세스의 **waiting list**를 활용하여 busy waiting을 없앨 수 있다.
- sleep()과 wakeup() 명령을 통해 waiting queue와 ready queue에 넣고 뺄 수 있다.
- **critical section이 짧다면** **busy waiting**도 길지 않기 때문에 **고려해볼 옵션**이다, 오히려 대기열을 사용하지 않아 더 효율적일 수 있다.

![](https://velog.velcdn.com/images/jws1228/post/e2687cdd-59b3-489b-92a1-33b73c14a3de/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">wait 코드 with no Busy Waiting </p>

\- 먼저 value 값을 줄이고 0보다 작으면 waiting queue에 둠(sleep)

![](https://velog.velcdn.com/images/jws1228/post/ad303d2c-a4c7-4865-95f5-602bfdca1df9/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">signal 코드 with no Busy Waiting </p>

\- 먼저 value 값을 늘리고 0 이하이면 waiting queue에서 ready queue로 이동(wakeup)
<br><br>

### 주의점
- 하나의 semaphore에 대해서 한번의 wait()와 signal이 실행되어야한다.
- 따라서, wait와 signal 코드역시 critical section에 놓여야한다
<br><br>
---

## Monitors
\- 앞의 주의점을 지키지 않고 사용한다면 error가 발생할 수 있기 때문에 이를 관리하기 위해 고급언어에서 지원하는 도구

![](https://velog.velcdn.com/images/jws1228/post/6684583b-8f7a-4570-9bd3-3de7ca41d7f7/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">wait와 signal을 잘못 사용한 예시</p>

![](https://velog.velcdn.com/images/jws1228/post/244bf49d-e021-4961-8480-e68c6f3b5c99/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">Monitor 구조</p>

- 모니터 타입내에 변수와 함수를 선언한다
- 변수는 오직 함수를 통해서만 접근가능하다
- 오직 하나의 프로세스만 실행되는 것이 보장된다
<br><br>
### Condition Variables
![](https://velog.velcdn.com/images/jws1228/post/012a0159-f4de-4676-b2fb-c2c8c68f6fa4/image.png)

\- 조건변수를 설명하기 위해 producer-consumer를 예시로 들자면

**condition x**: if count == 0 // 소비자가 먼저 올 때의 조건변수

x.wait() => 소비자가 먼저 진입하여 조건변수에 의해 내부에서 Monitor 내부에서 대기하게 됨
x.signal() => 생산자가 진입하여 내부에서 대기하는 소비자가 있으면 실행되게 함
<br><br>

### Semaphore를 이용한 Monitor 구현
![](https://velog.velcdn.com/images/jws1228/post/feeca3f4-18d1-4e09-8ced-c66a9016d1ad/image.png)

<p align="center" style=" font-size: 12px;
color: #999;">Monitor의 함수 내부 구현</p>

![](https://velog.velcdn.com/images/jws1228/post/e4e4d8c1-ab66-4cf5-b438-eb96f4c15a16/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">Monitor의 condition wait와 signal구현</p>


- 해당 구현은 **Signal and Wait** 방법을 채택한 것이다
- Monitor 내부 queue에서 FCFS 방법을 사용할 수도 있지만 가중치를 부여하여 구현하여 priority방법을 구현할 수도 있다.


---
실제로 자바에서는 'Monitor 인터페이스를 상속받아서 오버라이딩 하는건가?' 라는 생각이 들어 추가적으로 찾아보니

실제로는 최상위 클래스인 Object클래스에 **synchronized 키워드**를 사용하여  내부적으로 모니터 또는 락(lock)을 사용하여 동기화를 구현한다고 한다.

또한, **wait(), notify(), notifyAll()과 같은 메서드**들을 사용하여 조건 변수를 활용할 수도 있다고 한다. notify는 앞의 signal과 같다.