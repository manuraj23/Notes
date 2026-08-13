# **Java Notes**

# **Threading**

## **Multi-Tasking**
Multi-tasking means performing multiple tasks simultaneously.

Types: 
  1. Process-based Multi-Tasking: Multiple programs running simultaneously. eg: vscode, chrome, spotify etc. Each program has seperate memory.
  2. Thread-based Multi-Tasking: Single Application, multiple threads. eg: Chrome: Main thread, download thread, UI thread etc.

### Difference between Process, Program, and Thread

#### Program
- Set of instructions/code
- Passive entity
- Stored on disk
- Does not execute by itself
- Becomes a process when executed

#### Proces
- A program in execution
- Heavy Weight
- Has its own memory space
- Slower compared to threads
- Creation is costly

#### Thread
- Smallest unit of execution within a process
- Light Weight
- Shares memory with other threads of the same process
- Faster compared to processes
- Creation is relatively cheap

### The Main Thread
- The JVM automatically creates the **main thread** when a Java program starts.
- We can get the reference of the currently executing thread using `Thread.currentThread()`.

example:

```java
public static void main(String[] args) {
    Thread t = Thread.currentThread();

    System.out.println(t.getName());
}
```
## Thread Creation in Java

There are two common ways to create a thread in Java:

1. **Extending the `Thread` class**
2. **Implementing the `Runnable` interface**

### Extending the `Thread` Class
- We can create a thread by creating a class that **extends the `Thread` class**.
- We override the `run()` method to define the task that the thread should perform.
- To start the thread, we call the `start()` method.
- The `start()` method creates a new thread and internally calls the `run()` method.

#### Example

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("Thread is running...");
    }
}

public class Main {
    public static void main(String[] args) {

        MyThread t = new MyThread();

        t.start();
    }
}
```

#### How it works
`MyThread extends Thread` → creates a custom thread class.

`run()` → contains the code that the thread will execute.

`new MyThread()` → creates a thread object.

`start()` → starts a new thread and executes run().

`Important`: Do not directly call run() when you want to create a new thread.

```java
t.run();    // Normal method call, no new thread
t.start();  // Creates and starts a new thread
```

### Why use `start()` instead of `run()`?
`run()` is just a normal method.

`start()` tells the JVM to create a new thread.

The newly created thread then executes the `run()` method.

#### Limitation
- Java does not support multiple class inheritance.
- If a class extends Thread, it cannot extend another class.

```java
class MyThread extends Thread {
    // Cannot extend another class
}
```
- Because of this limitation, implementing `Runnable` is generally more flexible.

## Implementing Runnable Interface
- Another way to create a thread is by **implementing the `Runnable` interface**.
- The `Runnable` interface contains a single abstract method:

```java
void run();
```

- We implement the `run()` method to define the task that the thread should perform.
- Then, we create a Thread object and pass the Runnable object to its constructor.
- Finally, we call `start()` to start the thread.

```java
class MyTask implements Runnable {

    @Override
    public void run() {
        System.out.println("Thread is running...");
    }
}

public class Main {
    public static void main(String[] args) {

        MyTask task = new MyTask();

        Thread t = new Thread(task);

        t.start();
    }
}
```
### How it works
- `MyTask implements Runnable` → creates a class that defines the task.
- `run()` → contains the code that the thread will execute.
- `MyTask task = new MyTask()` → creates the task object.
- `Thread t = new Thread(task)` → creates a Thread object and associates the task with it.
- `t.start()` → starts a new thread, which executes the `run()` method.


### Using Lambda Expression

Since Runnable is a functional interface, we can use a lambda expression:

```java
Runnable task = () -> {
    System.out.println("Thread is running...");
};

Thread t = new Thread(task);
t.start();
```
### Advantage over Extending Thread
- Java supports single inheritance.
- If we extend Thread, our class cannot extend any other class.
- With Runnable, our class can extend another class while still being able to define a task for a thread.

```java
class MyClass extends SomeClass implements Runnable {

