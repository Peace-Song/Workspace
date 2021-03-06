Operating System
=================
# 1. Introduction
> OS is a software that converts hardware into a useful form for applications.

쉽게 말해, 운영체제는 일종의 정부이고 유저 애플리케이션은 정부의 관리와 지원 아래 동작하는 사회 각 기관이다. 


## 운영체제를 바라보는 시각들
### Application View
> OS provides an execution environment for running programs, and an abstract view of the underlying computer system.

참고로 abstraction은 복잡한 구현을 간단하게 보여주는 것이고, virtualization은 애플리케이션에게 실제로 존재하는 것 처럼 보여주는 것이다.

### System View
> OS manages various resources of a computer system.

운영체제는 메모리, I/O 디바이스, 심지어 전력 등의 자원을 다룬다. 여기서 다룬다는 의미는 시간과 공간을 프로세스에게 배분하고, 각 프로세스로부터 다른 프로세스를 보호하며, 공정하게 자원을 분배하는 등의 작업을 효율적으로 한다는 것을 뜻한다.

### Implementation View
> OS is highly concurrent, event-driven software.

운영체제는 동시에 여러 작업을 수행함과 동시에 event-driven, 즉 애플리케이션에서 보내는 system call이나 하드웨어에서 보내는 interrupt 등의 event가 발생했을 때에 돌아가야 한다.

## 운영체제의 역사
우리에게 친숙한 Unix의 역사는 1965년의 Multics로부터 시작된다. Multics는 MULTiplexed Information and Computing Service의 약자로, 그 시절에 굴러가기에는 너무 덩치가 컸지만 hierarchical file system, VM, user-level shell, dynamic linking, PL/I라는 high-level language로 구현되는 등 지금도 사용하는 여러 기술들이 처음으로 소개된 운영체제이다.

Unix는 C를 개발한 **Dennis Ritchie**와 **Ken Thompson**에 의해 개발되었다. Unix는 UNiplexed Information and Computing Service의 약자로, i-node의 도입, process control, pipe를 이용한 IPC, I/O redirection, signal 등의 기능이 추가되었다. Multics보다 가볍고 쉽게 짜였다고 한다. 운영체제의 역사에서의 Unix의 중요성을 모르는 사람이 이 글을 보고 있을 리가 없겠지?
<p align="center"><img src="./res/KT&DR.jpeg" height="300px" title="Dennis Ritchie and Ken Thompson" alt="Their Majesties"></img></br>존나 멋있다 나도 이런 사람들이 되고싶따</p>

# 2. Architectural Support for OS
## Issue 1. I/O

I/O 디바이스와 CPU는 동시에 돌아갈 수 있다. 따라서 I/O 디바이스에 명령을 넣고 이를 기다리는 건 소중한 CPU 자원의 낭비이므로, I/O 디바이스에 명령을 내린 후 디바이스가 이를 완료할 때 까지 CPU는 다른 작업을 하게 해야 한다. 이를 위해 디바이스는 작업을 완료하면 interrupt를 날리고 CPU는 이를 받아 interrupt handler를 실행한다. 

그런데 SSD를 예로 들어 보자. SSD는 아무리 빨라봤자 메모리보다 한참 느리지만, 한 번에 4KB씩 전송이 가능하다. 하지만 메모리는 한 번에 8B씩 전송이 가능하다. 이를 커버하려면 CPU는 512번 왔다갔다 하며 SSD에서 메모리로 load해야 할 것이다. 이 역시 소중한 CPU 자원의 낭비이다. 
<p padding="10px" align="center"><img src="./res/wo_DMA.jpeg" height="200px" title="CPU 혼자 다해" alt="CPU 혼자 다 하는 짤"></img></br>CPU가 메모리와 I/O 사이에서 열심히 구르고 있다.</p>

