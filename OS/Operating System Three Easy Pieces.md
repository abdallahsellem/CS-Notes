
## Concurrency chapter :
> Computers are hard enough to understand without concurrency. Unfortunately, with concurrency, it simply gets worse. Much worse

- **objdump** , **valgrind** ,**purify** are disassembler tools which get the assembly code from an .exe code 
- valgrind is a powerful tool to trace the concurrency problems in exe files (like deadlock and ....etc)
	- how to use : 
	- first build the file (if the code in C , just make write make in a folder of makefile or write `gcc -o "filename" "filename".c)`
	- then write 
	- `sudo valgrind --tool=helgrind ./"filename"`

### Thread vs Process :

- **thread** is very much like a separate process but :
	- **threads**  within a process share the same address space and thus can access the same data 
	- **process** can't share the same data as each process has its own address space 
- each thread has a **program** **counter** **(PC)**  and has its own private set of registers it uses for computation 
-  **context switch** must take place if we want to switch between two threads 
- there is a thread control blocks (**TCBs**) to store the state of each thread of a process
- in  **context switch** between threads the address space remains the same (i.e., there is no need to switch which page table we are using).
- **Single-Threaded**  Process has one stack 
- **Multi-Threaded** has multiple stacks 
- drawback of **Multi-Threaded** is that **heap** and **stack** become smaller  
	- **Multi-threading** will be a problem in programs that make heavy use of recursion
#### Why using Threads :
1. parallelism: 
	- when we have multiple CPUs and we want to make execution of a certain program faster by making each thread do a job
2. avoid blocking program progress due to slow I/O 
	- many modern server-based applications (web servers, database management systems, and the like) make use of threads in their implementations.
**in either of the cases mentioned above, you could use multiple processes instead of threads. However, threads share an address space and thus make it easy to share data, and hence are a natural choice when constructing these types of programs. Processes are a more sound choice for logically separate tasks where little sharing of data structures in memory is needed**



### Multi-Threading Problems :
```
  
#include <stdio.h>  
#include <pthread.h>  
  
static volatile int counter = 0;  
  
// Simply adds 1 to counter repeatedly, in a loop  
// No, this is not how you would add 10,000,000 to  
// a counter, but it shows the problem nicely.  
//  
void *mythread(void *arg) {  
    printf("%s: begin\n", (char *) arg);  
    int i;  
    for (i = 0; i < 1e7; i++) {  
        counter = counter + 1;  
    }  
    printf("%s: done\n", (char *) arg);  
    return NULL;  
} 
// Just launches two threads (pthread_create)  
// and then waits for them (pthread_join)  
int main(int argc, char *argv[]) {  
    pthread_t p1, p2;  
    printf("main: begin (counter = %d)\n", counter);  
    pthread_create(&p1, NULL, mythread, "A");  
    pthread_create(&p2, NULL, mythread, "B");  
// join waits for the threads to finish  
    pthread_join(p1, NULL);  
    pthread_join(p2, NULL);  
    printf("main: done with both (counter = %d)\n", counter);  
    return 0;  
  
}
```
- this code expect to output **2000000** but its really output a diffrenet number for each execution and this is because the two threads try to increase the same counter by one and if one thread increased the counter by any number and the other thread wants to increase the counter it will have an older version of the counter as each thread take a copy of the counter value or address to increase resulting to unpredictable Increase **(for more information see page 305)**
- **A critical section** is a piece of code that accesses a shared variable (or more generally, a shared resource) and must not be concurrently executed by more than one thread .
- **mutual exclusion**. This property guarantees that if one thread is executing within the critical section, the others will be prevented from doing so
- file systems use techniques such as **journaling** or copy- on-write in order to atomically transition their on-disk state, critical for operating correctly in the face of system failures
- **race condition** arises if multiple threads of execution enter the critical section at roughly the same time; both attempt to update the shared data structure, leading to a surprising (and perhaps undesirable) outcome.
- note that not all code that is multi-threaded uses the join routine. For example, a multi-threaded web server might create a number of worker threads, and then use the main thread to accept requests and pass them to the workers, indefinitely.
- we can use this two function to avoid deadlocks in C 
	- example :
	- 	```int pthread_mutex_trylock(pthread_mutex_t *mutex);
	int pthread_mutex_timedlock(pthread_mutex_t *mutex, struct timespec *abs_timeout); 
-  condition variable. Condition variables are useful when some kind of signaling must take place between threads, if one thread is waiting for another to do something before it can continue
```
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex); 
int pthread_cond_signal(pthread_cond_t *cond)
```
### Thread API Guide :
- **Keep it simple.** Above all else, any code to lock or signal between threads should be as simple as possible. Tricky thread interactions lead to bugs.
- **Minimize thread interactions**. Try to keep the number of ways in which threads interact to a minimum. Each interaction should be carefully thought out and constructed with tried and true approaches (many of which we will learn about in the coming chapters).
- I**nitialize locks and condition variables**. Failure to do so will lead to code that sometimes works and sometimes fails in very strange ways.
- **Check your return codes**. Of course, in any C and U NIX programming you do, you should be checking each and every return code, and it’s true here as well. Failure to do so will lead to bizarre and hard to understand behavior, making you likely to (a) scream, (b) pull some of your hair out, or (c) both.
- **Be careful with how you pass arguments** to, and return values from, threads. In particular, any time you are passing a reference to a variable allocated on the stack, you are probably doing something wrong.
- **Each thread has its own stack**. As related to the point above, please remember that each thread has its own stack. Thus, if you have a locally-allocated variable inside of some function a thread is executing, it is essentially private to that thread; no other thread can (easily) access it. To share data between threads, the values must be in the heap or otherwise some locale that is globally accessible.
- **Always use condition variables to signal between threads**. While it is often tempting to use a simple flag, don’t do it.

### Evaluating Locks :
- The first is whether the lock does its basic task, which is to provide mutual exclusion.Basically, does thelock work, preventing multiple threads from entering a critical section?
- The second is fairness. Does each thread contending for the lock get a fair shot at acquiring it once it is free? Another way to look at this is by examining the more extreme case: does any thread contending for the lock starve while doing so, thus never obtaining it?
- The final criterion is performance, specifically the time overheads added by using the lock


### Locks From Simple To Intermediate 
- first approach invented was to disable interrupts for a single processor system but it had many problems and doesn't work for multiprocessor computers 
```
void lock() {
DisableInterrupts();
}
void unlock() {
EnableInterrupts();
}
```
##### Locks by Single variable :
- with timely (untimely?) interrupts, we can easily produce a case where both threads set the flag to land both threads are thus able to enter the critical section
- second problem is the busy-waiting 
```
typedef struct __lock_t { int flag; } lock_t;