    @Override
    public void run() {
        System.out.println("Thread is running...");
    }
}
```

#### Important: `Runnable` represents the task, while `Thread` represents the execution thread that runs that task.

## Thread Life-Cycle

```java
New --> Runnable --> Running --> Blocked/Waiting --> Runnable --> Terminated
```
## Thread Methods
- `start()` → Starts a new thread and causes the JVM to execute its `run()` method.
- On Calling `start()`  twice on same thread, we get error of `IllegalThreadSafeException`

- We can not start() a thread more than once simultaneously.
#
- `run()` → Contains the code/task that the thread executes. Calling `run()` directly does **not** create a new thread.
#
- `sleep()` → Pauses the currently executing thread for a specified amount of time.
- used in `try-catch block`, otherWise use `throws Interrupted Exception`

```java
public class SleepTryCatchExample {
    public static void main(String[] args) {
        System.out.println("Execution started...");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {}
        System.out.println("Execution finished.");
    }
}
```
```java
public class SleepThrowsExample {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Execution started...");
        Thread.sleep(2000); 
        System.out.println("Resumed after 2 seconds.");
        System.out.println("Execution finished.");
    }
}
```

#
- `join()` → Makes the current thread wait until another thread finishes its execution.
- The Thread on which we do join, let them complete it first and then resume the other thread.
- eg: Main thread --> waiting
        
    t1 thread --> Runnable/Waiting

    Main Thread--> waiting --> runnable -->Terminated
-We can also pass time in join method like `t1.join(2000)` <-- This thread will execute for 2 sec, then main thread start.
#
- `yield()` → Gives a hint to the thread scheduler that the current thread is willing to temporarily give up CPU execution.
- not used in production because OS can reject it.
#
- `interrupt()` → It send signal to thread to stop doing what is its doing.
- Thread --> interrupt() (default true)

    interrupt()-> It is a flag. 

```java
public class SleepInterruptExample {
    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            try {
                System.out.println("Worker: Going to sleep for 10 seconds...");
                Thread.sleep(10000); 
                System.out.println("Worker: Finished sleep normally.");
            } catch (InterruptedException e) {
                System.out.println("Worker: I was interrupted while sleeping! Exiting...");
            }
        });
        worker.start();
        Thread.sleep(1000); 
        System.out.println("Main: Waking up the worker early...");
        worker.interrupt(); 
    }
}
```

- isInterrupted() --> return interrupt flag value (true or false)

- interrupted() --> return interrupt flag value(T/F) but also set it back to  false.
#
- `isAlive()` → Checks whether a thread has been started and has not yet finished execution. Returns `true` if the thread is alive, otherwise `false`.

```java
public class IsAliveExample {
    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            System.out.println("Worker: Starting my task...");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                System.out.println("Worker: Interrupted!");
            }
            System.out.println("Worker: Task finished.");
        });
        System.out.println("Before start() - Is worker alive? " + worker.isAlive()); // Output: false
        worker.start();

        System.out.println("After start() - Is worker alive? " + worker.isAlive()); // Output: true

        Thread.sleep(3000);
        System.out.println("After completion - Is worker alive? " + worker.isAlive()); // Output: false
    }
}
```

## Thread Priority

- **Thread Priority** is a value assigned to a thread that indicates its importance to the thread scheduler.
- Java provides priorities from **1 to 10**.
### Priority Values

- `1` → Minimum priority
- `5` → Normal priority (default)
- `10` → Maximum priority

Java provides constants:

```java
Thread.MIN_PRIORITY   // 1
Thread.NORM_PRIORITY  // 5
Thread.MAX_PRIORITY   // 10
```

#### Setting Thread Priority
- We can set the priority using the `setPriority()` method:

```java
Thread t = new Thread();

