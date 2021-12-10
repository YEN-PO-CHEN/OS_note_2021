# Chapter 5 : CPU Scheduling

## CPU Scheduler

- Four Process
  - (1) Switch from **running state** to **waiting state** (IO, wait for Child)
  - (2) Switch from **running state** to **ready state**
  - (3) Switch from **waiting state** to **ready state** (IO completion)
  - (4) Terminates

- Non-preemptive vs. Preemptive Scheduling

  - Non-Preemptive(不可搶奪型)

	> 除非一個 Process**主動釋放 CPU**，否則其他 Process 不能奪取其 CPU 的使用權

    - Cooperative Scheduling
    - 僅會發生在(1)、(4)
    - 不需要Hardware (timer) 去控制 

  - Preemptive (可搶奪型)

    > 不論是否正在執行的process 是否可以繼續執行，必要時    CPU使用權可能會被搶奪

    - 會發生在 (1)、(2)、(3)、(4)
    - 適用於：即時系統(Real-time System)、分時系統(Time Sharing System)
    - Context Switch (當 Process 交換時，須將 PCB 儲存，並且載入新的 PCB) 較大。(High overhead)
    - Unsafe

## Scheduling Criteria

- **CPU Utilization**
- **Throughput** 單位時間內，完成的 process 個數。
- **Turnaround Time** 一個 process 進入系統，到完成工作的時間。 
- **Waiting Time** process 待在 Ready Queue 等到獲取 CPU 的時間。
- **Response Time** User 下命令(request)到系統，系統產生**第一個回應**(response)的時間

## Scheduling Algorithm

#### **1. FCFS** (First-Come, First-Served) **Scheduling**

> 越早進來 (Arrival Time) 的 process 越優先取得 CPU。

- Practical
- **Convey Effort** (Average Waiting raise)
  - 多個 process 等待一個需要很長CPU time 才能完成的process (Average Waiting Time增加)
- **FCFS** is **Non-Preemptive**
- No Starvation

#### **2. SJF** (Shortest-Job-First) **Scheduling**

> 選 CPU Burst Time (中央處理器時間 ) 最短的，越優先取得 CPU。

- is the optimal Scheduling
- **SJF** is  **Non-Preemptive**
- Exist Starvation (有些process 無法跑完(Infinite Blocking))
- **不會有** Convey Effort

#### **3. SRTF** (Shortest-Remaining-Time-First) **Scheduling**

> 選 Process Remaining Time最短的，越優先取得 CPU。

- Content Switch 次數多
- **SRTF** is **Preemptive**
- Exist Starvation (有些process 無法跑完(Infinite Blocking))

#### **4. RR** (Round Robin) **Scheduling**

> OS 會規定一個 small unit of time ( **Time quantum** )，當 process 獲得 CPU 執行，若未能在 Time quantum 間完成，Process 會被迫放棄 CPU (From Running State to Ready State)，放回 ready queue 中，等待下一次使用 CPU。

- No Starvation
- **RR** is **Preemptive**
- Time-Sharing System
- **Longer average turnaround time** than SJF, but **better response time**
- Time quantum (q) performance
  - q large : FIFO
  - q small : Large context switch (HIGH overhead)
  - Time quantum 應該要大於 80% 的 CPU bursts

#### **5. Priority Scheduling**

> 優先權越高，越優先取得 CPU 控制權

- No Starvation
- Exist Starvation (Low priority processes may never execute)
  - Solution : **Aging** : 待在系統內時間長且還沒完成的process 逐步提高 priority value.
- **Priority Scheduling** is **Preemptive** or **Non-Preemptive**

#### **6. Multilevel Queue Scheduling**

>   Ready Queue 分成不同的queue，**process 會固定在同一個queue 中**。
>
>   1.   Fixed priority scheduling：
>        -   Foreground 會比 Background 先 process
>        -   會有 starvation
>   2.   不同 Queue 會有不同的 scheduling( Foreground / background )

1.	**Foreground** ( 需要交談的 )
	- RR 演算法
	- **高優先**
	- 80% 的 CPU 時間
2.	**Background** ( 可以批次的 )
	- FCFS 演算法
	- 低優先
	- 20% 的 CPU 時間

#### **7. Multilevel Feedback Queue Scheduling**

>   Ready Queue 分成不同的queue，**process 會在不同 level 的 queue 出現。**
>
>   -   Depend on **CPU bursts**
>       -   CPU time 使用量過高 => Lower priority。
>   -   每一層Queue使用不同的 Algorithm
>       -   Ex：越下層的( low priority 可以擁有的time quantum 越高(RR))，最下面一層可能是( FCFS )

1.   需要考慮：
     1.   number of queues
     2.   scheduling algorithm for each queue
     3.   the method used to determine when to upgrade a process
     4.   the method used to determine when to downgrade a process
     5.   the method used to determine which queue a process will enter when the process need service (簡單來說：就是哪個現在應該執行哪個 queue )

2.   Generic(通用的)、Complex Algorithm

## Multiple-Processor Scheduling

>    We consider **Homogenous Processors** ( A CPU include multi-cores )

### MP multi-processing

#### **ASMP**【Asymmetric Multiprocessing】

1.   每個(或每群) process 做不同的工作 ( 功能 )，其中一個 Processor 負責控制(Master processor)、協調及分配 process 到不同的 processor (slave processor)
     -   僅一個 processor access the OS data Structure( **kernel code** )，alleviating(減低) data sharing.
     -   other user processor run **user code**

#### **SMP** 【Symmetric Multiprocessing】 => **重點**

1.   每個 processor 都有相同的工作( 功能 ) ，當其中一個壞了，可以轉移到其他 processor 上。( 不會crash )

     -   Each processor is **self-scheduling**

     -   **Common or private queue**

         >   <img src="../image/ch5_1.png" style="zoom:50%;" />

         -   (左) Common ready queue ( **core** 不能同時做，存在lock)
         -   (右) Private ready queue     ( **core** 可以同時做，不存在lock)

##### Processor Affinity ( In SMP )

>   process 想儘量長時間運行在某個指定的 processor 上 ，且不被轉移至其他 processor( **Avoid Migration** )。
>
>   (盡可能的run在同一個 processor 上)
>
>   Cache miss rate ：在 process migrates to another processor

1.   **Soft Affinity**

     >   一個 process 可能會在不同 processor 上 run ( 無法保證在同一個 processor 上 )

2.   **Hard Affinity**

>   一個 process 保證在同一個 processor 上 run 完

##### Load Balancing ( In SMP )

>   負載度：讓工作量保持平均分配

-   Two Approaches:  [ Can co-exist (Linux) ]
    1.   Push migration
         -   將工作從過載的 processor 轉移 process 到其他 processor 中。
    2.   Pull migration
         -   從忙碌的 processor 內，拉回正在等待被執行的工作到閒置的 processor  中。

>   Load Balance will counteract the benefits of processor affinity

## Multicore Processors

