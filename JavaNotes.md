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
- `run()` → Contains the code/task that the thread executes. Calling `run()` directly does **not** create a new thread.
- `sleep()` → Pauses the currently executing thread for a specified amount of time.
- `join()` → Makes the current thread wait until another thread finishes its execution.
- `yield()` → Gives a hint to the thread scheduler that the current thread is willing to temporarily give up CPU execution.
- `interrupt()` → Interrupts a thread by setting its interrupt status. It can be used to request that a thread stop waiting, sleeping, or otherwise respond to interruption.
- `isAlive()` → Checks whether a thread has been started and has not yet finished execution. Returns `true` if the thread is alive, otherwise `false`.

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

