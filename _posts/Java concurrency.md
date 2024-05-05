---
title: Java concurrency (Study notes)
date: 2024-05-05 20:13:41
categories:
 - IT Technology
tags:
 - Java
 - Concurrency
---

# Java concurrency

The Fork/Join model in multi-threaded programming:

(1) Initial setup: The Main Thread
(2) Fork: Spawn new subtasks
(3) Parallel execution
(4) Join: consolidate results
(5) Repeat

Critical Section: shared resources or variables
Race Condition: multiple threads trying to do things.
Synchronization tools: coordinate threads with critical section. includes: Mutexes, Read/Write locks, Semaphores, Condition variables, and Barriers. 

<!-- more -->

# Synchronization
## Locks or Mutexes
It is mutual exclusion, ensuring that only one thread can access a shared resource.

```kotlin
fun mutex() {
    var counter = 0
    val lock = Any()
    counter = 0
    val executors = Executors.newFixedThreadPool(10)
    val futures = (1..20).map {
        executors.submit {
            synchronized(lock) {
                counter += 1
            }
        }
    }

    // join
    futures.map { it.get() }
    println(counter) // will always output 20
}
```

## Read/Write locks
Read/Write Locks, or Shared/Exclusive Locks is defined as below:

Read Lock: multiple threads can hold a read lock simultaneously, but if Write lock is occupied, then it needs to wait.

Write Lock: Write lock is exclusive. Only one thread can hold a write lock at any given time to ensure safe modification of the shared resource. When a thread holds a write lock, no other thread can acquire a read lock or a write lock, preventing concurrent access during write operation.

```kotlin
fun readWriteLock() {
    var counter = 0
    val target = 100
    val lock = ReentrantReadWriteLock()

    val writer = {
        while (true) {
            lock.writeLock().lock()
            try {
                if (counter < target) {
                    counter++
                }
                if (counter == target) break
            } catch (_: InterruptedException1) {
            } finally {
                lock.writeLock().unlock()
            }
            Thread.sleep(Random.nextLong(500))
        }
    }

    val reader = {
        while (true) {
            lock.readLock().lock()
            try {
                if (counter == target) break
                println("reader value: $counter")
            } catch (_: InterruptedException1) {
            } finally {
                lock.readLock().unlock()
            }
            Thread.sleep(Random.nextLong(100))
        }
    }

    val executors = Executors.newFixedThreadPool(10)
    val futures = (1..50).map { index ->
        executors.submit {
            if (index % 2 == 0) {
                reader()
            } else {
                writer()
            }
        }
    }

    // join
    futures.map { it.get() }
    println(counter) // will always output 100 no matter how many readers/writers
}
```

## Semaphores
Semaphores maintains an integer value to manage permits count. They permit multiple threads to access a shared resource, but only up to a predetermined number of permits.

Binary Semaphore: works the same as Mutex.
Counting Semaphore: limits the number of simultaneous accesses to a resource.

```kotlin
fun semaphore() {
    val counter = AtomicInteger(0)
    val target = 100
    val semaphore = Semaphore(5)

    val executors = Executors.newFixedThreadPool(10)
    val futures = (1..10).map { index ->
        Thread.currentThread().name = "Thread-$index"
        executors.submit {
            try {
                semaphore.acquire()
                while (true) {
                    if (counter.get() >= target) {
                        break
                    }
                    counter.incrementAndGet()
                    println("${Thread.currentThread().name} is working")
                    Thread.sleep(Random.nextLong(500))
                }
            } catch (_: InterruptedException) {
            } finally {
                semaphore.release()
            }
        }
    }

    // join
    futures.map { it.get() }
    println(counter)
    // will always output 100
    // and no matter how many threads we spawn, only 5 (permits number) are working
}
```

## Condition Variables
To allow threads to suspend execution until specified conditions are satisfied.

Wait (await): a thread waits for a condition to be true. While waiting, the thread is blocked and does not consume CPU resources.
Notify (signal / signalAll): if condition is true, will trigger to resume the execution of one or more waiting threads.

```kotlin
fun lockCondition() {
    val lock = ReentrantLock()
    val condition = lock.newCondition()
    var sharedResource = 0
    var ready = false

    val producer = {
        Thread.sleep(1000)
        lock.withLock {
            sharedResource = Random.nextInt(100);
            ready = true
            println("Producer has produced: $sharedResource")
            condition.signal()
        }
    }

    val consumer = {
        while (true) {
            lock.withLock {
                condition.await()
                if (ready) {
                    println("Consumer has consumed the number: $sharedResource")
                    return@withLock
                }

                println("I am busy waiting")
                // without condition notify and await, there are lots of busy waiting
                // with condition notify and await, no more busy waiting
            }
            if (ready) break
            Thread.sleep(10)
        }
    }

    val producerThread = Thread(producer)
    val consumerThread = Thread(consumer)
    producerThread.start()
    consumerThread.start()
    producerThread.join()
    consumerThread.join()
}
```

## Barrier
Barrier makes multiple threads wait until all threads reach a certain point, after which they can all resume to be  proceed. It ensures that no thread starts executing until all the threads have reached the barrier.


```kotlin
fun barrier() {
    val count = 3
    val barrier = CyclicBarrier(count) {
        println("$count threads have reached the barrier. Continue execution.")
    }
    val executors = Executors.newFixedThreadPool(count)
    val futures = (1..count).map {
        executors.submit {
            println("Thread $it is waiting at the barrier")
            barrier.await()
            println("Executing thread $it")
        }
    }
    futures.map { it.get() }
}

//    Thread 1 is waiting at the barrier
//    Thread 2 is waiting at the barrier
//    Thread 3 is waiting at the barrier
//    3 threads have reached the barrier. Continue execution.
//    Executing thread 2
//    Executing thread 1
//    Executing thread 3
```