**DMA(Direct Memory Access)**를 사용하여 이를 해결할 수 있다. DMA는 I/O에게는 CPU처럼 동작하여 CPU가 다른 일을 하는 동안 I/O 디바이스와 메모리를 중개한다. I/O 작업이 끝나면 디바이스가 아니라 DMA가 CPU에게 interrupt를 보내 일을 처리했음을 알린다.
<p padding="10px" align="center"><img src="./res/w_DMA.jpeg" height="200px" title="DMA한테 외주 줌" alt="DMA가 외주 받는 짤"></img></br>DMA가 CPU 대신 메모리와 I/O를 중개해주고 있다.</p>

## Issue 2. Protection

운영체제는 항상 사용자 애플리케이션이 선을 넘는 행동을 하지 않도록 감시해야 한다. 사용자 애플리케이션은 커널을 거치지 않고 하드웨어에 직접 접근할 수 없으며, control register이나 자기 자신에게 허용된 범위를 벗어나는 메모리 범위에 접근할 수 없고, HLT instruction도 실행해서는 안된다. 이러한 명령어들을 **Privileged** 또는 **Protected** instruction이라고 한다. 

이를 위해 운영체제는 2개 이상의 작동 모드를 지원한다. Privileged instruction을 실행할 수 있는 모드는 커널 모드, 그렇지 않은 모드는 유저 모드로 구분한다. x86에서는 Ring 0이 커널 모드, Ring 3이 유저 모드로 구분되며 RISC-V에서는 처음 부팅 시에만 가동되는 Machine 모드를 제외하고, Supervisor가 커널 모드, User이 유저 모드로 구분된다. 유저모드인 상태에서 privileged instruction이 들어오면 권한 오류가 날 것이다. 이 작동 모드는 CPU 내에 현재 작동하고 있는 모드 bit이 존재하여 구분할 수 있다.

## Issue 3. Servicing Requests

유저 애플리케이션이 실행할 수 있는 명령어의 범위에는 한계가 있다고 했다. 그렇다면 권한이 없는 명령어는 어떻게 실행해야 하는가? 이는 **System Call**을 통해 가능하다. System call은 사전에 운영체제 안에 정의되어 있으며, protected procedure call이므로 커널 모드에서 실행된다. 유저 모드에서 System call을 실행한다면 커널 모드로 바뀌어 실행되며, 프로시져를 나갈 때 다시 유저 모드로 바뀌어 나가게 된다. 예를 들어 유저 프로세스에서 printf()를 호출하면, 이는 libc.c에 정의된 write()를 호출하며, write()는 내부에서 system call을 호출하여 커널 모드에서 나머지를 실행하고 유저 모드로 돌아오는 것이다.

System call은 trap에 해당하며 intentional exception이다. x86에서는 INT instruction으로, RISC-V에서는 ecall instruction으로 호출한다. a7 레지스터에 system call number를 저장하여 어떤 종류의 system call인지 알 수 있다. 

----

c.f. Exceptions in x86    
| <center>Trap</center> | <center>Faults</center> | <center>Abort</center> |
| ---- | ---- | ---- |
| intentional | unintentional, but possibly recoverable | unintentional and unrecoverable |
| system calls, breakpoints, special instructions, ... | page faults(recoverable), protection faults(unrecoverable), ... | parity error, machine check, ... |
| returns control to the next instruction | re-execute current instruction or abort | abort | 

----

## Issue 4. Control
커널 코드는 돌아가고 있는 프로세스가 커널에게 CPU를 넘겨주던가(e.g. yield()), system call을 호출하거나, 아니면 하드웨어에서 interrupt가 나야 실행될 수 있다. 바꿔 말하면 프로세스가 CPU를 독점하면서(e.g. 무한 루프) system call을 호출하지 않고 하드웨어에서 interrupt가 나지 않는다면 커널 코드는 영원히 실행될 수 없고, 따라서 context change 등의 유용한 작업을 할 수 없게 된다.

이를 해결하기 위해 가장 쉬운 방법은 프로세스가 스스로 CPU의 사용을 멈추는 것이다! 프로세스가 yield() system call을 호출하면 커널에게 CPU 통제권이 넘어가고, 스케쥴러를 호출하는 등의 작업을 할 수 있게 된다. 이러한 방식은 커널이 유저 애플리케이션을 신뢰할 수 있을 때 유용하다.

