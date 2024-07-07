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

Starting with packaged task, package task solves the problem where threads are unable to return values after completion.

```c++ {linenos=true}
void task_thread()
{
    //create package task by passing function and arguments
    std::packaged_task<int(int, int)> task(f);
    //keep the result using std::future
    std::future<int> result = task.get_future();
    //run the function asynchronously via threads.
    std::thread task_td(std::move(task), 2, 10);
    //wait for function to finish
    task_td.join();
    //get result from task
    std::cout << "task_thread:\t" << result.get() << '\n';
}
```

---


### std::promise [(link)](https://en.cppreference.com/w/cpp/thread/promise)

Next, we have std::promise, std::promise is more like a communication channel between different threads.
Note that you will have to pass in the std::promise into the running function as you have to set the value into the promise.
Also note that std::promise is not copy-constructible and you have to use [std::move](https://en.cppreference.com/w/cpp/utility/move) to pass the promise into the function.

```c++{linenos=true}

void accumulate(std::vector<int>::iterator first,
                std::vector<int>::iterator last,
                std::promise<int> accumulate_promise)
{
    int sum = std::accumulate(first, last, 0);
    accumulate_promise.set_value(sum); // Notify future
}

int main()
{
    // Demonstrate using promise<int> to transmit a result between threads.
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6};
    std::promise<int> accumulate_promise;
    std::future<int> accumulate_future = accumulate_promise.get_future();
    std::thread work_thread(accumulate, numbers.begin(), numbers.end(),
                            std::move(accumulate_promise));
 
    // future::get() will wait until the future has a valid result and retrieves it.
    // Calling wait() before get() is not needed
    // accumulate_future.wait(); // wait for result
    std::cout << "result=" << accumulate_future.get() << '\n';
    work_thread.join(); // wait for thread completion
}
```

A few more pointers to consider when using std::promise
1. Like the previous few examples have shown, std::future .get() function will block until you get a promise where if you dont, if will forever be waiting on the promise (ill-formed program)
2. It is possible to detach the thread and just use future.get() to wait for the result

---

### std::async [(link)](https://en.cppreference.com/w/cpp/thread/async)
Lastly, we have std::async which can wraps up both packaged_task and promise to make it easier to use.

There are 2 launch policies for std::async, std::launch::async and std::launch::deferred.
For default calls of std::async, policy is treated as std::launch::async | std::launch::deferred.
1. std::launch::async - runs the function on a thread immediately and holds the result until a future.get() is called.
2. std::launch::deferred - runs the function WHEN the future.get() or future.wait() is called and on the thread that the future.get() or future.wait() is called.
3. std::launch::async | std::launch::deferred - will run async if system allows and if system ran out of resources for async to create threads, will run deferred instead.

```c++{linenos=true}


```


Pointers to note:
1. the returned std::future from the std::async has to be saved as the invoke will get destroyed when the std::future is destroyed.
2. 
