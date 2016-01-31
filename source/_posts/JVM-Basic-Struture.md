---
title: JVM Basic Structure
date: 2016-01-19 21:44:04
tags: [java, jvm]
categories:
---

It's been a long time before I want to summarise the structural basics within the JVM.
The graph below shows the relationship among those components, which can be cataloged into two sections, ones are private to individual thread,
and ones are independent to threads.

![](images/JVM_Internal_Architecture_small.png)

<!-- more -->
# Per Thread
A thread is a thread of execution in a program. The JVM allows an application to have multiple threads of execution running concurrently.

## System Threads
JVM System threads run along with the main thread in the background, created by invoking `public static void main(String[])`. The main background system threads in the Hotspot JVM are:

* **VM Thread**: waits for operations that require the JVM to reach a safe point such as: stop-world-gc, thread stack dumps.
* **Periodic Task Thread**: is responsible for timer related events.
* **GC Thread**: takes care of different garbage collection activities.
* **Compiler Thread**: compiles byte code to native code at runtime.
* **Signal Dispatcher**: receives signals sent to the JVM process and handle them inside the JVM by calling the appropriate JVM methods.

## Thread of Execution
Each thread of execution has the following components.

* **Program Counter(PC)**: keeps track of where JVM is executing instruction. PC will in fact be pointing at a memory address in _Method Area_.
* **Stack**: each thread has its own stack that holds frames for each method executing on the thread.
* **Native Stack**: not all JVM supports native stack.
* **Stack Restriction**: if a thread requires a larger stack than allowed, `StackOverflowError` will be thrown. If a thread requires a new frame and no memory to allocate, `OutOfMemoryError` will be thrown. Meanwhile, the size of the stack can be configurable, either dynamic or static.

## Frame
Frame is created when method is invoked, and be pushed to the top of stack. It will be popped when the methods returns normally or there is an uncaught exception. In each frame, it contains:

* **Local Variable Array**: contains all the variables used during the execution of the method. Reference to `this`, input parameters, and other local variables are included.
* **Operand Stack**: is used during the execution of byte code instructions like the general-purpose registers used in CPU.
* **Dynamic Linking**: Each frame contains a reference to the runtime constant pool, which references to the class of this method being executed for that frame.

# Common Components
To facilitate the memory management, shared area can be heap or non-heap, which are divided into three sections: **Young Generation**(eden and survivor), **Old Generation**(tenured generation), and **Permanent Generation**(no longer in Java 8).

## Heap

Heap is used to allocate class instances and arrays at runtime. Unlike the primitive variables and references in the local variable array, objects are always stored on the heap so they are not removed when a method ends. Instead objects can only be removed by the garbage collector.

## Non-Heap

Objects that are logically considered as part of the JVM mechanics are not created on the heap. Non-Heap memory includes:
* Permanent Generation: the _method area_ and _interned strings_
* Code Cache: used for compilation and storage of methods that have been comiled to native code by the JIT compiler.

## Method Area
All threads share the same method area, so access to the method area data and the process of dynamic linking must be thread safe. Method area stores per-class information such as:
* ClassLoader Reference
* Run time constant pool
* Field data
* Method data
* Method code

## Memory Management
1. New objects created into the young generation
2. Minor garbage collection in young generation. Alive objects will be moved from eden space to the survivor spaces (more detailed space can be specified).
3. Major garbage collection causes the application threads to pause. Alive objects will be moved across generations, young to old.
4. Permanent generation is collected every time the old generation is collected. They are both collected when either is full.
5. PermGen is replaced with Metaspace which is very similar in Java 8. Main difference is that Metaspace re-sizes dynamically i.e., It can expand at runtime. Java Metaspace space: unbounded (default)

## Just in Time (JIT) Compilation

Just-in-time (JIT) compiler is a program that turns Java bytecode (a program that contains instructions that must be interpreted) into instructions that can be sent directly to the processor.