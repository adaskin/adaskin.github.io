---
layout: post
title:  "Python multithread and multiprocess programming: Mini quantum simulator"
date:   2024-06-05 12:31:19 +0300
categories: quantum-computing
tags: quantum simulator multithread multiprocess programming python
comments: true
---

Recently, I've put a public mini quantum simulator repo on [github](https://github.com/adaskin/a-simple-quantum-simulator). While it is desirable to use quantum simulator packages such as [PennyLane](https://www.pennylane.com/) or [Qiskit](https://www.ibm.com/quantum/qiskit), sometimes as a researcher, it is best to have our own simulator code so that we can try different things on it.
While combining my old python codes, I've decided to put also a multiprocess version of the simulator. The followings are my observations of multihreading and multiprocessing in Python for my quantum simulator.

# Multiprocessing in Python
If your familiar with low level multiprocessing, then many familiar concepts are provided as higher level library in Python [multiprocessing](https://docs.python.org/3/library/multiprocessing.html). For inter process communication (data transfer), it provides shared_memory and pipes. For shared memory, it is enough to create a shared memory (either an array or a value) with a lock for synchronization:
```python
shm = multiprocessing.Array(ctypes.c_double, N, lock=True)
``` 
Here, when ``lock`` is true, it is created as a shared memory and it returns a **synchronized wrapper** so all reads and writes on the array are synced. You can also use ``shm.acquire()...shm.release()`` explicitly to acquire and release the lock.

In some cases, it makes sense to use our own lock: For instance in our mini quantum simulator, while computing probabilities, at the beginning we just read the vector elements. Therefore, there is no need for synchronization at the beginning. 
In these case, you can create a shared memory without a lock and use your own Lock() object for the synchronization.
```python
shm = multiprocessing.Array(ctypes.c_double, N, lock=False)
...
lock = multiprocessing.Lock()
...
#in functions run by child processes
lock.acquire()
...#critical section
lock.release()
```
For complete example see the file ``qusimmultiprocwithshm.py`` in the [repo](https://github.com/adaskin/a-simple-quantum-simulator). Below is the implementation for computing probabilities of a qubit:
```python
def worker_prob_of_a_qubit(psi, start, end, qshift, fshared):
    flocal = np.zeros(2)
    for j in range(start, end):
        qbitval = (j >> qshift) & 1
        flocal[qbitval] += np.real(np.abs(psi[j])**2)

    fshared.acquire()
    try:
        for i in range(len(fshared)):
            fshared[i] += flocal[i]
    except:
        print("exception occured while getting lock")
    finally:
        fshared.release()   

    return flocal

def prob_of_a_qubit(psi, qubit):

    N = len(psi)
    n = int(np.log2(N))
    fshared = mp.Array(ctypes.c_double, 2, lock=True) 
    fzero = np.zeros(2,dtype=ctypes.c_double)
    fshared[:] = fzero[:]
    qshift = n - qubit -1
    
    processes = []
    
    #for each thread assign part of the mem
    for ti in range(nthreads):
       
        start = thread_data_range*ti
        end = thread_data_range*(ti+1)
        end = N if end > N else end
        p = mp.Process(target=worker_prob_of_a_qubit,
                    args=(psi, start, end, qshift, fshared))
        processes.append(p)
        p.start()
    
    for p in processes:
        p.join()
    return fshared
```
## Notes on pipes
Note that we can also use pipes to get the return value of the processes. **However, when the dimension of the return value is high and you are able to use shared memory, using pipes would not make much sense since it needs to send/recv huge chunk of data between processes**. As an example for pipes, we can compute partial probability for a qubit by using following function:
```python
def thread_partial_prob_with_pipe(psi, start, end, qshift, pipe):
    flocal = np.zeros(2)    
    for j in range(start, end):
        qbitval = (j >> qshift) & 1
        flocal[qbitval] += np.real(np.abs(psi[j][0])**2)

    pipe.send(flocal)    
    return flocal
```
In the calling function, we use other end of the pipe to read the data send by any process.
```python
def prob_of_a_qubit_multiprocess_with_pipe(psi, qubit):
    N = len(psi)
    n = int(np.log2(N))
    fshared = np.zeros(2)
    qshift = n - qubit -1
    processes = []
    pipes = []
    #for each thread assign part of the mem
    for ti in range(nthreads):
        recv_end, send_end = mp.Pipe(False)
        
        start = thread_data_range*ti
        end = thread_data_range*(ti+1)
        end = N if end > N else end
        p = multiprocessing
            .Process(target=thread_partial_prob_with_pipe,
                    args=(psi, start, end, qshift, send_end))
        processes.append(p)
        pipes.append(recv_end)
        p.start()
    
    for p in processes:
        p.join()
    for pipe in pipes:
        flocal = pipe.recv()
        for i in range(len(fshared)):
           fshared[i] += flocal[i] 
```

# Notes on multithreading
For those who used to using multithread programming in C/C++ or Java-C#, Python multithreading may be a little surprising at the beginning: Because of global interpreter lock on Python objects(see [GIL](https://wiki.python.org/moin/GlobalInterpreterLock)), if your multithread program( whether it uses threadpools or regular threading) is doing more computations than waiting data on I/O, then it does not provide any speed-up (yes I've also tried this on my mini quantum simulator in different versions :neutral_face:.  So the regular multiprocessing lib described above should be used for  multithreading of CPU bound tasks.).
In comparison to [threadpools](https://docs.python.org/3/library/concurrent.futures.html) or implementing with lock on memory,  the following implementation runs faster since it does not have lock between threads ( **even though it is still slower than serial implementation because of GIL**):
```python
class ProbThread(Thread):
    def __init__(self, psi, start, end, qshift):
        self.flocal = np.zeros(2)
        self.psi = psi
        self.start = start
        self.end = end
        self.qshift = qshift
    
    def run(self):
        for j in range(self.start, self.end):
            qbitval = (j >> self.qshift) & 1
            self.flocal[qbitval] += np.real(np.abs(self.psi[j])**2)
    def join(self):
        return self.flocal

def prob_of_a_qubit_with_ProbThread_run(psi, qubit):
    N = len(psi)
    n = int(np.log2(N))
    fshared = np.zeros(2)
    qshift = n - qubit -1
    threads = []   
    #for each thread assign part of the mem
    for ti in range(nthreads):
        start = thread_data_range*ti
        end = thread_data_range*(ti+1)
        end = N if end > N else end
        thread = ProbThread(psi, start, end, qshift)
        threads.append(thread)
        thread.run()
    #global sum of local sums
    for ti in range(nthreads):
        fi = threads[ti].join()
        for j in range(len(fshared)):
            fshared[j] += fi[j]

    return fshared

```

We can also implement with lock as we have done in multiprocessing:
```python
def thread_partial_prob_with_lock(psi, start, end, qshift,fshared,lock):
    flocal = np.zeros(2)
    for j in range(start, end):
        qbitval = (j >> qshift) & 1
        flocal[qbitval] += np.real(np.abs(psi[j][0])**2)
    
    with lock:
        for i in range(len(fshared)):
            fshared[i] += flocal[i]
    return flocal
def prob_of_a_qubit_with_lock(psi, qubit):
    N = len(psi)
    n = int(np.log2(N))
    fshared = np.zeros(2)
    qshift = n - qubit -1
    threads = []
    lock = threading.Lock()
    
    #for each thread assign part of the mem
    for ti in range(nthreads):
        start = thread_data_range*ti
        end = thread_data_range*(ti+1)
        
        end = N if end > N else end

        thread = Thread(target=thread_partial_prob_withlock, 
                        args=[psi, start, end, qshift, fshared, lock])
                
        threads.append(thread)
        thread.start()
        

    
    for ti in range(nthreads):
        threads[ti].join()

    return fshared
```
Or we can use thread pools. Note that this is the slowest implementation: 
```python
def thread_partial_prob(psi, start, end, qshift):
    flocal = np.zeros(2)
    for j in range(start, end):
        qbitval = (j >> qshift) & 1
        flocal[qbitval] += np.real(np.abs(psi[j])**2)
    return flocal

def prob_of_a_qubit_with_pool(psi, qubit)
    N = len(psi)
    n = int(np.log2(N))
    fshared = np.zeros(2)
    qshift = n - qubit -1
    futures = []
    with concurrent.futures.ThreadPoolExecutor(max_workers=MAX_THREADS) as executor:
        #for each thread assign part of the mem
        for ti in range(nthreads):
            start = thread_data_range*ti
            end = thread_data_range*(ti+1)
            end = N if end > N else end

            futures.append(executor.submit(thread_partial_prob, 
                                    psi, start, end, qshift))
    
    for f in futures:
        flocal = f.result()
        for i in range(len(fshared)):
            fshared[i] += flocal[i] 
    return fshared
```
{% include disqus.html %}