void init(lock_t *mutex) {
// 0 -> lock is available, 1 -> held
mutex->flag = 0;
}
void lock(lock_t *mutex) {
while (mutex->flag == 1) // TEST the flag
; // spin-wait (do nothing)
mutex->flag = 1;
// now SET it!
}
void unlock(lock_t *mutex) {
mutex->flag = 0;
}
```
##### Locks by test and set :
- still we have the problem of busy waiting 
- it works because if testAndSet function is atomically executed by OS , if a thread tried to change the flag and an interrupt happen lead to second thread take the execution the old value of the flag will be returned and thus the flag wont be settled by the two thread which lead to one thread be executed 

```
typedef struct __lock_t {
int flag;
} lock_t;

void init(lock_t *lock) {
// 0: lock is available, 1: lock is held
lock->flag = 0;
}

void lock(lock_t *lock) {
while (TestAndSet(&lock->flag, 1) == 1)
; // spin-wait (do nothing)
}

void unlock(lock_t *lock) {
lock->flag = 0;
}

```
##### Lock by Queues Test and Set , Yield and wake up  :
```

typedef struct __lock_t {
	int flag;
	int guard;
	queue_t *q;
} lock_t;
void lock_init(lock_t *m) {
	m->flag = 0;
	m->guard = 0;
	queue_init(m->q);
}
void lock(lock_t *m) {
	while (TestAndSet(&m->guard, 1) == 1)
	; //acquire guard lock by spinning
	if (m->flag == 0) {
	m->flag = 1; // lock is acquired
	m->guard = 0;
	} else {
	queue_add(m->q, gettid());
	m->guard = 0;
	park();
	}
}
void unlock(lock_t *m) {
	while (TestAndSet(&m->guard, 1) == 1)
	; //acquire guard lock by spinning
	if (queue_empty(m->q))
	m->flag = 0; // let go of lock; no one wants it
	else
	unpark(queue_remove(m->q)); // hold lock
	// (for next thread!)
	m->guard = 0;
}
```
1. **Interruptibility during lock acquisition/release:** The thread might be interrupted while acquiring or releasing the lock, causing other threads to spin-wait for it to run again. However, the time spent spinning is limited, and thus this approach may be reasonable.
    
2. **Ordering of guard release and park:** If the release of the guard lock were to come after the park operation, it would lead to undesirable consequences. Specifically, it could result in problematic scenarios due to timing issues, such as a thread potentially sleeping forever after being about to park.
    
3. **Flag not being reset when a thread is woken up:** The flag representing the lock state does not get set back to 0 when another thread is woken up. This might seem like a race condition, but it is a necessity to ensure smooth handover of the lock from one thread to another.
    
4. **Perceived race condition before park():** There's a perceived race condition in the solution just before the call to park(). A thread could be about to park, assuming it should sleep until the lock is no longer held. However, if another thread holding the lock gets scheduled at that time and releases the lock, the subsequent park by the first thread could lead to it sleeping forever.
    
5. **Solution using setpark():** The provided solution suggests using the setpark() system call to address the wakeup/waiting race. By indicating to the system that it is about to park, a thread can avoid sleeping forever if it gets interrupted and another thread calls unpark before park is actually called.

for point 4 and 5 we was able to solve it by adding 
```
queue_add(m->q, gettid());
setpark(); // new code
m->guard = 0;
park();
```
if the lock released before parked of the second thread it won't sleep for infinite as OS will know it's intend to park and will return immediately

### Data Structures with Mutex locks :
- Using many threads don't always lead to better performance 

#### Local and Global counters :




**Example**: Let's consider a web server application that needs to track the total number of requests served across multiple CPU cores. Each core handles incoming requests independently and maintains its own local counter to track the requests it serves. However, there is also a need to display the total number of requests served by the server accurately.

- **Without Local and Global Counters**: If we had only a single global counter shared among all cores, every request processed by any core would require acquiring and releasing a lock to update the global counter. This could lead to significant contention, especially during periods of high traffic, resulting in poor performance due to frequent lock contention and context switching.
    
- **With Local and Global Counters**: By using local and global counters, each core can increment its local counter without contention. This significantly reduces contention and improves scalability since each core operates independently. Meanwhile, periodic updates to the global counter (based on a threshold) ensure that the total count remains accurate, albeit with a slight delay.
- **how to know which structure is faster (more concurrent or simple ):**
	- build both alternatives (simple but less concurrent, and complex but more concurrent) and measure how they do. In the end, you can’t cheat on performance; your idea is either faster, or it isn’t.


#### **AVOID PREMATURE OPTIMIZATION (KNUTH’S L AW )**
> When building a concurrent data structure, start with the most basic approach, which is to add a single big lock to provide synchronized access. By doing so, you are likely to build a correct lock; if you then find that it suffers from performance problems, you can refine it, thus only making it fast if need be. As Knuth famously stated, “**Premature optimization is the root of all evil.**”




### condition variable :
This is introduction of another important synchronization primitive beyond locks: condition variables. By allowing threads to sleep when some program state is not as desired, CVs enable us to neatly solve a number of important synchronization problems, including the famous (and still important) producer/consumer problem, as well as covering conditions

- **TIP : ALWAYS HOLD T HE LOCK WHILE SIGNALING**
- **grep foo file.txt | wc -l. This example runs two processes concurrently; grep writes lines from file.txt with the string foo in them to what it thinks is standard output**
- **TIP : USE WHILE (NOT IF ) FOR CONDITIONS**
##### Producer Consumer :

Imagine one or more producer threads and one or more consumer threads. Producers generate data items and place them in a buffer; consumers grab said items from the buffer and consume them in some way.
This arrangement occurs in many real systems. For example, in a multi-threaded web server, a producer puts HTTP requests into a work queue (i.e., the bounded buffer); consumer threads take requests out of this queue and process them.





##### General Implementation of the Producer Consumer Solution :
```
int buffer[MAX];
int fill_ptr = 0;
int use_ptr = 0;
int count= 0;
cond_t empty, fill;
mutex_t mutex;

