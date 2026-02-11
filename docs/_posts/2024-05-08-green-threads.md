---
layout: post
title:  "Green threads"
date:   2024-05-08 12:31:19 +0300
categories: hpc
tags: multithread programming 
comments: false
---

{% include mathjax.html %}

In this post, I am going to give my brief understandings for the terms `green threads, goroutines, coroutines` that are easily confused  when they are seen in multithread programming context.

## Processes and Threads
From a process point of view, its memory segmented as text (for its instructions)-data(heap (for dynamic memory), uninitialized, initialized data part)-and stack. In each process, when there are function calls, they are executed by creating function frames that include parameters, local variables, and return values. Therefore, for each process, the top of the stack needs to be tracked to be able to determine which function frames and scopes the process is currently executing and which reachable variables are accesible from that scope.
As a result, for each process, there is at least a `stack pointer` that indicates the top of the stack, a `program counter` that indicates which instruction needs to be executed.

In a Unix environment a process or a thread  can be created by using the same system call: a new process is created as a child process of the current process by using `fork()` or `clone()` system calls. As the name indicates, when we create a new process, basically we are creating a clone of its whole memory (stack-heap-data-text).
On the other hand, we create threads for a process by limiting this operation only to  **stack**: That means we are kind of dividing the process's stack into multiple segments where we have separate `stack pointers` and `program counters` to run more than one function frame at the same time. Therefore, they are kind of lightweight processes because only their stack frames are different.

## Green Threads (fibers) and Goroutines
In the above multithread scenario, each thread we create in our program indicates an underlying kernel side thread (or a lightweight process). That means we have one-to-one mapping from each software thread to a kernel side thread.

`Green threads` are those software threads that do not necessarily have one-to-one mapping. That means we have a few kernel threads: Let's say $nk$.
And we have a few software (user side) threads: Let's say $nu$.
The basic property is that we generally have $nu \geq nk$. This means we have to have a thread scheduler that schedules these software threads to the kernel threads in user space. The scheduler is generally defined in the runtime library or virtual machine of the programming languages.
We call these software threads as `green threads` or `fibers`. 


`Goroutines` are the green threads that are implemented for `Go` programming language.

## Coroutines
Coroutines are somehow related to the definitions we have above. It is a programming design pattern where we indicate parts a program can be suspended and execution can be started concurrently (independently) with different data. This design pattern is used to do cooperative multitasking.
They do not provide multithreading themselves. If they are scheduled to run on separate stack frames, they are called `stackful coroutines`. Stackful coroutines can be suspended and executed from the same point. If they are run inside the host frame, then they are called `stackles coroutines`. 
Green threads (fibers) can be considered running `stackful coroutines` since they describe tasks with a stack frame. 

Below is an example of a couroutine with Python generators:

```python
1. def generator_function():
2.    for i in range(10):
3.       print(f"generator returning {i} with yield")
4.       given = (yield i)
5.       print("given char: ", given)
```
then we use this as

```
1. gen = generator_function()
2. value = gen.__next__() #first yield i
3. print("generated value:", value)
4. value = gen.send("a") #runs till next yield i
5. print("generated value:", value)
6. value = gen.send("b") #runs till next yield i
7. print("generated value:", value)
```
In this case, we are sending a value to the generator and it yields with value `i` in the next iteration.  The output would be

```text
generator returning 0 with yield
generated value: 0
given char:  a
generator returning 1 with yield
generated value: 1
given char:  b
generator returning 2 with yield
generated value: 2
```

Note that this is not a multithreaded program. However, we can convert this into multithreading  by using threadpool, that is `concurrent.futures.ThreadPoolExecutor()`, with the argument `send` function. In this way `send()` can be called by different threads at different times. However, calls to the generator needs synchronization: That is, the send calls from different threads cannot interleave. An implementation with a simple lock mechanism would be as follows(note that the lock makes only one `send()` at a time):

```python
1. class thread_sync:
2.     def __init__(self, generator):
3.         self.generator = generator
4.         self.lock = threading.Lock()
5. 
6.     def __iter__(self):
7.         return self
8. 
9.     def next(self, msg=None):
10.         r = None
11.         try:
12.             if msg == None:
13.                 with self.lock:
14.                     r = self.generator.__next__()
15.             else:
16.                 with self.lock:
17.                     r = self.generator.send(msg)
18.         except StopIteration:
19.             pass
20.         return r
```
We use this class as:

```python
1. def generator_function():
2.     for i in range(100):
3.         given = (yield i)
4.         time.sleep(0.1)
5.         print("threadid:{}, given char:{}, prev yield:{}"
6.                 format(threading.current_thread().ident, given, i))
```
Again we use this as:
```python
1. gen = generator_function()
2. tgen = thread_sync(gen)
3. tgen.next()
4. with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executer:
5.     yield_values = list(executer.map(tgen.next, string.ascii_letters))
6. 
7. for val in yield_values:
8.     if val != None:
9.         print("value:", val)

```
As a final remark note that coroutines are much more than the simple generator we have designed. And they can be very useful for asynchronous tasks. See [python doc](https://docs.python.org/3/library/asyncio-task.html) for explanation.

Discussion on the usefulness of fibers:  
[https://open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1364r0.pdf](https://open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1364r0.pdf)   
[https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0866r0.pdf](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p0866r0.pdf)   
[https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1520r0.pdf](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1520r0.pdf)    



