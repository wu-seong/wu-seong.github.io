---
title: 운영체제 정리 - Introduction
date: 2023-06-23 00:00:00 +/- TTTT
categories: [CS, 운영체제]
tags: [공부 정리]		# TAG는 반드시 소문자로 이루어져야함!
---
## 운영 체제 챕터 1: 소개
-----
### 운영체제는 무엇을 하는가?

유저와 컴퓨터 간의 중계 역할을 한다.

	1. 편리한 인터페이스를 제공하고
	2. 자원을 관리한다.

-----
	
### 컴퓨터 시스템 구조
    
    
  ![](https://velog.velcdn.com/images/jws1228/post/2910e559-46ee-40b4-90bb-270538477d85/image.png)

   
하드웨어 - 운영체제 - 응용 프로그램 - 유저
        
-> 중간 역할을 한다.

  
 *유저 입장에서의 운영체제가 하는 일*
 
 1. 자원 활용보다는 **성능의 편리함**에 초점


 2. 모바일 기기 - 통신, 입출력의 상호작용 
 

 3. 임베디드 - UI가 없거나 거의 없음
 
*컴퓨터 입장에서 운영체제가 하는 일*

1. 자원 할당 **resource allocator**

2. 프로그램 제어, 원할히 동작하도록 **control program**

<br>

\- 운영체제는 종류마다 모두 달라 한마디로 정의하기 어려움

#### 운영 체제 주요 구성 요소

* **kernel**:* 컴퓨터에서 항상 실행되는 프로그램, 하드웨어와 소프트웨어 사이에서 인터페이스 역할을 하며 운영체제의 다양한 기능 제공

* **middle ware**: 응용 프로그램 개발자가 운영체제로부터 제공받는 서비스 외의 추가적인 서비스를 제공하는 소프트웨어 프래임워크

* **system programs**: kerenl이외의, 시스템을 관리하는 프로그램

------
### 컴퓨터 시스템 조직

 CPU <- system bus -> device controllers
 
 * 각 기기의 **device controller**를 **device driver**를 통해 통신한다.
 
 * 각 컨트롤러에는 local buffer와 register가 있음
 
 **local buffer**: 작업한 결과를 저장하는 중간 다리
 
 **register**: 드라이버가 명령한 신호를 받는 부분
![](https://velog.velcdn.com/images/jws1228/post/dd277190-8779-4581-a5a4-25193a27ef10/image.png)




* local buffer와 main memory가 데이터를 주고 받는다.

* CPU에는 interrupt를 감지하는 센서가 있다.

#### Interrupt Handling

- **interrupt-request line**에 신호를 보낸다.

- cpu는 모든 동작마다 **interrupt-request line**을 확인한다.
- cpu가 **interrupt**를 감지하면, interrupt를 처리하는 루틴의 주솟값이 저장돼 있는**interrupt vector**를 통해 **interrupt-handler routine**으로 건너뛴다(우선 실행한다).

- 상태와 P.C 저장 -> interrupt 처리 -> 저장한 곳으로 복원

#### Interrupt 종류

* **Nonmaskable interrupt** - 중요한 인터럽트
ex) 전원 공급 장애, 기계 오작동, 메모리 손실

* **Maskable interrupt** - 덜 중요한 인터럽트 
ex) DMA(Direct memory access)인터럽트, 외부 인터럽트...
이 인터럽트는 우선순위가 높은 작업중이라면 지연될 수 있음

------
### 저장장치