void put(int value) {
buffer[fill_ptr] = value;
fill_ptr = (fill_ptr + 1) % MAX;
count++;
}

int get() {
int tmp = buffer[use_ptr];
use_ptr = (use_ptr + 1) % MAX;
count--;
return tmp;
}
void *producer(void *arg) {
int i;
for (i = 0; i < loops; i++) {
Pthread_mutex_lock(&mutex); // p1
while (count == MAX) // p2
Pthread_cond_wait(&empty, &mutex); // p3
put(i); // p4
Pthread_cond_signal(&fill); // p5
Pthread_mutex_unlock(&mutex); // p6
}
}
void *consumer(void *arg) {
int i;
for (i = 0; i < loops; i++) {
Pthread_mutex_lock(&mutex);
while (count == 0)
Pthread_cond_wait(&fill, &mutex);
int tmp = get();
Pthread_cond_signal(&empty);
Pthread_mutex_unlock(&mutex);
printf("%d\n", tmp);
}
}
```

### Semaphores :

- semaphores always ensures that its variable value are non-negative so if its inilized with zero and sem_wait run , it will sleep the thread but its variable will still have value 0 

##### Semaphore as a Lock 
- initiate the semaphore value with 1 and this will make one thread only access the critical section 
##### semaphore as ordering (Condition Variable )
-  initiate the semaphore value with 0 and this will make the parent thread wait for the child thread 

##### semaphore as bonding buffer :
###### Wrong implementation lead to DeadLock See page 384 
```
sem_t empty;

