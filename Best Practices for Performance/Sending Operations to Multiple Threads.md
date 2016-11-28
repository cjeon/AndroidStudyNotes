# Sending Operations to Multiple Threads
[link](https://developer.android.com/training/multiple-threads/index.html)

<!-- TOC -->

- [Sending Operations to Multiple Threads](#sending-operations-to-multiple-threads)
- [Specifying the Code to Run on a Thread](#specifying-the-code-to-run-on-a-thread)
    - [Define a Class that Implements Runnable](#define-a-class-that-implements-runnable)
    - [Implement the run() Method](#implement-the-run-method)
- [Creating a Manager for Multiple Threads](#creating-a-manager-for-multiple-threads)
    - [Define the Thread Pool Class](#define-the-thread-pool-class)
- [Running Code on a Thread Pool Thread](#running-code-on-a-thread-pool-thread)
- [Communicating with the UI Thread](#communicating-with-the-ui-thread)

<!-- /TOC -->

# Specifying the Code to Run on a Thread

[`Runnables`](https://developer.android.com/reference/java/lang/Runnable.html)
> The Runnable interface should be implemented by any class whose instances are intended to be executed by a thread. The class must define a method of no arguments called run.

> One or more Runnable objects that perform a particular operation are sometimes called a task.

## Define a Class that Implements Runnable

``` java
public class PhotoDecodeRunnable implements Runnable {
    ...
    @Override
    public void run() {
        /*
         * Code you want to run on the thread goes here
         */
        ...
    }
    ...
}
```

## Implement the run() Method

> At the beginning of the run() method, set the thread to use background priority by calling `Process.setThreadPriority()` with `THREAD_PRIORITY_BACKGROUND`. This approach reduces resource competition between the Runnable object's thread and the UI thread.

`android.os.Process.setThreadPriority(android.os.Process.THREAD_PRIORITY_BACKGROUND);`

# Creating a Manager for Multiple Threads

[`ThreadPoolExecutor`](https://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html)

> An ExecutorService that executes each submitted task using one of possibly several pooled threads, normally configured using Executors factory methods.

## Define the Thread Pool Class



# Running Code on a Thread Pool Thread

# Communicating with the UI Thread
