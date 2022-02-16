# Notes on *Operating Systems : Three Easy Pieces*

Author: Akiyama

Create : 13 Feb 2022

Update: 15 Feb 2022









# Visualization : Part One





## Process need go over

TO DO





## Scheduling

### Metric:

* $ T_{turnaround} = T_{complection} - T_{arrival} $
* $T_{response}  = T_{firstrun} - T_{arrival}$

### FIFO (First In, First Out) / FCFS (First Come, First Serve)

* Bad in both Response Time and Turnaround TIme
* Non-preemptive scheduler

### Shortest Job First (SJF)

* Non-preemptive scheduler
* Bad in Response Time
* Improved in Turnaround Time

### Shortest Time-to-Complection First (STCF)

* Preemptive scheduler 
* Much improved in Turnaround Time
* Bad in Response Time

### Round Robin

* Good in Reponse Time. 
* Less the __Time Slice__, better the Response Time
* Bad in Turnaroung Time

### Multiple Level Feedback Queue

* Good Turnaround Time
* Good Response Time

### Lottery Scheduling

__Unfairness Metric :__ $ U =  Time_{A} / Time_{B}$ , where $ U = 1 $ will get the best fairness

* Only be precise in large job scale, because we use randomness
* Use List structure for implementation

### Stride Scheduleing

* Avoid randomly unfairness even in small number of jobs

* Pick the process with lowest __pass value__. , run it with given __stride__ of time, and increase its __pass value__, and push it back to queue finally
* It have __global state__ , so it's worse than __Stride Scheduling__ in some way

### Linux Complete Fairness Scheduler (CFS, Only Real-world Scheduler)

* Pick process with lowest __vruntime__, CFS use `sched_latency` / number of process, to determine the time of the process to run, but it cannot be lower than `min_granularity`
* Introduced Weighting to calculate __vruntime__
* Use RB-Tree to store running processes



## Virtual Memory

### Principles

* __Isolation__: even some kernal are divided into serveral __microkernal__
* __Transparency__: cannot be deteceted from process that its memory adrdress is virtual
* __Efficiency__: on time and space

### Fun Fact

* Any address A programmer can print out is a __virtual address__

* Allocation and deallocation is implicitly managed by compiler
* `malloc()` use `brk` and `sbrk` system calls to implement itself



### The `malloc()` Call

`````c
#include <stdlib.h>
...
void *malloc(size_t size);
`````

`sizeof()` is a _compile-time_ operator, meaning that the actual size is know at _compile time_

Think it as an operator, but not a function call(which would take place at _run time_.

To allocate space for a double-precision floating point value, you simply do this: 

``` c
double *d = (double *) malloc(sizeof(double));
```

Another place to be careful is with strings. When declaring space for a string, use the following:

```c
malloc(strlen(s) + 1) //adds 1 to it in order to make room for the end-of-string character
```

It returns a `void *p` , then the programmer do the __cast__  to tell the _compiler_ to do the right thing

### The `free()` Call

```c
int *x = malloc(10 * sizeof(int));
...
free(x);
```

### Typical Errors

* Forgetting to free memory

* Freeing memory before you're done with it

* Freeing memory repeatly

* Call `free()` with wrong argument

  

## Address Translation

### Aside: Static Relocation

In early days, we use software called __loader__ takes an executable, rewrite its all address by adding desired offset. It does not have _protection_, and _hardware support_ is needed for real protection!

### Dynamic (Hardware-based) Relocation

* Required hardware registers: __base__ register + __bounds__(also called __limit__) register, they are called __memory management unit(MMU)__

* Translation: $ address_{physical} = address_{virtual} + base $
* Happens at __run time__, that's why it's called "dynamic"

### Requirement for Dynamic Relocation

| Hardware Requirements                                        | Notes                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Privileged mode                                              | Needed to prevent user-mode processes from executing privileged operations |
| Base/bounds registers                                        | Need pair of registers per CPU to support address translation and bounds checks |
| Ability to translate virtual addresses and check if within bounds | Circuitry to do translations and check limits; in this case, quite simple |
| Privileged instruction(s) to update base/bounds              | OS must be able to set these values before letting a user program run |
| Privileged instruction(s) to register + exception handlers   | OS must be able to tell hardware what code to run if exception occurs |
| Ability to raise exceptions                                  | When processes try to access privileged instructions or out-of-bounds memory |

| OS Requirements        | Notes                                                        |
| :--------------------- | :----------------------------------------------------------- |
| Memory management      | Need to allocate memory for new processes; Reclaim memory from terminated processes; Generally manage memory via free list |
| Base/bounds management | Must set base/bounds properly upon context switch            |
| Exception handling     | Code to run when exceptions arise; likely action is to terminate offending process |

### Internal Fragmentation

Code, stack and heap size are smaller than the allocated unit, cause memory wasted

## Segmentation

* 2 bits for segmentation, 12 bits for offset

```c
1 // get top 2 bits of 14-bit VA
2 Segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT
3 // now get offset
4 Offset = VirtualAddress & OFFSET_MASK
5 if (Offset >= Bounds[Segment])
6 	RaiseException(PROTECTION_FAULT)
7 else
8 	PhysAddr = Base[Segment] + Offset
9 	Register = AccessMemory(PhysAddr)
```

* 1 bit to indicate if "grows positive"
* __Protection bits__ to indicate if read / execute / write

### External Fragmentation

The process’s memory request cannot be fulfilled because the memory are in many "little holes", which cause waste of memory