![계층적 저장장치 구조](https://velog.velcdn.com/images/jws1228/post/71038eb2-1b62-463e-a7ea-075816a78910/image.png)

**RAM(Random access Memory):** 휘발성 메인 메모리로 DRAM 과 SRAM이 있음

<img src="https://velog.velcdn.com/images/jws1228/post/d3025b4b-45d2-4b02-92ee-88be50f12394/image.png" width="50%" height="50%" ></image>

**Bootstrap program:** OS를 메인 메모리에 올리기 위한 시스템 , 비휘발성 


### 입출력 구조
\- Controller의 buffer에 들어갈 정도의 작은 I/O 입출력(마우스, 키보드 등의 입출력)은 상관 없지만 NVS I/O와 같은 **용량이 큰 크기의 I/O 입출력**이라면 한번의 **interrupt**로 처리할 수 없어 **비효율적인 작업**이 일어나게 된다. 

이를 해결하기 위해서 **DMA(Direct Memory Access)**가 필요하다.

**DMA:** 빠른 I/O 입출력을 위해 Interrupt없이 buffer의 정보를 메모리에 올리는 것(block 단위), 이때 interrupt는 하나의 block당 일어난다 

=> interrupt를 (buffer size)/(block size)만큼 줄임

![](https://velog.velcdn.com/images/jws1228/post/2b2080f1-54db-41fd-9ea2-2283f2d868eb/image.png)

---------
### 컴퓨터 시스템 구조

#### Single-Processor System
- 하나의 cpu를 가진 프로세서를 사용하는 시스템

#### Multiprocessor System
1. Symmetric multiprocessing
<image src 	="https://velog.velcdn.com/images/jws1228/post/628d1065-da05-473c-8e9a-3b9e3f0c7a43/image.png" width ="50%" height ="50%" ></image>

<br/>
   
- 일반적인 single-processor system을 여러개 가지고 있는 시스템   
<br/>

- 동시에 여러 작업을 처리할 수 있다(병렬적으로)    
<br/>

- 하나에 작업이 쏠리지 않도록 운영체제가 관리 해야한다.    
<br/>


2. Multicore System
<br/>
  
<image src ="https://velog.velcdn.com/images/jws1228/post/50a8712e-1fd1-4892-baf9-1fe27b4bab47/image.png" width = "50%" height = "50%"></image>

<br/>
- 위의 Symmetric Multiprocessing과 다르게 하나의 칩에 여러개의 cpu가 있다.
<br/>
- 따라서 cpu간의 통신이 빠르다
<br/>
- 더 적은 전력을 소모한다.
<br/>
  
3. Non-Uniform Memory Access 
  
<br/>  
<image src ="https://velog.velcdn.com/images/jws1228/post/dfbab0cb-7b36-4056-811d-776f11c5bba7/image.png
" width = "70%" ></image>
<br/>
  - 각 cpu마다 고유 메모리가 있다.
<br/>
  - 확장성이 좋다
  <br/>
  - local memory에 대한 접근은 빠르다
  <br/>
  - 하지만 remote memory에 대해서는 interconnect에 대한 지연이 있다
  <br/>
<br/>
 4. Clustered Systems
 
    
<image src ="https://velog.velcdn.com/images/jws1228/post/49e9a6be-48fc-4ecb-8280-7d8e2234e9e5/image.png
" width = "70%" ></image>
<br/>
 - 여러개의 컴퓨터를 연결하여 만든 하나의 공유 저장소이다.
 <br/>
 - 이 저장소를 SAN(storage-area network)라고 한다.
 <br/>
 - 병렬 처리와 분산 컴퓨팅을 통해 높은 능률을 지닌다.
 <br/>
 - 확장성이 좋고 높은 가용성을 지닌다(하나가 다운되도 시스템이 유지)
    


-----

### OS 명령
    
#### Multiprogramming

<image src="https://velog.velcdn.com/images/jws1228/post/90183cd5-8c95-40bc-8e7f-b140ca1bc171/image.png" width="50%"></image>
  
  

- 여러개의 프로그램을 동시에 실행되도록 하는 것
- cpu가 항상 동작하게 하여 **cpu utilization을 높인다.**
  ex) I/O wait일 때 다른 process를 실행

  
- OS가 process를 동시에 실행하도록 한다(Not in parallel, concurrently)

    
#### Dual-Mode
  

![](https://velog.velcdn.com/images/jws1228/post/718f5033-fa8a-4b6a-b5dc-9147d2360fe7/image.png)
  
- OS를 보호하기 위해서 **user mode**와 **kernel mode** 두 개로 나눔
- user mode는 kernel 모드에 접근하기 위해 **system call**을 이용함
