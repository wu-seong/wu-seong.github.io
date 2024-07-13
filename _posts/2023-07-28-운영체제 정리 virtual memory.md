---
title: 운영체제 정리 - Virtual Memory
date: 2023-07-28 00:00:00 +/- TTTT
categories: [CS, 운영체제]
tags: [공부 정리]		# TAG는 반드시 소문자로 이루어져야함!
---

## 배경

실행 순간에 필요한 것만 메모리에 올려 놓음으로 메모리 활용성이 높일 수 있고 가상 메모리를 통해 이를 실현할 수 있다.
![가상 메모리](https://velog.velcdn.com/images/jws1228/post/17f9c1fa-4398-4648-99aa-ec371df0909e/image.png)

가상 주소를 통해 사용하여 실제 메모리 크기보다 큰 저장공간을 메모리 공간처럼 사용할 수 있다.
<br><br>

### 가상 메모리를 이용한 공유 라이브러리

![공유 라이브러리](https://velog.velcdn.com/images/jws1228/post/988115d4-891b-4de9-abf9-839a20a4e564/image.png)

stack과 heap사이의 메모리 공간을 공유 라이브러리의 공간으로 사용할 수 있다
<br><br>

---
## Demand Paging
프로세스의 메모리 페이지를 필요한 시점에 물리적 메모리에 올리는 방법
<br><br>

### Page Fault

demand paging 기법을 통해 필요한 것만 메모리에 올리다 보면 참조해야 하는 논리적 주소가 물리적 주소에 존재하지 않는 상황이 발생할 수 있음 그리고 이러한 상황이 **page fault**

#### page fault의 처리
![page fault 처리 과정](https://velog.velcdn.com/images/jws1228/post/86172501-fbdb-4c15-b576-21e6b70d5f62/image.png)

**valid-invalid 비트**를 통해 메모리에 존재하는지를 확인한 다음 존재하지 않는다면 **OS**에 제어권이 넘어가 **backing store**에서 정보를 가져와 메모리에 올린다음 **page table을 최신화**하여 다시 시도한다

## Free-Frame List
![free-frame-list](https://velog.velcdn.com/images/jws1228/post/d5531f4d-3578-414e-aa9a-9840b2faa19a/image.png)

page fault를 대비하여 대부분의 OS에서는 사용 가능한 물리 메모리를 유지한다
<br><br>

---
## Copy-On-Write
![copy-on-write](https://velog.velcdn.com/images/jws1228/post/0a0d1b99-cf01-4358-a6c6-2537213275d3/image.png)
fork를 통해 프로세스를 복제할 때는 같은 물리 메모리를 갖고 이후에 수정되는 것은 새로운 메모리를 가진다

---
## Page Replacement
**메모리가 가득 찼을 때** 필요한 페이지를 메모리로 옮기기 위해 메모리에 있는 **페이지를 제거**하는 것, 이때 어떤 페이지를 제거할지에 대해 여러가지 전략들이 있다
<br><br>

## First-In-First-Out(FIFO)
![FIFO](https://velog.velcdn.com/images/jws1228/post/449e4ff9-46e2-424a-b85e-be085a8f14bf/image.png)
가장 간단한 방법으로, 가장 빨리 들어온 것이 가장 빨리 나가는 전략이다

특이한 점으로 frame 크기에 비례하여 page fault가 감소하지 않는다.   오히려 늘어 날 수도 있다 이 현상을 **Belady's Anomaly**라 한다
<br><br>

## Optimal
![optimal](https://velog.velcdn.com/images/jws1228/post/57b39980-126f-447b-a9ad-1052f551ff32/image.png)
가장 이상적인 전략으로 앞으로 **가장 나중에 사용될 페이지를 제거**하는 전략이다. 하지만 실제로는 미래를 예측하기 어렵기 때문에 사용하기 어렵다
<br><br>

## Least Recently Used(LRU)
![LRU](https://velog.velcdn.com/images/jws1228/post/32e52d56-d72d-4b07-81b8-49a386bd11db/image.png)
**가장 마지막에 사용된 페이지를 제거**하는 전략이다. FIFO보단 효율적이고 Optimal보단 비효율적이다. 

### LRU구현
LRU를 구현하기 위한 두 가지 방법이 있다

1. ** 계수기**를 통해 각 페이지가 사용된 시각을 기록하여 가장 오래된 페이지를 제거하는 방법

2. **Stack** 자료구조를 이용하여 사용할 때마다 top으로 올리고 bottom에 있는 페이지를 제거하는 방법
<br><br>

### LRU-Approximation
LRU 방식을 구현하려면 보통 추가적인 하드웨어 장치가 필요하기 때문에 추가적인 **Reference bit**를 이용하여 LRU와 비슷하지만 조금은 완화된 방식을 많이 사용한다 
<br>
**LRU-Approximation 방법 종류**

- second-Chance    
- Enhanced Second-Chance   
- Additional-Reference-Bits   

#### Second-Chance 
![Second-Chance](https://velog.velcdn.com/images/jws1228/post/41534950-751f-4dfb-9312-0baf62a7fe14/image.png)
최근에 참조가 되었으면 page의 참조비트를 1로 set한다. **page replacement**가 필요할 때 페이지를 순회하며 참조 비트가 0(최근에 참조되지 않음)이면 선정하고 1(최근에 참조됨)이면 0으로 바꾸고 지나친다. 
<br><br>

#### Enhanced Second-Chance
참조 비트외에 **수정 비트**를 추가하여 페이지의 수정여부도 확인하여 victim을 선정하는 방법
<br><br>

#### Additional-Reference-Bits
**8개의 비트**를 이용하여 최근 사용되었는지를 표시하고 가**장 낮은 숫자를 가진 페이지를 제거**한다
ex) 
00000000 - 최근에 사용x, 우선순위가 가장 높음
10000000 - 가장 최근에 사용됨
01000000 - 최근에 사용되었지만 위보단 교체 우선순위가 높음
제거 우선순위 : 00000000 > 01000000 > 10000000
<br><br>

