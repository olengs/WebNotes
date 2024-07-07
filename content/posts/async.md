---
title: "std::future and asynchronous programming"
summary: "Notes for c++ async"
date: 2024-07-03
tags: ["c++", "async", "multithreading"]
author: ["JC"]
draft: true
weight: 0
ShowToc: true
---

### Summary
This post will talk about std::async, std::future, std::packaged_task, std::thread and asyncronous programming as a whole

In c++, There is currently 3 ways, as of writing, to create non-blocking functions.
1. std::promise (asynchronous)
2. std::async (asynchronous)
3. std::thread (multi-threaded)

So, when do we use these? It's dependent on your use-case.

Let's start with std::thread.

--- 

### std::thread[(link)](https://en.cppreference.com/w/cpp/thread/thread)

std::thread is the simplest of them all to use. But the conditions are also very limited.
You pass a function and its arguments into a thread constructor and it will start running.

```c++{linenos=true}
#include <thread>

void doSomething(int a, int b) {
    //do something
}

auto thread = std::thread(&doSomething, 2, 5);
```

The thread class also have the following features.
1. std::thread::join
2. std::thread::detach
3. std::thread::swap

The thread class will stop all task when the reference is destroyed. Hence you have to save the thread so that it doesn't get destroyed while running.
The thread.join() allows you to wait for the thread to finish, before continuing to run the next line of code. Which blocks the current thread until the process is completed.
The thread.detach() allows the thread to execute independently from the thread handle. Which would prevent the thread from being destroyed until it is finished.
The thread.swap() simply swaps the two threads passed.

```c++{lineos=true}
#include <thread>

void doSomething(int a, int b) {
    //do something
}

auto thread = std::thread(&doSomething, 1, 2);
auto thread2 = std::thread(&doSomething, 3, 4);
thread.join();
thread2.detach();
thread.swap(thread2);
```

You might have realised a flaw in the above std::thread usage. There is NO WAY to return a value from the function.
This is where std::future comes in.

---

### std::future[(link)](https://en.cppreference.com/w/cpp/thread/future)

std::future is the result, the value returned from an asynchronous process.
it is commonly used with
1. [std::packaged_task](../async#std::packaged_task)
2. [std::promise](../async#std::promise)
3. [std::async](../async#std::async)

Packaged task and promise are super similar with async being only slightly different.
Let's go through these features.

---

### std::packaged_task [(link)](https://en.cppreference.com/w/cpp/thread/packaged_task)

Firstly, packaged task solves the problem where threads are unable to 