하지만 대부분의 경우 커널은 유저 애플리케이션을 신뢰할 수도 없고 신뢰해서도 안된다. 이를 해결하기 위해 하드웨어 레벨에서 **timer**를 설정하여 이를 해결할 수 있다. 하드웨어 timer는 주기적으로 interrupt를 생성해냄으로써, 주기적으로 무조건 OS에게 컨트롤을 쥐어줄 수 있다. 

## Issue 5. Memory Protection
유저 애플리케이션은 하드웨어에 직접 접근할 수 없기 때문에 반드시 커널을 거쳐야 한다. 하지만 메모리만큼은 직접 접근할 수 있다. 메모리마저도 system call을 거쳐 접근한다면 너무 느려지기 때문이다. 하지만 이 경우에도 커널은 유저 애플리케이션을 신뢰할 수 없기 때문에, 각 유저 애플리케이션은 서로로부터 보호되어야 하며, 커널 자체도 유저 애플리케이션으로부터 보호되어야 한다. Virtual Memory는 메모리를 가상화하여 각 프로세스마다 비어있는 가상 메모리 공간을 할당하여 메모리를 보호할 수 있다. **MMU**는 하드웨어 차원에서 메모리 가상화를 지원해준다. 

## Issue 6. Synchronization
프로세스끼리는 서로의 메모리를 볼 수 없지만, 스레드는 가능하다. 따라서 스레드끼리 공유하는 메모리에 모종의 조치를 취하지 않으면 예상치 못한 동작을 할 수 있다. 이를 보호하기 위해서 특별한 **atomic instructions**가 지원된다.

# 3. Process
> Process is **an instance of a program in execution**.

프로세스끼리는 서로 분리되어, 각 프로세스는 protection의 가장 기본적인 단위가 된다. 하나의 프로세스는 다음과 같은 정보들을 포함한다:
* CPU context (registers)
* OS resources(address space, open files, etc.)
* Other information(PID, state, owner, etc.)

<p align="center"><img src="./res/prog-proc.jpeg" height="200px" title="Process와 Program의 관계" alt="Process-Program"></img></br>프로세스는 실행 중인 프로그램의 인스턴스이다.</p>

## Unix와 Windows의 프로세스 차이

| UNIX | Windows|
| ---- | ---- |
| Process Group 존재 | X |
| fork()와 exec()이 따로 존재 | CreateProcess() == fork() + exec() |

Unix에서 fork()와 exec()을 분리한 이유는 I/O redirection, pipe 등을 fork 후, exec 전에 실행하기 위해서이다. xv6/user/sh.c 참조할 것.

## Process Termination
* Normal exit: voluntary
* Error exit: voluntary
* Fatal error: involuntary
  * Segmentation fault, Protection fault, Exceeded allocated resources, ...
* Killed by another process: involuntary
  * Killed by receiving a signal

한 프로세스가 위와 같은 이유로 죽으면 메모리에서 즉시 사라지는 것이 아니라 **Zombie process**가 된다. 이는 좀비가 된 프로세스가 자신의 사망 원인을 가지고 있도록 하여, 이 정보를 읽은 후에야 성불할 수 있게 하기 위함이다. 나무아미타불 관세음보살...

## Context Switch
### PCB (Process Control Block)
하나의 PCB는 하나의 프로세스를 나타낸다. PCB에는 다음과 같은 정보들이 있다.
* CPU registers
* PID, PPID, process group, priority, process state, signals
* CPU scheduling information
* Memory management information
* Accounting information
* File management information
* I/O status
* Credentials
* ...

PCB에서는 Linux에서 **task_struct** 구조체로 구현되어 있으며, xv6에서 **proc** 구조체로 구현되어 있다.

