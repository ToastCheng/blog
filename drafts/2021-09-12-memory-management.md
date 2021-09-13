## Lec 06 Memory Management – Part I
https://www.youtube.com/watch?v=eua26hBK0h8&ab_channel=NYCUOCW

pros:
- shared by processes
- isolate processes from each other
- large contiguous per process address space
- use memory more efficiently

cons:
- slow down process, overhead such as
  - translating virtual memory to physical memory  
只有 single process 並不一定要 MMU
有些情況也會為了減少 overhead 或為了更好的 dererministic 關掉 MMU，

compiler, user 不會知道其拿到的記憶體是 physical 或 virtual

CPU -> translate memory -> 對 memory bus issue



## mechanism
### segmentation
virtual address: [ s | d ]
s => segment table 的 offset (index)
d => 記憶體地址的 offset (從 base 開始算)

      |    ...        |      ...            |
      |    ...        |      ...            |
      |    ...        |      ...            |
      |    ...        |      ...            |
s --> | limit (有多長) | base (從什麼地方開始) |
      |    ...        |      ...            |

檢察機制多半是在 CPU 發生的

segment table: 存 limit, base，可放 CPU 也可放 memory
  - 放 CPU 快速，但會占空間，排擠 CPU 的其他功能
  - 放 memory，segment 會需要兩次存取，但第二次存取同一個 segment 便不用

| segment | limit  |  base  |
| ------- | ------ | ------ |
| 1       | 0x1234 | 0x1200 |
| 3       | 0x2000 | 0x1234 |

還會有一個描述表，說明每段 segment 的權限等等
1: r only
3: rw


### page

virtual address: [ p | d ]

p => MMU 中的 offset (index)

[ p | d ] => MMU => [ f | d ] => physical memory
              |
             /
page table (in memory)

page with TLB
放在 CPU 內的 table 的 page table cache
context switch 會 flush TLB

### x86 用了 segment + page
[logical address] --> segmentation --> [linear address] --> paging --> [physical address]

page fault -> exception handler -> set memory -> rerun

設計 1: kernel 和 user 區分 address space
實際上沒有使用這種設計
因為導致每次有 syscall
進出 OS， Page table, TLB, Cache 就都得 flush
就得 context switch
成本太高

設計 2: 委屈求全 讓 kernel 和 user 共用 address space
32 bit 4G
1G Kernel 3G User
(比例可改)

## Lec 07 Memory Management – Part II
https://www.youtube.com/watch?v=kPmMeFSc-bQ&ab_channel=NYCUOCW

memory management
作業系統提供一套 api, syscall 讓應用程式可以取得/釋放合法的使用空間

Physical Memory Management
always required:即使關掉 MMU, 沒有 virtual memory, 還是需要 physical memory management
w/o mmu:
  - low cost
  - better performance
  - more dererministic
How to protect memory without MMU? MPU
存放在 CPU 內部的 table，記錄不同 segment(開始到結束) 的 policy (rw, cache) 以及用途

挑戰:
### external fragmentation 外部片段化
allocated memory 之間存在細小碎片 無法妥善利用 表的大小增加

設法將大型 allocation 和小型 allocation 做出區隔

### internal fragmentation

Frame table
把 physical memory 切成一塊一塊的 frame
#### 1.frame size = page size
要 1 byte 給 4k (frame)，多給的記憶體即是 internal fragmentation


trade-off:
- frame size 大 table 小
- frame size 小 internal segmentation 嚴重

#### 2.different frame size
embedded system 統計一下 memory 使用情況
同時避免 internal fragmentation 和 table 過大


### Design factor
- managing memory cost memory
- physical contiguos or logical contiguous?
  - application thread logical 連續, physical 不連續 因為比較不重要
  - kernel thread logical/physical 都連續
- external fragmentation
- internal fragmentation -> slab
- O(1) -> buddy system
- 提供 api 給 kernel (for drivers, OS code)/ user 使用

```
                     [ AP ]
                       |
                     malloc()               user mode
--------------------------------------------------------
                       | brk() / mmap()     kernel mode
                       |
vmalloc() ->    [ memory mapping ] -----------------\
physically 不連續       |                            |
logically 連續          | 小 memory                  |
                       |                            |
                       | kmalloc() /                |
                       | kmem_cache_alloc()         |
                       |                            |
                [ Slab Allocator ]                  | 大 memory
                       | _alloc_pages() /           |
                       | kmem_getpages()            |
                       |                            |
                [ Buddy System ] <------------------/
                       |
                       |
                       |
                [      Zone       ]
                [ Physical page   ]
                [ Physical memory ]
```

page allocator: 大的 2^n * 4k 
slab allocator: 小的

node, zone
ZONE_DMA (IO device), ZONE_NORMAL (kernel), ZONE_HIGH (user)

### slab allocator
發現 external frag 問題不大，主要是 internal frag 
發現 kernel 要記憶體的大小就固定那幾個 (task_struct, mm_struct)

長時間 allocate + free 造成片段化 配置時間增加
不妨 allocate 就給， free 也不用還，先留著
在 kernel program 和 buddy system 中間加入 cache



## Lec 08 Memory Management – Part III
https://www.youtube.com/watch?v=1zMipcUhsOs&ab_channel=NYCUOCW

Process address space 相關的 memory management

per process logical memory

kernel: 1G
stack: declare variable
...
memory mapping
...
heap: dynamic
bss: block start with symbol, 像是 int buffer[200]
data: static data, const, global
text: 程式 binary code


task_struct --> mm_struct -->  stack, heap, bss, data, ...
task_Struct 用來 schedule
mm_struct 用來作 memory management

## VMA
vm_area_struct --> page1 --> frame11
               --> page2 --> frame3
               --> page3
               --> page4 --> frame102

anonymous:
    stack
    heap
    bss
file-backed memory:
    data
    text
    memory mapping: ld.so, libc.so

file-backed 好處
不同 process 可以共用，例如 glibc
可作 memory-mapped IO
可作 IPC


VMA search using RB tree
用 vm_end 當 key