sem_t full;

sem_t mutex;

void *producer(void *arg) {

int i;

for (i = 0; i < loops; i++) {

sem_wait(&mutex);

sem_wait(&empty);

put(i);

sem_post(&full);

sem_post(&mutex);

}
}
void *consumer(void *arg) {

int i;

for (i = 0; i < loops; i++) {

sem_wait(&mutex);

sem_wait(&full);

int tmp = get();

sem_post(&empty);

sem_post(&mutex);

printf("%d\n", tmp);

}

}
int main(int argc, char *argv[]) {

sem_init(&empty, 0, MAX); // MAX buffers are empty to begin with...

sem_init(&full, 0, 0);

sem_init(&mutex, 0, 1);
}
```

###### Working Solution :
- just replace the signal code and mutex lock together 
```
void *producer(void *arg) {
int i;
for (i = 0; i < loops; i++) {
sem_wait(&empty);
sem_wait(&mutex);
put(i);
sem_post(&mutex);
sem_post(&full);
}
}
void *consumer(void *arg) {
int i;
for (i = 0; i < loops; i++) {
sem_wait(&full);
sem_wait(&mutex);
int tmp = get();
sem_post(&mutex);
sem_post(&empty);
printf("%d\n", tmp);
}
}

```

### Reader Writer Problem 
its a problem happen when there are many readers want to read a buffer or a value and there are one writer or more , this may lead to reader may read a new value that's updated after the reader made the query , so we want to make when a reader made a query no writers could write to the same value that's read 

- a simple reader writer approach on **page 286** may
- lead to starvation of writers , writers may be suspended for ever as reader are reading 




### The Dining Philosophers Problem 

suppose there are 5 threads and each process want a resource on the left and resource on the right of it , now the problem arise when each thread take just the resource on the left of it , this will lead to deadlock as each process need two resources and all resources are taken 
![[Pasted image 20240306223952.png]]

#### solution : 
we will change how the last thread acquire the look by make it to acquire the right lock first then the left lock 
so how this will solve the problem 
process 0  will acquire resource 0  , process 1 will acquire resource 1 , ..................... until we will reach the last process , it will try to acquire the process 0 which is already acquired so this process will be put to sleep , so the process before it will be able to acquire the right resource avoiding deadlock
```
void getforks() {
if (p == 4) {
sem_wait(forks[right(p)]);
sem_wait(forks[left(p)]);
} else {
sem_wait(forks[left(p)]);
sem_wait(forks[right(p)]);
}
}
```


Dataset stats::
Normal :: 2000
Cataract :: 594
Diabetes :: 1818
Glaucoma :: 610
Hypertension :: 374
Myopia :: 479
Age Issues :: 551
Other :: 324

### Deadlocks:
##### Example : 
- Take the operating system, for example. The virtual memory system might need to access the file system in order to page in a block from disk; the file system might subsequently require a page of memory to read the block into and thus contact the virtual memory system. Thus, the design of locking strategies in large systems must be carefully done to avoid deadlock in the case of circular dependencies that may occur naturally in the code.
- Another reason is due to the nature of **encapsulation**. As software developers, we are taught to hide details of implementations and thus make software easier to build in a modular way. Unfortunately, such modularity does not mesh well with locking.
![[Pasted image 20240415035535.png]]

##### Conditions should be met to get a deadlock :
Four conditions need to hold for a deadlock to occur :
- **Mutual exclusion**: Threads claim exclusive control of resources that they require (e.g., a thread grabs a lock).
- **Hold-and-wait**: Threads hold resources allocated to them (e.g., locks that they have already acquired) while waiting for additional resources (e.g., locks that they wish to acquire).
- **No preemption**: Resources (e.g., locks) cannot be forcibly removed from threads that are holding them.
- **Circular wait**: There exists a circular chain of threads such that each thread holds one or more resources (e.g., locks) that are being requested by the next thread in the chain.



##### Avoid Circular Wait :
- t**otal ordering:** The most straightforward way to do that is to provide a total ordering on lock acquisition. For example, if there are only two locks in the system (L1 and L2), you can prevent deadlock by always acquiring L1 before L2. Such strict ordering ensures that no cyclical wait arises; hence, no deadlock
- **partial ordering:** An excellent real example of partial lock ordering can be seen in the memory map-ping code in Linux ; the comment at the top of the source code reveals ten different groups of lock acquisition orders.
###### Enforce Lock Ordering by Address :
```
In some cases, a function must grab two (or more) locks; thus, we know
we must be careful or deadlock could arise. Imagine a function that is
called as follows: do something(mutex t *m1, mutex t *m2). If
the code always grabs m1 before m2 (or always m2 before m1), it could
deadlock, because one thread could call do something(L1, L2) while
another thread could call do something(L2, L1).
To avoid this particular issue, the clever programmer can use the address
of each lock as a way of ordering lock acquisition. By acquiring locks in
either high-to-low or low-to-high address order, do something() can
guarantee that it always acquires locks in the same order, regardless of
which order they are passed in. The code would look something like this:
if (m1 > m2) { // grab locks in high-to-low address order
pthread_mutex_lock(m1);
pthread_mutex_lock(m2);
} else {
pthread_mutex_lock(m2);
pthread_mutex_lock(m1);
}
// Code assumes that m1 != m2 (it is not the same lock)
By using this simple technique, a programmer can ensure a simple and
efficient deadlock-free implementation of multi-lock acquisition.
```
##### Avoid Hold and Wait :
The hold-and-wait requirement for deadlock can be avoided by acquiring all locks at once, atomically. In practice, this could be achieved as follows:
```
pthread_mutex_lock(prevention);
// begin lock acquisition
pthread_mutex_lock(L1);
pthread_mutex_lock(L2);
...
pthread_mutex_unlock(prevention); // end
```
this approach is called **Lock a acquiring the Lock** 

##### Avoid No Preemption :
**No Preemption**: means that if  a thread hold a lock we can't force it to release the lock 
Many thread libraries provide a more flexible set of interfaces to help avoid this situation.Specifically, the routine pthread mutex trylock() either grabs the lock (if it is available) and returns success or returns an error code indicating the lock is held
```
top:
pthread_mutex_lock(L1);
if (pthread_mutex_trylock(L2) != 0) {
pthread_mutex_unlock(L1);
goto top;
}
```
in this code we hold the first lock but if we can't hold the second lock we release the first lock and try again 
this is called a ***dead-lock free program*** 
this solution of deadlock could lead to another problem called **Live-Lock** 
- Live-Lock : its a problem when two threads repeatedly trying to acquire the same locks but it fails , so this will lead in no progress of the program as the locks will be never held by anyone of them 
- we can solve this problem by adding **random delay** so at some moment of time , no two threads will try to held the locks at the same time 
- another problem will arises after using this solution is all locks or resources  acquired should be released upon the return to the top after using the ***goto*** so it will be hard to track and release all the resources 
this solution isn't adding preemption to the code , as thread itself unacquire their owned locks  => this approach doesn’t really add preemption(the forcible action of taking a lock away from a thread that owns it),but rather uses the trylock approach to allow a developer to back out of lock ownership (i.e., preempt their own ownership) in a graceful way.

##### Avoid Mutual Exclusion :
The final prevention technique would be to avoid the need for mutual exclusion at all. In general, we know this is difficult, because the code we wish to run does indeed have critical sections.The idea behind these lock-free (and related wait-free) approaches here is simple: using powerful hardware instructions, you can build data structures in a manner that does not require explicit locking.
As a simple example, let us assume we have a compare-and-swap instruction, which as you may recall is an atomic instruction provided by the hardware that does the following:
```
int CompareAndSwap(int *address, int expected, int new) {
if (*address == expected) {
*address = new;
return 1; // success
}
return 0; // failure
}
```
also this a code using a free-lock technique to increment the value :
```
void AtomicIncrement(int *value, int amount) {
do {
int old = *value;
} while (CompareAndSwap(value, old, old + amount) == 0);
}
```
also we can apply this on more complex operations like Insert element in a List :

- Code with Locks:
```
void insert(int value) {
node_t *n = malloc(sizeof(node_t));
assert(n != NULL);
n->value = value;
pthread_mutex_lock(listlock);
// begin critical section
n->next = head;
head
= n;
pthread_mutex_unlock(listlock); // end critical section
}
```
- code with Free-Lock :
```
void insert(int value) {
node_t *n = malloc(sizeof(node_t));
assert(n != NULL);
n->value = value;
do {
n->next = head;
} while (CompareAndSwap(&head, n->next, n) == 0);
} 
```
##### Detect and Recover
- One final general strategy is to allow deadlocks to occasionally occur, and then take some action once such a deadlock has been detected. For example, if an OS froze once a year, you would just reboot it .
- Many database systems employ deadlock detection and recovery techniques. A deadlock detector runs periodically, building a resource graph and checking it for cycles. In the event of a cycle (deadlock), the system needs to be restarted.
-

***Perhaps the best solution is to develop new concurrent programming models: in systems such as MapReduce (from Google)***

### Event-based Concurrency:
**Event-based Concurrency** : has become popular in some modern systems, including
server-side frameworks such as **node.js**
