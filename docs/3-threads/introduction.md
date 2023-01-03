---
layout: post
title: '3.1 Introduction'
parent: '3. Threads'
---

## What are threads?

A thread is an isolated instance that is responsible for the execution of some task. While a microcontroller usually only has 1 CPU, the RTOS is able to have multiple tasks execute (seemingly) simultanously by exchanging the thread that gets run on the CPU as dictated by the scheduler. 

![rtos-basic-execution](../../images/3-threads/rtos_basic_execution.gif)

Each thread has the following key properties:
- **Stack area**: a region of memory used for the thread's stack. The size can be adjusted as required by the thread's processing.
    ```c
    /* size of stack area used by each thread */
    #define STACKSIZE 1024
    ```

- **Thread control block**: for internal bookkeeping of the thread's metadata. An instance of the type `k_thread`.
    ```c
    K_THREAD_STACK_DEFINE(threadA_stack_area, STACKSIZE);
    static struct k_thread threadA_data;
    ```

- **Entry point function**: invoked when the thread is started. Up to 3 argument values can be passed to this function.
    ```c
    void threadA(void *dummy1, void *dummy2, void *dummy3)
    {
	    ARG_UNUSED(dummy1);
	    ARG_UNUSED(dummy2);
	    ARG_UNUSED(dummy3);

        // Execute some logic here
    }
    ```

    {: .note }
    > `_ARG_UNUSED`: indicate to the compiler that an argument is not used in our thread function. 

- **Scheduling priority**: determines which thread gets priority.
- **Scheduling policy**: instructs the kernel's scheduler how to allocate CPU time to the thread. (This will be covered more in-depth in [5. Scheduling](docs/5-scheduling/introduction.md))
- **Execution mode**: can be supervisor or user mode. 
  - *Supervisor mode*: by default, allows access to privileged CPU instructions, the entire memory address space and peripherals.
  - *User mode*: thread has reduced set of privileges, this depends on the `CONFIG_USERSPACE` option.

The specifics of how to define a thread will be discussed in the next section

## When to use threads?

Use threads to handle logically distinct processing operations that can execute in parallel.

## How does Zephyr choose which thread to run?

"Thread is ready" = eligible to be selected as the next running thread.

Following factors can make a thread unready:
- Thread has not been started
- Waiting for a kernel object to complete an operation (for example, the thread is waiting for a semaphore that is not available)
- Waiting for a timeout to occur
- Thread has been suspended
- Thread has been terminated or aborted

The following diagram shows all the possible states a thread can find itself:

![thread_states](/images/threads/thread-states.png)