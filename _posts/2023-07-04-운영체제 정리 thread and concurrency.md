---
title: 운영체제 정리 - Thread&Concurrency
date: 2023-07-04 00:00:00 +/- TTTT
categories: [CS, 운영체제]
tags: [공부 정리]		# TAG는 반드시 소문자로 이루어져야함!
---

## Overview

### Thread란?

- cpu를 사용하는 기본단위이다.  
<br>
  
  

### Thread 구성요소
- thread ID
- program counter(PC)
- register set
- stack

### 공유하는 부분
- code section
- data section
- OS resources(open files and signals)
![싱글쓰레드와 멀티 스레드 구조](https://velog.velcdn.com/images/jws1228/post/924742cf-f557-453c-9a01-443762f77840/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">싱글쓰레드와 멀티 스레드 구조</p>


- 현대의 대부분의 프로그램은 멀티스레드 프로그램이다
- 스레드가 프로세스에 비하여 훨씬 가볍다(light-weight)

### 스레드의 장점
- 반응이 빨라진다(block되지 않으므로)
- 공유하는 부분이 정해져 있으므로 자원을 쉽게 공유한다.
- 더 light하므로 context switch가 더 부담이 없다
- multiprocessor 구조를 이용하므로 필요하면 칩을 추가하여 확장할 수 있다

---
## Multicore Programming

\- multihreaded programming은 multicore나 multiprocessor 활용하여 동시성을 높인다(improve concurrency)

#### concurrecny vs parallelism
![concurrecny vs parallelism](https://velog.velcdn.com/images/jws1228/post/e7b66ea3-5fed-46ef-870d-d7e0e994d0ec/image.png)


**concurrency**: 여러개의 일이 동시에 진행되는 것

**parallelism**: 실제적으로 두 부분에서 일이 진행 되는 것

-> **싱글 코어**에서는 **concurrent**하기만 하고 **멀티 코어/멀티 프로세서에서**는 **concurrent**하고 **parallel**한 것

### Programming의 어려움

- multi core/processor 시스템을 고려하여 프로그래밍을 하는 데는 다음과 같은 어려움이 있다.
1. 작업 나누기
2. 데이터 나누기
3. 데이터 의존성
4. 밸런스
5. 테스트와 디버깅(여러 변수가 많음)

### Types of Parallelism
![Types of Parallelism](https://velog.velcdn.com/images/jws1228/post/e480c5f6-8e03-4301-97c9-c9e9f82c5467/image.png)

#### Data parallelism
- 여러 데이터에 동일한 작업을 할 때 데이터 부분을 나누어 병렬적으로 처리
#### Task parallelism
- 여러 개의 독립적인 작업을 병렬로 실행하는 것

-> 실제로는 두 가지 방법 모두 활용
<br><br>
### Amdahl's Law
- 칩을 추가하여 **병렬 처리성을 아무리 높여도** 순차적으로 실행되어야 하는 부분이 있기 때문에 **성능이 높아지는데는 한계**가 있다

![speedUp-Chip 그래프](https://velog.velcdn.com/images/jws1228/post/a8b44f69-0334-44d9-b274-0821baa7f482/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">speedUp-Chip 그래프</p>

-> 50%이면 성능이 아무리 증가해도 2배에 수렴
![Amdahl's Law](https://velog.velcdn.com/images/jws1228/post/8df3f168-bc49-4115-a198-36f9f7f8c730/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">Amdahl's Law</p>

---
## Multithreading Models

#### user thread 
- kernel의 위에서 동작하고 user-level thread library의 지원을 받는다
- 사용자 스레드 자체는 병렬 처리를 지원하지 않으므로 병렬 처리를 위해 개발자가 직접 스레드 생성, 동기화, 작업 분할 등을 직접 구현해야 함
#### kernel thread
- kernel의 지원을 받아 동작한다 
-  운영체제가 커널 스레드를 관리하기 때문에 스레드의 생성, 스케줄링, 동기화, 자원 할당 등의 관리를 효율적으로 수행할 수 있다

### Many-to-One
![many-to-one](https://velog.velcdn.com/images/jws1228/post/0b8b5248-db5e-490c-8bb3-b1deafb1b518/image.png)

- 여러개의 user thread가 하나의 kernel thread에 매핑
- 하나의 kernel thread 에 **의존적**이다
- **parallel하게 동작할 수 없다**
<br>

### One-to-One
![one-to-one](https://velog.velcdn.com/images/jws1228/post/971319be-201e-4907-84ec-bb5453e55019/image.png)

- 각각의 user thread와 kernel thread가 1:1로 매핑
- 동시성(concurrency)가 높아진다.
- multiprocessor환경에서는 **parallel**하게 동작한다
- 프로세스 당 **스레드의 수**가 **제한될 수** 있다.
<br>
### Many-to-Many
![many-to-many](https://velog.velcdn.com/images/jws1228/post/81cb5304-3b89-403b-aa76-6bdbd68ea3d6/image.png)

- user thread 수와 같거나 더 작은 kernel thread가 매핑 되어있다
- kernel thread **자원을 아낄 수** 있다.
- multiprocessor환경에서 parallel 하게 동작 가능
- 더 **유연하게 처리**할 수 있다(blocking 문제 x)
- **구현**이 **어렵다**
<br>

### Two-level 
![two-level](https://velog.velcdn.com/images/jws1228/post/6673bb61-1f3c-4248-b866-e65cc90cc2c3/image.png)

- **many-to-many의 변형**으로 ono-to-one과 many-to-many를 합쳐 놓음
- **중요한 작업**은 **ono-to-one**으로 처리

---
## Threading Issues

### fork()와 exec()의 처리
- fork를 호출한 thread만 복사할지 모든 스레드를 다 복사할지
- exec는 그냥 보통 다 덮어씀  
<br><br>
### signal handling
**signal:** **운영체제와 프로세스 간의 통신 메커니즘**으로 사용되며, 이벤트 기반 프로그래밍이나 비동기 처리 등에 활용될 수 있다

**signal handler:** 프로세스에서 시그널을 처리할 때 사용하는 것, 커널에서 동작하는 default handler와 개발자가 이를 오버라이딩한 user-defined signal handler존재

- 이 시그널을 특정 thread에만 보낼지 모든 thread에 보낼지, 또는 시그널을 전담하는 스레드를 할당할지
<br><br>

### Thread Cancellation

#### Thread Cancellation이란?

\- 정상적으로 스레드가 끝나기 전에 종료시키는 것, cancel시킨 스레드를 **target thread**라 함

#### Asynchronous Cancellation
- target thread를 즉시 종료 시킴

#### Deferred Cancellation
- target thread에게 스스로 종료할 수 있을 때 종료하도록 명령

![Tread Cancellation 표](https://velog.velcdn.com/images/jws1228/post/137ecb34-0c02-456e-a8b9-f740d33fee1b/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">Thread Cancellation 표</p>

- target thread가 Disabled 상태라면 cancel시킬 수 없다.
- Default는 Deferred Mode이며 이때 종료까지 지연되는 지점을 **cancellation point**라 한다.
<br><br>

### Thread-Local Storage

- 각각의 스레드가 가지고 있는 저장소
- thread 생성에 대한 제어권이 없을 때 유용
- 스레드 내에서만 공유되며 지역 변수와 전역 변수와는 다름
<br><br>

### Scheduler Activations

\- Many-to-many, Two-level model은 스레드간 매핑이 유동적이다. 이 매핑은 중간에 LWP가 관리하여 적절하게 커널 스레드가 유지되도록 한다.

![LWP의 위치](https://velog.velcdn.com/images/jws1228/post/58739a94-27e6-49a2-a668-4ddfe361a37f/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">LWP의 위치</p>

#### LWP(Lightweight Process)
- 각 커널 스레드마다 존재하는 **가상의 프로세스**
- 커널 스케줄러에 의해 스케줄되며, 스레드의 생성, 종료, 스케줄링, 동기화 등을 관리한다.
- 사용자 스레드와 커널 스레드 간의 **인터페이스 역할**


#### Scheduler activation
- 운영체제의 스레드 스케줄링을 개선하기 위한 **사용자 수준**의 **스레드 라이브러리**이다
- 이벤트가 발생할 때 **upcall**을 통해 커널 스레드에 알린다.
- 커널에서는 **upcall handler**를 통해 처리한다.
- 이러한 과정을 통해 유동적으로 매핑이 일어나고 이는 **LWP**를 통해 처리된다.