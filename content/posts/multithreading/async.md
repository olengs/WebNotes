---
title: "Asynchronous programming using std::future"
summary: "Notes for c++ async"
date: 2024-07-03
tags: ["C++", "Async", "Multithreading"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---

### Summary
This post will talk about std::async, std::future, std::packaged_task, std::thread and asyncronous programming as a whole

In c++, There is currently 3 ways, as of writing, to create non-blocking functions.
1. [std::promise](#stdpromise) (asynchronous)
2. [std::async](#stdasync) (asynchronous)
3. [std::thread](#stdthread) (multi-threaded)

So, when do we use these? It's dependent on your use-case.

Let's start with std::thread.

--- 

### std::thread 

[std::thread](https://en.cppreference.com/w/cpp/thread/thread) is the simplest of them all to use. But the conditions are also very limited.
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

### std::future

[std::future](https://en.cppreference.com/w/cpp/thread/future) is the result, the value returned from an asynchronous process.

it is commonly used with
1. [std::packaged_task](#stdpackaged_task)
2. [std::promise](#stdpromise)
3. [std::async](#stdasync)

Note that the std::future has to be saved as the invoke will get destroyed when the std::future is destroyed. This can be fixed with std::shared_future.

How it is used:

```c++{linenos=true}

//create the future
std::future<T> future = std::async(&func, args...);
//wait for the results to finish
future.wait()
//get the results (will also wait for results to finish, you do not have to call wait() before this)
std::cout << "result=" << future.get() << '\n';

//wait for a certain duration
std::future<T> future_waitfor = std::async(&func, args...);
if(std::future_status::ready == future_waitfor.wait_for(std::chrono::microseconds(0)))
    std::cout << "future process is completed: " << future_waituntil.get() << "\n";
else
    std::cout << "future process did not complete!\n";

//wait until a certain time
std::future<T> future_waituntil = std::async(&func, args...);
std::chrono::system_clock::time_point two_seconds_passed
        = std::chrono::system_clock::now() + std::chrono::seconds(2);
if(std::future_status::ready == future_waituntil.wait_until(two_seconds_passed))
    std::cout << "future process has completed: " << future_waituntil.get() << "\n";
else
    std::cout << "future process did not complete!\n";

//checks if future is still valid
std::future<T> future_valid = std::async(&func, args...);
future_valid.valid(); //returns true
future_valid.get();
future_valid.valid(); //returns false

//shares the future as a shared future
std::future future_share = std::async(&func, args...)
std::shared_future shared_future;
shared_future = future_share.share();
future_share.valid(); //returns false

```

future.wait_for() and future.wait_until() is a great way to check whether the process has ended.

---

### std::shared_future

[std::shared_future](https://en.cppreference.com/w/cpp/thread/shared_future) works the same way as a future.
The differences are:
1. a shared future will only be destroyed when all instances of the shared state has been destroyed
2. shared_future.get() will not invalid the future
3. A shared_future may be used to signal multiple threads simultaneously, similar to [std::condition_variable::notify_all()](https://en.cppreference.com/w/cpp/thread/condition_variable/notify_all).

---

### std::packaged_task 

Starting with [std::packaged_task](https://en.cppreference.com/w/cpp/thread/packaged_task), package task solves the problem where threads are unable to return values after completion.

```c++ {linenos=true}
#include <iostream>
#include <thread>
#include <future>

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


### std::promise 

Next, we have [std::promise](https://en.cppreference.com/w/cpp/thread/promise), std::promise is more like a communication channel between different threads.

Pointers to note:
1. You will have to pass in the std::promise into the running function as you have to set the value into the promise.
2. std::promise is not copy-constructible and you have to use [std::move](https://en.cppreference.com/w/cpp/utility/move) to pass the promise into the function.
3. If you look at the example below, std::future .get() function will block until you get a promise where if you dont, if will forever be waiting on the promise (ill-formed program)
4. It is possible to detach the thread and just use future.get() to wait for the result

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

---

### std::async 
Lastly, we have [std::async](https://en.cppreference.com/w/cpp/thread/async) which can wraps up both packaged_task and promise to make it easier to use.

There are 2 launch policies for std::async, std::launch::async and std::launch::deferred.
For default calls of std::async, policy is treated as std::launch::async | std::launch::deferred.
1. std::launch::async - runs the function on a thread immediately and holds the result until a future.get() is called.
2. std::launch::deferred - runs the function WHEN the future.get() or future.wait() is called and on the thread that the future.get() or future.wait() is called.
3. std::launch::async | std::launch::deferred - will run async if system allows and if system ran out of resources for async to create threads, will run deferred instead.

Pointers to note:
1. The returned std::future from the std::async has to be saved as the invoke will get destroyed when the std::future is destroyed. This can be fixed with std::shared_future.
2. the async function has a [[no_discard]] with means the return of the async function cannot be discarded. Hence, save the future somewhere. This is implemented as if the future is discarded, the async process will be destroyed right after it has been created.

```c++{linenos=true}
#include <future>
#include <thread>
#include <iostream>

int sum(int a, int b) {
    //do something
    return a+b;
}
//run async with default policy std::launch::async | std::launch::deferred
auto future = std::async(&sum, 5, 2);
//run async with async policy
//auto future = std::async(std::launch::async, &sum, 5, 2);

//run async with deferred policy
//auto future = std::async(std::launch::async, &sum, 5, 2);

//wait and get result of async process
std::cout << "result=" << future.get() << '\n';

```

---

