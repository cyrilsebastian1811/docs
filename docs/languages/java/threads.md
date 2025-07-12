# __Multithreading__

## Monitor Lock and Object Synchronization

- Every Java object has an intrinsic monitor lock(also called a monitor).
- A thread must acquire the monitor lock of an object before it can execute any `#!java synchronized` block or method associated with that object.
- Without synchronization, the thread does not hold the lock, and the JVM will throw an IllegalMonitorStateException.
- Monitor lock is released either when synchronized block/method completes, or an exception is thrown insided the `#!java synchronized` block/method.
- If a thread attempts to acquire a lock that is already held by another thread, it enters the __Blocked__ state until the lock becomes available.
- __Automatic vs. Manual__:
    - Monitor locks are implicitly acquired and released with `#!java synchronized`.
    - Explicit locks like `#!java ReentrantLock` require manual locking and unlocking.

### Synchronization scopes

- Method-Level Synchronization:
    - The lock is held for the entire duration of the method execution.
    ```{.java}
    public synchronized void increment() {
        count++;
    }
    ```
- Block-Level Synchronization:
    - The lock is held only for the specific block of code within `#!java synchronized`.
    - The lock is acquired on the __monitor of the object (this)__ when entering the block.
    - You can acquire lock on other objects as well.
    ```{.java .copy hl_lines="3 6 13"}
    public class Counter {
        private int count = 0;
        private final Object lock = new Object(); // Separate lock object

        public void increment() {
            synchronized (lock) { // Locking on a dedicated object
                count++;
            }
        }

        public int getCount() {
            // Not helpful, as locks acquired on 2 different objects
            synchronized (this) {
                return count;
            }
        }
    }
    ```

### Why is the lock required?

The lock ensures mutual exclusion and proper coordination between threads. Here's why the lock is necessary:

- __Preventing Race Conditions__:
    - `#!java wait()` releases the lock temporarily, allowing another thread to acquire it.
    - Without synchronization, multiple threads might access and modify shared resources concurrently, leading to race conditions.
- __Ensuring Consistency__:
    - A thread must hold the lock to `#!java wait()` because only the thread that holds the lock can "coordinate" with other threads using `#!java notify()` or `#!java notifyAll()`.
- __Thread Communication__:
    - `#!java wait()` makes the thread temporarily stop execution and release the lock so another thread (e.g., a producer) can access the shared resource.
    - Once `#!java notify()` or `#!java notifyAll()` is called, the waiting thread reacquires the lock before resuming execution.

Without a synchronized block, proper communication and coordination between threads would break down.

## Producer Consumer problem

The Producer-Consumer Problem is a classic synchronization problem in multithreading where two types of threads work together:

- __Producer__: Produces data and adds it to a shared resource (e.g., a buffer).

- __Consumer__: Consumes data from the shared resource.

To ensure proper operation, synchronization is required so that:

- The producer does not add data if the buffer is full.
- The consumer does not remove data if the buffer is empty.

=== "`#!java wait()`"

    - Causes the current thread to release the lock it holds and wait until another thread calls `#!java notify()` or `#!java notifyAll()`.
    - The thread moves into the WAITING state.
    - Must be called from within a synchronized block or method; otherwise, it throws IllegalMonitorStateException

=== "`#!java notify()`"

    - Wakes up one thread that is waiting on the object's monitor (lock).
    - If multiple threads are waiting, only one thread is notified randomly.
    - The awakened thread will continue execution only after the notifying thread releases the lock.
    - `#!java notify()` does not immediately release the lock.
    - Other threads cannot acquire the lock until the current thread finishes execution of the synchronized block/method or enters waiting state.

=== "`#!java notifyAll()`"

    - Wakes up all threads waiting on the object's monitor.
    - However, only one thread will acquire the lock (as only one thread acquires monitor lock at a time), and others will keep waiting.

=== "`#!java yield()`"

    - Hints to the thread scheduler that the current thread is willing to pause and allow other threads of the same or higher priority to execute.
    - It does not release locks held by the thread.
    - The thread might immediately resume execution if no other thread is ready to run.

=== "`#!java join()`"

    - Allows one thread to wait for the completion of another thread.
    - If `#!java t.join()` is called, the current thread pauses until thread `#!java t` completes its execution.
    - You can also specify a timeout with `#!java join(milliseconds)` to limit the wait time.

=== "`#!java sleep()`"

    - Puts the current thread into a timed waiting state for a specified number of milliseconds or nanoseconds.
    - ==Does not release locks held by the thread.==
    - After the sleep duration, the thread moves to the Runnable state and waits for CPU time.