### Context Switch
CPU가 실행할 프로세스를 다른 프로세스로 바꾸는 것을 context switch라고 한다. 모든 컴퓨터의 작업이 그렇지만 당연히 overhead가 존재하며, 이는 register와 메모리 매핑을 저장 및 복구하거나 cache를 flush하고 reload하는 등의 작업에서 기인한다. 당연히 cache locality 따위는 기대할 수 없으며, 초반에는 cold miss가 빈발할 것이다. 따라서 너무 잦은 context switch는 독이 된다. 보통 초당 100-1000번 이루어진다고 한다.

xv6에서 context switch는 다음과 같은 과정을 통해 이루어진다. 각 단계는 atomic하게 실행된다.

----

1. Timer interrupt (**Process A** in **user mode**)
   
2. Set *sepc* <- *pc*, *scause*     
   Disable interrupt    
   Change to kernel mode            
   Jump to trap handler at *stvec*   
   
3. Save *user registers* to *trapframe(A)* (**Process A** in **kernel mode**)    
   Change to kernel page table   
   Make A's state *RUNNABLE*   
   Save A's *context* to *PCB(A)*   
   Run *scheduler*   
   Make B's state *RUNNING* (**Process B** in **kernel mode**)   
   Restore B's *context* from *PCB(B)*

4. Change to user page table    
   Restore *user registers* from *trapframe(B)*    
   return from trap (sret)    

5. Move back to user mode (**Process B** in **user mode**)    
   Enable interrupt    
   Set *pc* <- *sepc*    

----

~~~ c
// Saved registers for kernel context switches.
struct context {
  uint64 ra;
  uint64 sp;

  // callee-saved
  uint64 s0;
  uint64 s1;
  uint64 s2;
  uint64 s3;
  uint64 s4;
  uint64 s5;
  uint64 s6;
  uint64 s7;
  uint64 s8;
  uint64 s9;
  uint64 s10;
  uint64 s11;
};

struct trapframe {
  /*   0 */ uint64 kernel_satp;   // kernel page table
  /*   8 */ uint64 kernel_sp;     // top of process's kernel stack
  /*  16 */ uint64 kernel_trap;   // usertrap()
  /*  24 */ uint64 epc;           // saved user program counter
  /*  32 */ uint64 kernel_hartid; // saved kernel tp
  /*  40 */ uint64 ra;
  /*  48 */ uint64 sp;
  /*  56 */ uint64 gp;
  /*  64 */ uint64 tp;
  /*  72 */ uint64 t0;
  /*  80 */ uint64 t1;
  /*  88 */ uint64 t2;
  /*  96 */ uint64 s0;
  /* 104 */ uint64 s1;
  /* 112 */ uint64 a0;
  /* 120 */ uint64 a1;
  /* 128 */ uint64 a2;
  /* 136 */ uint64 a3;
  /* 144 */ uint64 a4;
  /* 152 */ uint64 a5;
  /* 160 */ uint64 a6;
  /* 168 */ uint64 a7;
  /* 176 */ uint64 s2;
  /* 184 */ uint64 s3;
  /* 192 */ uint64 s4;
  /* 200 */ uint64 s5;
  /* 208 */ uint64 s6;
  /* 216 */ uint64 s7;
  /* 224 */ uint64 s8;
  /* 232 */ uint64 s9;
  /* 240 */ uint64 s10;
  /* 248 */ uint64 s11;
  /* 256 */ uint64 t3;
  /* 264 */ uint64 t4;
  /* 272 */ uint64 t5;
  /* 280 */ uint64 t6;
};
~~~

> Q. context에 있는 모든 정보는 trapframe에도 있는데 굳이 둘을 나누는 이유는? context에 있는 레지스터는 callee saved라는 것과 관계가 있나? 필기에는 trapframe은 user process의 state, context는 그 process가 커널 모드로 들어갔을 때 커널의 state라고 되어 있는데, 이 말은 커널 모드에서는 callee-saved register만 사용한다는 의미인가? 본인 x86충이라 RISC는 잘 모르겟움 ㅠ

### Policy와 Mechanism
* Policy
  * **WHAT** should be done?
  * resource allocation이나 scheduling에 대해서 이루어진다.
  * e.g. 다음에는 어떤 프로세스를 돌려야 하는가?