t.setPriority(8);
```

#### Getting Thread Priority

We can get the priority using the `getPriority()` method:

```java
System.out.println(t.getPriority());
```

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("Thread is running...");
    }
}
public class Main {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        t1.setPriority(8);
        System.out.println(t1.getPriority());
        t1.start();
    }
}
```

#### Important Points
- Default priority of a newly created thread is normally 5.
- A child thread generally inherits the priority of the thread that creates it.
- Priority is only a hint to the scheduler, not a guarantee.
- A higher-priority thread is not guaranteed to execute before a lower-priority thread.
- Valid priority values are 1 to 10.
- If a value outside this range is passed to `setPriority()`, Java throws `IllegalArgumentException`.

```java
t1.setPriority(11);  // IllegalArgumentException
```

## Daemon Thread
- Background running thread
- terminated when all user thread has completed its exextion
- mostly used in garbage collections

#### Example of How to make a Daemon thread

```java
public class DaemonThreadExample {
    public static void main(String[] args) {
        Thread backgroundCleaner = new Thread(() -> {
            while (true) {
                System.out.println("Daemon Thread: Cleaning up temporary files in the background...");
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    System.out.println("Daemon Thread: Interrupted.");
                }
            }
            //Despite of infinite loop, it will stop after execution of main thread;
        })
        backgroundCleaner.setDaemon(true); //setting deamon flag true;

        backgroundCleaner.start();

        System.out.println("Main Thread: Starting core application logic...");
        try {
            Thread.sleep(2000); 
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Main Thread: Finished execution. Application is exiting now!");
    }
}
```

## Problem of Multi-Threading

### Race Condition
- Race condition in java multithreading occurs when two or more threads access and modify the same shared data simultaneously and final result depend on order of execution.

#### Critical section -> part of program which is accessed by many threads


### Atomicity

To solve problem of Atomicity we have two approch

- Synchronizes block
- Atomic Integer/Double/Float

Atomic operation are like: `int x=2`, `Student s1 =new Student()`;

Non-Atomic Operations are like: `count++`

`Synchronized` Keyword solve it. --> It locks critical area and allows only one thread to execute.

### Visibility
One thread updates a variable but another does not see the updated value;

We use the keyword `volatile` to solve the issue of visibility.

#### `println` is already sunchronized

### Ordering
Sequence in which instruction are executed is not necessory.

Solved with  `synchronized` or `volatile`

## Thread Interference:
All the problem occur like Non-Atomic options, Shared Resources, Race conditon, ordering visibility are called Thread interference.

It leads to Data inconsistency.

Solved by: Sunchronization, volatile, Atomic Integer, Proper-Locking Machenisms

### Synchronized

eg:
```java
class Counter {
    private int count = 0;
    public synchronized void increment() {
        count++;
    }
    public int getCount() {
        return count;
    }
}
public class SynchronizationDemo {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();
        Thread thread1 = new Thread(()->{
            for (int i = 0; i < 10000; i++){
                counter.increment();
            }
        });
        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                counter.increment();
            }
        });
        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();
        System.out.println("Final Count: " + counter.getCount());
    }
}
```
- Firstly T1 acquire lock
- T2 tries to lock block state.
- T1 will perform task.
- As soon T1 exit T2 can enter.

Why we need Synchronization:
- To protect shared data
- To make any option atomic
- To ensure visibility
- To prevent reordering

Synchronized has two locks: Monitor lock and object lock

locks belongs to class and object.

#### `Static Synchronization` --> when we are not taking lock on object, we are taking lock on class

eg:

```java
class Counter {
    private static int count = 0;
    // Static synchronized method locks the Counter.class object
    public static synchronized void increment() {
        count++;
    }
    public static int getCount() {
        return count;
    }
}

public class StaticSynchronizationDemo {
    public static void main(String[] args) throws InterruptedException {
  
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                Counter.increment();
            }
        });
        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                Counter.increment();
            }
        });
        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println("Final Static Count: " + Counter.getCount());
    }
}
```

#### We can take one lock on class and other on object.
