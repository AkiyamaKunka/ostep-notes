# Notes on _Operating Systems : Three Easy Pieces_

Author: Akiyama

Create : 13 Feb, 2022

Update: 25 Mar, 2022

# Visualization

First part of Operating System: The Three Easy Pieces

## Process

//TODO

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
// get top 2 bits of 14-bit VA
Segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT
// now get offset
Offset = VirtualAddress & OFFSET_MASK
if (Offset >= Bounds[Segment])
	RaiseException(PROTECTION_FAULT)
else{
  PhysAddr = Base[Segment] + Offset
	Register = AccessMemory(PhysAddr)
}	
```

* 1 bit to indicate if "grows positive"
* __Protection bits__ to indicate if read / execute / write

### External Fragmentation

The process’s memory request cannot be fulfilled because the memory are in many "little holes", which cause waste of memory



# Concurrency

Second part of Operating System: The Three Easy Pieces

## Thread

* Like a separate process, but __share the same address space__ and can __access same data__
* __Thread Control Blocks__ store the sate of each thread of a process, when context switch occurs
* Each thread has a __stack__, but it will be __small__

## Lock

### Evaluation of Locks

* Fairness
* Correct Mutual Exclusion
* Performance

### Build Lock: Controlling interrupts

```c
1 void lock() {
2 DisableInterrupts();
3 }
4 void unlock() {
5 EnableInterrupts();
6 } 
```

Drawbacks:

* Do not support multiple-processors
* Need give priviledge to dreedy program, they may call lock() forever
* Lost data when "turn off interupts", e.g. Disk read data and ready to interupt to store them

* Low performance for CPU to "turn on/ off interrupts"

Advantage:

* It's ok for OS itself to do so in some cases, no security problem

###  Test-And-Set 

__Atomic exchange__ instruction from hardware to build __mutual exclusion primitive__:

```c
1 int TestAndSet(int *old_ptr, int new) {
2 int old = *old_ptr; // fetch old value at old_ptr
3 *old_ptr = new; // store 'new' into old_ptr
4 return old; // return the old value
5 } 
```

This instruction is supportted by almost all systems now

A simple __spin lock__

```c
1 typedef struct lock_t {
2 int flag;
3 } lock_t;
4
5 void init(lock_t *lock) {
6 // 0 indicates that lock is available, 1 that it is held
7 lock->flag = 0;
8 }
9
10 void lock(lock_t *lock) {
11 while (TestAndSet(&lock->flag, 1) == 1)
12 ; // spin-wait (do nothing)
13 }
14
15 void unlock(lock_t *lock) {
16 lock->flag = 0;
17 } 
```

* Correct on mutual exclusion
* No fairness
* May result bad performance: 1 thread get lock, N - 1 threads spin for a tick to wait for lock

### Compare-And-Swap/Exchange

Instruction provided by _SPARC and x86_

```c
1 int CompareAndSwap(int *ptr, int expected, int new) {
2 int actual = *ptr;
3 if (actual == expected)
4 	*ptr = new;
5 return actual;
6 } 
```

and replace `lock()`function above

```c
1 void lock(lock_t *lock) {
2 while (CompareAndSwap(&lock->flag, 0, 1) == 1)
3 ; // spin
4 } 
```

__Test-And-Set__  : assign value of `*prt` in each call

__Compare-And-Swap__ : assgin value of `*prt` only for 1 time

### Load-Linked and Store-Conditional

Instruction provided by _ARM, PowerPC, Alpha_

```c
1 int LoadLinked(int *ptr) {
2 return *ptr;
3 }
4
5 int StoreConditional(int *ptr, int value) {
6 if (no one has updated *ptr since the LoadLinked to this address) { 
7 	*ptr = value;
8 	return 1; // success!
9 } else {
10 	return 0; // failed to update
11 }
12 }
```

Lock Implemenation:

```c
1 void lock(lock_t *lock) {
2 while (1) {
3 	while (LoadLinked(&lock->flag) == 1)
4 	; // spin until it's zero
5 	if (StoreConditional(&lock->flag, 1) == 1)
6 		return; // if set-it-to-1 was a success: all done
7 	// otherwise: try it all over again
8 	}
9 }
10
11 void unlock(lock_t *lock) {
12 lock->flag = 0;
13 } 
```

### Fetch-And-Add (Ticket Lock)

Intruction by Mellor-Crummey & Michael Scott

```C
1 int FetchAndAdd(int *ptr) {
2 int old = *ptr;
3 *ptr = old + 1;
4 return old;
5 }
```

Lock Implementation:

```C
1 typedef struct lock_t {
2 int ticket;
3 int turn;
4 } lock_t;
5
6 void lock_init(lock_t *lock) {
7 lock->ticket = 0;
8 lock->turn = 0;
9 }
10
11 void lock(lock_t *lock) {
12 int myturn = FetchAndAdd(&lock->ticket);
13 while (lock->turn != myturn)
14 ; // spin
15 }
16
17 void unlock(lock_t *lock) {
18 FetchAndAdd(&lock->turn);
19 } 
```

__Advantage__: it ensures all thread which get the ticket can be scheduled, imporved __starvtation__ problem

### Improvement: Using Queues + Sleep

__Neither spin or yield cannot prevent waste of CPU or starving__ , Solaris provide `park()` and `unpark()` to implement 

* Improve __startvation__ by using __queue__

* Save __CPU waste__ from freqeut context swtich by make the thread to __sleep__
* Implementation highlight: It set __lock__ to _flag_ part in `lock()` function

```c
1 typedef struct lock_t {
2 int flag;
3 int guard;
4 queue_t *q;
5 } lock_t;
6
7 void lock_init(lock_t *m) {
8 m->flag = 0;
9 m->guard = 0;
10 queue_init(m->q);
11 }
12
13 void lock(lock_t *m) {
14 while (TestAndSet(&m->guard, 1) == 1)
15 ; //acquire guard lock by spinning 
16 if (m->flag == 0) {
17 m->flag = 1; // lock is acquired
18 m->guard = 0;
19 } else {
20 queue_add(m->q, gettid());
21 m->guard = 0;
22 park();
23 }
24 }
25
26 void unlock(lock_t *m) {
27 while (TestAndSet(&m->guard, 1) == 1)
28 ; //acquire guard lock by spinning
29 if (queue_empty(m->q))
30 m->flag = 0; // let go of lock; no one wants it
31 else
32 unpark(queue_remove(m->q)); // hold lock (for next thread!)
33 m->guard = 0;
34 } 
```

_Linux_ use `futex` interface to implement similar things

## Condition Variables

What if threads want to check whether a __condition__ is true before continuing its execution? E.g. a parent thread want to wait its child thread to be done.

 We introduce __condition variables__:

* It's a queue
* Thread can add them into it when condition is not satified
* Other thread can change the condition, can wake thread waiting on the condtion by sending signal

### Implement `join()`

You can declare a condition variable :`pthread_cond_t c;`

There' re two operation `wait()` and `signal()` for condition varaible:

```c
pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m);
pthread_cond_signal(pthread_cond_t *c); 
```

_POSIX_ calls look like:

```c
1 int done = 0;
2 pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
3 pthread_cond_t c = PTHREAD_COND_INITIALIZER;
4
5 void thr_exit() {
6 Pthread_mutex_lock(&m);
7 done = 1;
8 Pthread_cond_signal(&c);
9 Pthread_mutex_unlock(&m);
10 }
11
12 void *child(void *arg) {
13 printf("child\n");
14 thr_exit();
15 return NULL;
16 }
17
18 void thr_join() {
19 Pthread_mutex_lock(&m);
20 while (done == 0)
21 	Pthread_cond_wait(&c, &m);
22 Pthread_mutex_unlock(&m);
23 }
24
25 int main(int argc, char *argv[]) {
26 printf("parent: begin\n");
27 pthread_t p;
28 Pthread_create(&p, NULL, child, NULL);
29 thr_join();
30 printf("parent: end\n");
31 return 0;
32 } 
```

* `wait()` will assume thread is holding the lock `mutex` when it's called, then release `mutex`

* State variable`done` is the key that stores the information that the threads want to know. Like if it's the right time for parent thread to `wait()` in this example.

* Always hold the lock when sending `signal()`to prevent race condition, and stay healthy :)

### Use `while` but not `if`

* Mesa semantics: refer to "sending signal to thread only means condition changed, but no gaurantee it's the expect value before the thread's execution "
* Hoare semantics: The contrast of Mesa semanics

Thus, we always use `while(condition)`to replace `if(condition)` to check if condition is expected under Mesa semantics (because all computer systems are built under Mesa semantics)

### The Producer/Consumer Problem

* 2 __condtions__: `empty` is the queue where __producers__ sleep; `fill` is the queue where `consumer` sleep and wait to consume. When `count  == 0` which means no data in buffer, then thread send signal to `empty` to wake up __producer__. Vice Versa.

* Why we have 2 __condtions__? because __consumer__ should only wake up __producer__, vice versa. __Condition__ conceptually is like a queue, and _decide which specific thread should wake up another specific thread_

`put()` and `get()` functions for data access:

```c
1 int buffer[MAX];
2 int fill = 0;
3 int use = 0;
4 int count = 0;
5
6 void put(int value) {
7 buffer[fill] = value;
8 fill = (fill + 1) % MAX;
9 count++;
10 }
11
12 int get() {
13 int tmp = buffer[use];
14 use = (use + 1) % MAX;
15 count--;
16 return tmp;
17 } 
```

Final Impelementation:

```c
1 cond_t empty, fill;
2 mutex_t mutex;
3
4 void *producer(void *arg) {
5 int i;
6 for (i = 0; i < loops; i++) {
7 Pthread_mutex_lock(&mutex); // p1
8 while (count == MAX) // p2
9 	Pthread_cond_wait(&empty, &mutex); // p3
10 put(i); // p4
11 Pthread_cond_signal(&fill); // p5
12 Pthread_mutex_unlock(&mutex); // p6
13 }
14 }
15
16 void *consumer(void *arg) {
17 int i;
18 for (i = 0; i < loops; i++) {
19 Pthread_mutex_lock(&mutex); // c1
20 while (count == 0) // c2
21 	Pthread_cond_wait(&fill, &mutex); // c3
22 int tmp = get(); // c4
23 Pthread_cond_signal(&empty); // c5
24 Pthread_mutex_unlock(&mutex); // c6
25 printf("%d\n", tmp);
26 }
27 } 
```

### Covering Condition

Lampson and Redell use only 1 condition in producer/consumer problem, but they __wake up all__ sleeping threads to check which one is need. If a thread is not need, it go back to sleep imediately.

### Seemaphores

It's the single primitive for all things related to synchronization. We can use it as both locks and condtion varables. Invited by __Edsger Dijkstra__