* Mechanism
  * **HOW** to do something?
  * Policy를 실제로 구현하는 도구이다.
  * e.g. 어떻게 여러 프로세스가 동시에 돌아가게 할 수 있는가?

Policy는 여러 상황에 따라 변할 수 있다. 하지만 mechanism은 policy와는 별개로 구현되어 어떠한 policy를 사용하더라도 문제가 없도록 유연하게 구현되어야 한다. 이러한 policy와 mechanism의 구분은 더 modular한 OS를 설계하는 데 도움을 준다.


# 4. CPU Scheduling
**Non-preemptive scheduling**은 OS가 유저 프로그램을 신뢰할 수 있을 때 사용하는 것이 좋다. 유저 프로그램은 자신이 할 일을 마친 후 CPU를 내려놓기 때문에, OS가 간섭할 여지가 없기 때문이다. 만약 어떠한 악성 프로그램이 CPU를 내려놓지 않는다면, 이를 제재할 방법은 없다.

**Preemptive scheduling**은 OS가 유저 프로그램을 신뢰하지 않는 경우에 사용된다. 어떠한 정책으로든, 스케쥴러가 실행 중인 프로세스를 중단하고 CPU 권한을 회수하여 context switch가 발생하게 된다.

## Workload Assumptions

----

1. Each job runs for the same amount of time.
2. All jobs arrive at the same time.
3. Once started, each job runs to completion.
4. All jobs only use the CPU (no I/O)
5. The run time of each job is known.

----

## Metrics

----

1. Turnaround time
  * Job을 제출한 이후부터 Job이 끝날 때 까지의 시간
  * T<sub>turnaround</sub> = T<sub>completion</sub> - T<sub>arrival</sub>

2. Response time
  * Job을 제출한 이후부터 Job이 실제로 실행될 때 까지의 시간
  * T<sub>response</sub> = T<sub>firstrun</sub> - T<sub>arrival</sub>


----

## FIFO
> First-In, First-Out

Jobs가 들어온 순서대로 처리된다. 모든 jobs는 공평하게 들어온 순서대로 실행되므로 starvation이 일어나지 않고, non-preemptive하다. 하지만 **Convoy effect**가 발생하게 된다. 즉, 실행 시간이 짧은 job이 그렇지 않은 job보다 늦게 도착할 경우 해당 job이 실행될 때 까지 기다려야 하므로 평균 turnaround time이 증가하게 된다.

## SJF
> Shortest Job First

**"Assumption 1. Each job runs for the same amount of time"**을 제외하자. 그렇다면 jobs의 실행 시간은 다를 수 있고, convoy effect를 최소화하기 위해 실행 시간이 가장 짧은 job부터 순차적으로 실행하게 된다. 이 경우 turnaround time을 최적화할 수 있다는 것이 '알려져 있다'. 이 스케쥴링도 non-preemptive하다. 하지만 이 스케쥴링은 여전히 모든 job이 동시에 도착하는 것을 전제로 하므로, job의 도착 시간이 제각기인 경우 최적해임을 보장할 수 없으며 이 경우 starvation이 여전히 일어날 수 있다.

<p align="center"><img src="./res/FIFO_SJF.png" width="500px" title="FIFO와 SJF 비교" alt="FIFO-SJF"></img></br>FIFO와 SJF</p>

## STCF
> Shortest Time-to-Completion First

**"Assumption 2. All jobs arrive at the same time"**과 **"Once started, each job runs to completion"**이 제외되었다. 그렇다면 이제 스케쥴러는 preemption을 수행할 수 있다. 따라서 스케쥴러의 실행 시점에서 실행 완료에 가장 가까운 프로세스를 선택하여 실행하게 된다. 이 경우에도 starvation은 여전히 일어날 수 있다.

<p align="center"><img src="./res/SJF_STCF.png" width="500px" title="SJF와 SCTF 비교" alt="SJF-STCF"></img></br>SJF와 SCTF</p>

## RR
> Round Robin

