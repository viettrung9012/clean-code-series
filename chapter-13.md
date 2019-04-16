# Chapter 13 - Concurrency

Writing clean concurrent programs is hard—very hard. It is much easier to write code that executes in a single thread. It is also easy to write multithreaded code that looks fine on the surface but is broken at a deeper level.

Concurrency is a decoupling strategy: it helps us decouple **what** gets done from **when** it gets done.

## Myths and Misconceptions

- Concurrency always improves performance.
  
  Concurrency can sometimes improve performance, but only when there is a lot of wait time that can be shared between multiple threads or multiple processors. Neither situation is trivial.

- Design does not change when writing concurrent programs.
  
  In fact, the design of a concurrent algorithm can be remarkably different from the design of a single-threaded system. The decoupling of what from when usually has a huge effect on the structure of the system.

- Understanding concurrency issues is not important when working with a container such as a Web or EJB container.

  In fact, you’d better know just what your container is doing and how to guard against the issues of concurrent update and deadlock described later in this chapter.
  
  Here are a few more balanced sound bites regarding writing concurrent software:

  - Concurrency incurs some overhead, both in performance as well as writing additional code.

  - Correct concurrency is complex, even for simple problems.

  - Concurrency bugs aren’t usually repeatable, so they are often ignored as one-offs instead of the true defects they are.

  - Concurrency often requires a fundamental change in design strategy.

## Concurrency Defense Principles

1. Single Responsibility Principle

    Keep your concurrency-related code separate from other code.

2. Corollary: Limit the Scope of Data

   Take data encapsulation to heart; severely limit the access of any data that may be shared.

3. Corollary: Use Copies of Data

    If you can introduce immutable objects instead of synchronizing, do so. It’s worth the garbage collection overhead.

4. Corollary: Threads Should Be as Independent as Possible
  
   Attempt to partition data into independent subsets that can be operated on by independent threads, possibly in different processors.

## Know Your Library

Use java.util.concurrent.* collections instead of regular ones. In particular, you should use ConcurrentHashMap instead of HashMap always, as it is more performant in general.

Become familiar with java.util.concurrent.*, java.util.concurrent.atomic.*, java.util.concurrent.locks.*, e.g.
  
    - ReentrantLock - A lock that can be acquired in one method and released in another.
    - Semaphore - An implementation of the classic semaphore, a lock with a count.
    - CountDownLatch - Releases all threads after receiving a number of events.

## Know Your Execution Models

1. **Producer-Consumer**

   One or more producer threads create some work and place it in a buffer or queue. One or more consumer threads acquire that work from the queue and complete it. The queue between the producers and consumers is a bound resource. They have to signal each other and wait for the signal from another before doing anything.

2. **Readers-Writers**
  
   where readers read more often than writers write. Writers could wait for all readers to finish before they write, but they will starve in case of continuous readers. A balance needs to be found.

3. **Dining Philosophers**

**_Most concurrency problems are combination of the three concurrent execution models_**

## Beware Dependencies Between Synchronized Methods

Avoid using more than one method on a shared object.

## Keep Synchronized Sections Small

Keep your synchronized sections as small as possible.

## Writing Correct Shut-Down Code Is Hard

Writing correct shut-down is harder than you think. Plan enough time for that.

## Test Threaded Code

Write tests that have the potential to expose problems and then run them frequently, with different programmatic configurations and system configurations and load. If tests ever fail, track down the failure. Don’t ignore a failure just because the tests pass on a subsequent run.

- Make your threaded code pluggable.

- Make your threaded code tunable.

- Run with more threads than processors.

- Run on different platforms.

- Instrument your code to try and force failures.

## Conclusion

Concurrent code is difficult to get right. Code that is simple to follow can become nightmarish when multiple threads and shared data get into the mix. If you are faced with writing concurrent code, you need to write clean code with rigor or else face subtle and infrequent failures.

**First and foremost, follow the Single Responsibility Principle.** Break your system into POJOs that separate thread-aware code from thread-ignorant code. Make sure when you are testing your thread-aware code, you are only testing it and nothing else. This suggests that your thread-aware code should be small and focused.

Know the possible sources of concurrency issues.

Learn your library and know the fundamental algorithms.

Learn how to find regions of code that must be locked and lock them.

Issues will crop up. The ones that do not crop up early are often written off as a one-time occurrence. These so-called one-offs typically only happen under load or at seemingly random times. Therefore, you need to be able to run your thread-related code in many configurations on many platforms repeatedly and continuously. Testability, which comes naturally from following the Three Laws of TDD, implies some level of plug-ability, which offers the support necessary to run code in a wider range of configurations.

You will greatly improve your chances of finding erroneous code if you take the time to instrument your code. You can either do so by hand or using some kind of automated technology. Invest in this early. You want to be running your thread-based code as long as possible before you put it into production.

**If you take a clean approach, your chances of getting it right increase drastically.**
