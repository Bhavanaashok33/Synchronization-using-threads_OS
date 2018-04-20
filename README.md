1. EXECUTIVE SUMMARY
The following document presents the design and implementation of the producerconsumer
problem and provides a solution on a multi-process scenario. The producer
and consumer are two different processes who share a buffer that acts as a queue. The
producer has a task of adding the data into the buffer and the consumer holds the task of
removing the data from the buffer.
The whole solution revolves around addressing two scenarios
1. The producer should not add any data to the buffer, when the queue is full. While,
the consumer will avoid drawing the data from the buffer when the buffer is
empty.
2. The producer should sleep when the buffer is already full and discard the
additional data, if any. The consumer should sleep if and when the buffer is
empty.
If any situation arises wherein, the consumer draws any data from buffer, the producer
will wake up and would input the next set of data into the buffer.
2. INTRODUCTION
Producer consumer problem is a classic example of a multiprocessor synchronization
problem. The synchronization is required to make sure that that the producer stops
producing when the buffer is full and the consumer stop consuming when the buffer is
empty. The variations of the producer consumer problem can be implemented in various
applications [1], [2], [3], and can be run on both single and multiprocessor systems. In
this project we will discuss two different approaches to solve the producer consumer
synchronization problem and implement the solution. The first approach is the more
traditional approach using synchronization between producers and consumers with
higher-level algorithms like semaphores. The second implementation is to use an
4
approach that is similar to the solution of the problem using a ‘monitor’ with condition
variables to implement the producer and consumer operations. This method also helps
avoid issues due to spurious wakeups for full and empty buffer conditions.
3. BACKGROUND AND OBJECTIVES
The problem statement describes two processes, which share a common fixed size
buffer. The producer generates the data and puts it into the buffer. At the same time the
consumer consumes the data from the buffer. The problem is to make sure that the
producer won't try to add data into the buffer if it's full and that the consumer won't try to
remove data from an empty buffer.
The following can ensure the solution to the problem. The producer discards data or goes
to sleep when the buffer is full. It then waits for the consumer to consume an item (data)
and send notification to the producer to resume its process. Similarly, the consumer goes
to sleep when the buffer is empty and waits for a notification from producer. This can be
achieved through inter process communication. However, an inadequate solution can
result race condition and deadlock.
A simple solution would be to use the library routines sleep() and wakeup(). When
sleep() routine is called, the caller is blocked until another process wakes it up using the
wakeup routine. However, the wakeup() call gets discarded in case the process is not
sleeping. Since there is no synchronization among the processes, a special scenario may
arise where in a process might get a wakeup() call before going to sleep mode resulting
in both the processes going sleep simultaneously. This could lead to a deadlock.
Semaphores, mutexes or monitors can be used to overcome this flaw. [4]
5
Semaphores solve the problem of lost wakeup calls. Two semaphores can be used, one to
determine the number of items already in the buffer [fillCount] and the other to
determine the number of available spaces in the buffer where items could be written
[emptyCount]. The semaphore fillCount is incremented and emptyCount is decremented
when new item is put into buffer. If producer tries to decrement emptyCount when its
value is zero, the producer is put to sleep. The next time an item is consumed,
emptyCount is incremented and the producer wakes up. The consumer works
analogously. This works well for single consumer and producer. With multiple
producers or consumers sharing the same memory space for buffer, a race condition can
occur. To overcome the above problem, we need to ensure that only a single producer is
adding data to the buffer at any instance that is we need to execute critical section with
mutual exclusion.
One way of solving the above problem is to use three semaphores. We use one
semaphore to determine the number of items already in the buffer, one to determine the
number of available spaces in the buffer where items could be written and a mutex with
initial value 1. Mutex follows the ownership concept, that is it can only be incremented
back (set to 1) by the process that decremented it. All other process wait until mutex is
available for decrement. This ensures mutual exclusion. However, the order in which the
semaphores are incremented or decremented is important, as it could result in deadlock.
Another way of solving the problem is by using the monitors. It is an object consisting of
several monitor procedures where in only one thread is active at any time. This ensures
mutual exclusion. Since mutual exclusion is implicit in monitors no extra effort is
needed to protect the critical section.
6
In our project we are solving the producer consumer problem by pthreads through
condition variables in monitors. Here, the producer thread takes a lock, places a work
item in it, releases the lock and resumes the consumer thread. The consumer takes a lock
on the shared buffer, if the buffer is empty it releases the lock and hibernates. When
consumer thread awakens, it reacquires the lock and removes the item from buffer. The
producer can keep placing data on buffer even if the consumer hasn’t consumed it.
Keeping track of available space in buffer size through semaphores ensures this. This
solution is better than the above-discussed solutions. A mutex only allows you to wait
until the lock is available; a condition variable allows you to wait until some applicationdefined
condition has changed. This helps to avoid polling and busy waiting.
4. METHODOLOGY
The problem consists of various producer threads and consumer threads sharing a buffer
with multiple slots (storage cells); each producer deposits one element (the item it
produces) into one slot of the buffer each time it “produces”, and each consumer extracts
one element from the buffer each time it “consumes”. The buffer is treated as a circular
queue, with insertions into one end of the queue, and extractions from the queue from the
other end.
The traditional approach to solve the problem uses mutex (binary semaphore) objects to
provide mutual exclusion between the producers and consumers. Additionally, we need
to ensure that the producer is blocked when buffer is full and consumer is locked when
buffer is empty. Lock and sleep operations are used for this. In another approach, that is
the approach we are implementing, we use condition variables. The condition variables
are able to release the lock and sleep on the condition atomically. A single mutex is used
to lock access to the buffer. Each consumer and each producer has a loop used to control
waiting while a buffer is empty (consumer) and while a buffer is filled (producer).
7
Within the loop is the semaphore control. This loop implementation is also used to
compensate for those spurious wakeups.
We begin our development by solving one producer and one consumer problem with a
shared buffer and then build the full solution.
1. SHARED BUFFER
The first thing we need in a producer consumer scenario is the shared buffer in
which the producer puts the data and out of which the consumer takes the data. In
our program we consider the buffer as a queue. We use the put() and get() function
to store and remove values from the buffer. We can either store pointers or
integers with multiple buffer slots in the buffer.
2. PRODUCER
A producer only sleeps if all buffers are currently filled. We use two condition
variables to wait on the buffer. One condition variable [emptyCount] determines
the number of buffer slots that are empty. The other variable [fillCount]
determines the number of buffer slots that are full. The “emptyCount” variable is
used to wait on the producer. The initial value of this variable is the maximum
buffer size, it is decremented every time the producer adds data to buffer and
incremented every time the consumer consumes data. The producer is blocked
when the value reaches zero and signaled to wake when value increments to one.
3. CONSUMER
A consumer sleeps only when the buffer is empty. The condition variable
“fillCount” is used to wait on the consumer. The initial value of this variable is
zero and increments every time the producer adds an item (produces data) into the
buffer. The consumer is blocked whenever the value of the condition variable
“fillCount” is zero as the buffer is empty at that point.
8
5. FINDINGS AND ANALYSIS
Findings
- The Producer will populate the buffer and sleep when the buffer is full and the
consumer will empty the buffer and sleep when the buffer is empty.
- Producer and consumer each run on a different threads.
- The flow chart shown describes the flow of the process in the producer consumer
problem
9
Shown below is a screenshot of the output for a single producer consumer in the program
fig 1 :: case where buffer is full and producer goes to sleep
fig 2 :: case where buffer is empty and consumer goes to sleep
Analysis:
- Set of threads use a mutex lock and condition variables to allow serial access to a
critical region.
- The locks and condition variables ensure synchronization among the processes
and threads
- Each thread has its own stack and local variables.
- Global variables are shared.
10
- In summary, we need a condition variables to automatically block producers when
the buffer is full, and block consumers when the buffer is empty. Note that the
first condition variable is signaled (by a consumer) when the buffer is not full, and
the condition variable is signaled (by a producer) when the buffer is not empty.
- What are the initial values?One condition variable (empty) is initialised to max
buffer size and the other(fill) is initialised to 0 . Why? The empty variable
determines the number of spaces available which is initially maximum. The
producer goes to sleep when it reaches zero. Similarly, the fill variable determines
the number of spaces filled. Consumer is in sleeping state when it is zero
6. Conclusion and recommendation
Conclusion
- The producer consumer synchronization problem was solved using condition
variables. Program was written for a single producer consumer and later
generalized for multiple producer consumer
- The efficiency raises with around 30% from single Producer Consumer to using
multi producer and consumer. The multi-threading will help in many producers
and consumers be involved in the process simultaneously. This helps in efficient
and faster transfer of data.