Run queue는 이제 Circular queue로 구현된다. 모든 job은 timeslice를 분배받게 되며, 자신의 timeslice 동안 프로세스는 CPU를 마음껏 사용하거나 I/O를 기다리기 위해 yield할 수 있다. timeslice를 다 쓰면 run queue 상의 이웃 job에게 CPU를 넘겨주게 된다. preemptive한 스케쥴링이며, 어쨌든 기다리다 보면 timeslice를 받기 때문에 starvation이 일어나지 않고, 공평하게 시간을 분배하므로 response time이 향상된다. 다만 SJF보다 turnaround time은 느려지게 된다. FIFO의 preemptive한 버전이라고 할 수 있겠다. 

<p align="center"><img src="./res/SJF_RR.png" width="500px" title="SJF와 RR 비교" alt="SJF-RR"></img></br>SJF와 RR</p>

## Priority Scheduling

각 job에는 priority가 있으며, priority value가 낮을 수록 높은 우선순위를 가진다고 간주된다. job마다 priority가 주어져 바꿀 수 없으면 static priority scheduling이고, 바꿀 수 있다면 dynamic priority scheduling이다. 그리하여 가장 높은 priority를 가진 job을 선택하고, 같은 priority를 가진 job끼리는 RR이나 FIFO를 사용한다. preemptive할 수도, non-preemptive할 수도 있지만 보통은 preemptive한 스케쥴링을 사용한다. 여전히 starvation problem에서 벗어날 수는 없는데, 계속해서 높은 priority를 가진 job이 밀려온다면 낮은 priority의 job은 실행되지 않기 때문이다.

**"Assumption 4. All jobs only use the CPU (no I/O)"** 전제를 제외하자. 그렇다면 job은 I/O를 받을 수 있게 되고, 그 동안 job은 CPU를 사용하지 않을 것이다. 그렇다면 CPU를 사용하는 동안의 job과 I/O를 기다리는 동안의 job은 같은 job이어도 다르게 본다면, I/O를 기다리는 동안 다른 job을 스케쥴링하여 실행할 수 있을 것이다. 그렇다면 turnaround time과 response time 두 경우 모두 개선될 수 있을 것이다. 

## MLFQ
> Multi-Level Feedback Queue

----
1. If Priority(A) > Priority(B), A is chosen.
2. If Priority(A) == Priority(B), A and B runs in RR.
3. When a job is submitted, it is given the highest priority.
4. If a job uses up its given time slice, regardless of its number of yields, the priority is reduced. (**Precise accounting**)
5. After some time period, all jobs are moved to the topmost queue. (**Priority boost**)

<p align="center"><img src="./res/priorityboost.png" width="500px" title="priority boost" alt="priority boost"></img></p>
<p align="center"><img src="./res/preciseaccounting.png" width="500px" title="preciseaccounting" alt="preciseaccounting"></img></p>


## CFS
> Completely Fair Scheduler

리눅스는 140개의 priority를 가지고 있으며, 일반적인 유저 프로세스는 100부터 139까지의 priority를 할당받는다. 이 때 priority 120은 nice 0에 해당하여, priority = 120 + nice로 나타내진다. CFS는 각 task의 nice value마다 weight를 할당하여 CPU의 시간을 할당해준다. nice value 1 차이당 10%의 CPU 시간을 주도록 매핑하여 매핑값을 미리 계산해둔다.

이 때 CPU 사용량을 해당 task가 nice value 0이었을 때를 기준으로 normalize하여 virtual runtime을 계산한다. 이는 다음과 같다.

VR(T) = ( Weight<sub>0</sub> / Weight(T) ) * PR(T)

CFS는 모든 task를 virtual runtime을 기준으로 red-black tree를 사용하여 sort함으로써, 항상 O(log N) 시간에 가장 작은 virtual runtime을 가진 task를 찾음으로써 이를 스케쥴링 할 수 있다. 이 때 virtual runtime이 적다는 것은 실제 써야 할 CPU 시간보다 더 적게 CPU를 사용했다는 의미이다. 


