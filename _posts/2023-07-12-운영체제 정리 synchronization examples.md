---
title: 운영체제 정리 - Synchronization Examples
date: 2023-07-12 00:00:00 +/- TTTT
categories: [CS, 운영체제]
tags: [공부 정리]		# TAG는 반드시 소문자로 이루어져야함!
---


---

## Classic Problems of Synchronization 

### Bounded-Buffer Problem

\- 3장의 Process 부분에서 잠깐 나왔던 생산자-소비자 문제와 같은 것이다.

[Process 정리 보기](https://velog.io/@jws1228/운영체제-정리-Processes)

![](https://velog.velcdn.com/images/jws1228/post/0accfac7-4189-4c18-9652-869e61b8b4d6/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">Bounded-Buffer Problem에 필요한 변수</p>

- 공통적인 Critical Section에 진입을 담당하는 세마포어 **mutex**
- 생산자가 남은 공간일 때만 진입할 수 있게 하는 세마포어 **full**
- 소비자가 소비할 것이 있을때만 진입할 수 있게 하는 세마포어 **empty**
<br><br>
![](https://velog.velcdn.com/images/jws1228/post/cf9263eb-ebed-4421-9152-64b220bd258a/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">생산자 코드</p>


![](https://velog.velcdn.com/images/jws1228/post/301570e6-32b6-4297-add9-534e344a1777/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">소비자 코드</p>


<br><br>
### Readers and Writers Problem
\- 글을 읽고 쓰는 과정에서 reader와 writer의 Critical Section 문제

#### 필요한 세마포어

- writer가 진입할 수 있게 하는 세마포어인 rw_mutex
- 글을 읽을 reader의 수를 세는 counting semaphore인 read_count
- read_count의 증감을 관리하는 세마포어인 mutex


![](https://velog.velcdn.com/images/jws1228/post/bcc1fe61-7810-437d-8133-6888a44420a5/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">writer 코드</p>

![](https://velog.velcdn.com/images/jws1228/post/01ac4cdd-f8f5-41d7-93d3-04d6bb612567/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">reader 코드</p>

- **writer**는 writer와reader 둘 다 상호배제 되어야 하고** reader**는 같은 reader끼리는 같이 진입을 해도 된다. 
- writer가 글 작성을 완료해야만 reader가 진입이 가능하다.
- 모든 reader가 글을 읽어야 writer가 글을 쓸 수 있다.
- 이 때 글 작성은 첫번째 reader만 확인하면되고, 모든 글을 읽었는지는 남은 reader를 확인하면 되니, 이를 나타내기위해 **read_count**를 사용한 것
<br><br>
### Dining-Philosophers Problem
![](https://velog.velcdn.com/images/jws1228/post/0e5b90f6-fec3-41ce-909d-b76f71c87391/image.png)
\- 철학자 5명이 원탁에 둘러 앉아 저녁을 먹는상황, 두 젓가락이 모두 있어야 밥을 먹을 수 있고 젓가락은 각자 1개씩 양 옆에 있음

![](https://velog.velcdn.com/images/jws1228/post/7cf34816-9c02-4b9f-b5a6-d277d6e68fbc/image.png)
<p align="center" style=" font-size: 12px;
color: #999;">semaphore를 이용한 방법</p>

\- 해결이 된 것처럼 보이지만 만약 모든 철학자들이 동시에 오른쪽/왼쪽 젓가락을 집는다면 deadLock(무한대기)에 빠지게 됨

#### 해결방법
1. 동시에 젓가락 한짝을 잡을 수 있을 때만 잡을 수 있도록 한다.
2. 최대 4명만 같은방향의 젓가락을 집을 수 있도록 한다.

이외에도 홀수/짝수 번째 사람만 먼저 잡도록 하기 등등..

\- 아래에서는 THIKING, EATING 이외에 HUNGRY 상태를 추가하여 두 젓가락 모두 잡을 수 있을 때만 잡도록 하였다. 
 => 두 젓가락을 집는 것을 하나의 원자적인 행동으로 보장

![](https://velog.velcdn.com/images/jws1228/post/738c7dd5-43ed-46d3-acdf-f0683a74a303/image.png)

<p align="center" style=" font-size: 12px;
color: #999;">1번 방법을 Monitor를 활용하여 구현한 코드</p>

\- 데드락은 발생하지 않지만, 좌우 철학자가 번갈아가며 먹으면 starvation은 발생할 수 있다

---