``` java title="Producer-Consumer example" hl_lines="11 16 21 22"
import java.util.LinkedList;

class Buffer {
    private LinkedList<Integer> list = new LinkedList<>();
    private int capacity;

    public Buffer(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void produce() throws InterruptedException {
        int value = 0;
        while (true) {
            while (list.size() == capacity) { // (1)!
                // Buffer is full; wait for the consumer
                wait();
            }
            System.out.println("Producer produced: " + value);
            list.add(value++);
            Thread.sleep(1000); // Simulate production time
            // Notify the consumer
            notify();
        }
    }

    public synchronized void consume() throws InterruptedException {
        while (true) {
            while (list.isEmpty()) {
                // Buffer is empty; wait for the producer
                wait();
            }
            int value = list.removeFirst();
            System.out.println("Consumer consumed: " + value);
            Thread.sleep(1000); // Simulate consumption time
            // Notify the producer
            notify();
        }
    }
}

public class ProducerConsumer {
    public static void main(String[] args) {
        Buffer buffer = new Buffer(5);

        Thread producerThread = new Thread(() -> {
            try {
                buffer.produce();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        Thread consumerThread = new Thread(() -> {
            try {
                buffer.consume();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        producerThread.start();
        consumerThread.start();
    }
}
```

1. In multithreading with wait()/notify(), while is used to re-check the condition after the thread is awakened. This is necessary because:

    - __Spurious wakeups happen__: Threads waiting on wait() can sometimes wake up without being notified, due to OS-level behavior.
    - __Race conditions on shared state__: When multiple threads are waiting or notifying, the condition may have changed by the time a thread wakes up.
    - __`if()` only checks once__: Using `if()` assumes the condition is still valid after waking up — this is unsafe.

!!! warning

    - __Deadlock__: Threads are stuck waiting for each other and can never proceed.
    - __Livelock__: Threads keep responding to each other in a way that prevents progress (e.g., retrying actions indefinitely).
    - __Starvation__: A thread waits indefinitely because higher-priority threads are always favored.


## Threads

- ==Every Java program starts with a main thread. It begins when the JVM calls your main() method.==
- A thread is the smallest unit of execution in a program. Java allows multithreading.
- Threads in Java are managed by the JVM and the underlying OS.
- You can create threads in 3 main ways:

=== "`Thread` Class"

    ```java hl_lines="1 3 11"
    class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("Thread running: " + Thread.currentThread().getName());
        }
    }

    public class Main {
        public static void main(String[] args) {
            MyThread t1 = new MyThread();
            t1.start();  // Starts the new thread
        }
    }
    ```

=== "`Runnable` Interface"

    - ==Allows extending another class (since Java supports single inheritance).==

    ```java hl_lines="1 3 11"
    class MyRunnable implements Runnable {
        @Override
        public void run() {
            System.out.println("Thread running: " + Thread.currentThread().getName());
        }
    }

    public class Main {
        public static void main(String[] args) {
            Thread t1 = new Thread(new MyRunnable());
            t1.start();
        }
    }
    ```

=== "Lambda Expression"

    ```java hl_lines="3-6"
    public class Main {
        public static void main(String[] args) {
            Thread t1 = new Thread(() -> {
                System.out.println("Thread running: " + Thread.currentThread().getName());
            });
            t1.start();
        }
    }
    ```

### Components

| Component                | Description                                      |
| ------------------------ | ------------------------------------------------ |
| __Thread Stack__         | Stores method calls and local variables          |
| __Program Counter (PC)__ | Keeps track of which instruction to execute next |
| __Thread Object__        | Contains thread ID, name, priority, etc.         |


### Lifecycles

| State                     | Description                           |
| ------------------------- | ------------------------------------- |
| __New__                   | Thread is created but not started     |
| __Runnable__              | Ready to run, waiting for CPU         |
| __Running__               | Actively executing                    |
| __Blocked__               | Waiting for a lock                    |
| __Waiting/Timed Waiting__ | Waiting for another thread or timeout |
| __Terminated__            | Thread has completed or crashed       |


### Memory Allocation in Multithreading

- Java threads operate on the Java Memory Model (JMM), which defines how memory is shared between threads.

| Area            | Shared?      | Description                             |
| --------------- | ------------ | --------------------------------------- |
| __Heap__        | ✅ Shared     | All objects, class instances live here  |
| __Stack__       | ❌ Not shared | Each thread has its __own stack__       |
| __Method Area__ | ✅ Shared     | Stores class metadata, static variables |


### Multithreading Terms

| Term                | Description                                              |
| ------------------- | -------------------------------------------------------- |
| __Concurrency__     | Multiple threads make progress independently             |
| __Parallelism__     | Threads truly run in parallel (on multi-core CPUs)       |
| __Race Condition__  | Two threads accessing shared data without sync           |
| __Thread Safety__   | Code behaves correctly when accessed by multiple threads |
| __Synchronization__ | Controls access to shared resources                      |

### Thread Priorities

- Each thread in Java has a priority, which is an integer value between 1 (lowest) and 10 (highest). 
- Used by the thread scheduler (provided by the JVM and OS) to decide which thread to run next, especially when multiple threads are competing for CPU time.
- Priority Constants:
    - `Thread.MIN_PRIORITY`: 1
    - `Thread.NORM_PRIORITY`: 5
    - `Thread.MAX_PRIORITY`: 10

```java
Thread t1 = new Thread(() -> {
    System.out.println("Running Thread 1");
});

t1.setPriority(Thread.MAX_PRIORITY);  // Set high priority
// Or custom values
t1.setPriority(7);
t1.start();
```


### Thread Pools