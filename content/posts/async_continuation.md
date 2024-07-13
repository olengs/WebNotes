---
title: "Async continuation for c++"
summary: "My take on continuation for c++"
date: 2024-07-13
tags: ["c++", "async", "multithreading"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---



### Summary

In this post, I will be talking about the process of me making a continuation function for std::async to chain async calls and things that I have learnt while doing this. Why did I make this? I wanted to chain async functions using std::async like c# where u can do task.then().then(); I will only be share snippets, contact me for full project/details.


### 1. std::async (you can refer to [here](../async) as well)

Firstly, I require the ability to run a task asynchronously using async. I created a task class that holds a std::shared_future (result).
I also added a typedef to allow access to the type of the task.

``` {linenos = true}
template <typename T>
class Task {
public:
    typedef T type;
private:
	//shared future will retain results until all associations are destroyed
	std::shared_future<T> results;
}
``` 

### 2. Adding an init call to run function, using perfect forwarding of arguments.
``` {linenos = true}
	template<typename Func, typename ...Args>
	inline void Init(Func&& func, Args&& ...args) {
		results = std::async(std::launch::async, func, std::forward<Args>(args)...);

		isInit = true;
	}
``` 

### 3. Making the continuation function

My idea of creating this continuation function is to move the blocking call onto another thread. Hence, I called a std::thread to move my function onto a new thread and used async.get() to block the call until it is done, afterwards calling the new function to start.

``` {linenos = true}
	template <typename Func, typename ...Args>
	inline Task<_INVOKE_RESULTS_FTA>* then(Func&& func, Args&& ...args) 
	{
		using _Ret = _INVOKE_RESULTS_FTA;
		Task<_Ret>* ret = new Task<_Ret>();
		std::thread cont = std::thread(
			[=](Task<_Ret>* newTask, Task<T>* prevTask, Func func, Args ...args)
			{
#ifdef JC_DEBUG
				std::cout << "async thread: " << std::this_thread::get_id() << std::endl;
#endif
				while (prevTask->GetStatus() != std::future_status::ready)
				{
#ifdef JC_DEBUG
					if (!prevTask->HasInit()) {
						std::cout << "waiting for prev to init: " << std::to_string(UPDATE_TIME / 1000000.0f) << "s\n";
					}
					else
						std::cout << "waiting for prev to finish: " << std::to_string(UPDATE_TIME / 1000000.0f) << "s\n";
#endif
					std::this_thread::sleep_for(std::chrono::microseconds(UPDATE_TIME));
				}
				newTask->Init(func, prevTask->get(), std::forward<Args>(args)...); 
			}
			, ret, this, std::forward<Func>(func), std::forward<Args>(args)...);
		
			cont.detach();
		return ret;
	}
```

let me just break down how the above code works.

1. The function declaration is made using std::results_of_t or std::invoke_results. (you can look up the docs [here](https://en.cppreference.com/w/cpp/types/result_of))
The invoke results returns the return type of the function. This is to get the continued function's return type to return the results to the caller.

Note: _INVOKE_RESULTS_FTA is a macro I made based on different invoke results conditions, please contact me details.

``` {linenos = true}
inline Task<_INVOKE_RESULTS_FTA>* then(Func&& func, Args&& ...args) 
```

2. Creating the new task that should be continued

Next, I created the new task that should be continued using the return type of the function passed.
Then, I used a lambda function that waits for the previous task to end, then initialising the next continued function. During this, I used std::future_status to check whether the previous task has ended, else I will wait for the task for x seconds.
Lastly, I called the lambda function to a new thread to move the "blocking" to the other thread.

``` {linenos = true}
using _Ret = _INVOKE_RESULTS_FT;
Task<_Ret>* ret = new Task<_Ret>();
		std::thread cont = std::thread(
			[=](Task<_Ret>* newTask, Task<T>* prevTask, Func func)
			{
				while (prevTask->GetStatus() != std::future_status::ready)
				{
					std::this_thread::sleep_for(std::chrono::microseconds(UPDATE_TIME));
				}
				newTask->Init(func, prevTask->get());
			}
		, ret, this, std::forward<Func>(func));
```

### Conclusion

Some quite time-consuming things when I was working on this was getting the std::invoke_results right, getting the status of the task, and making sure the functions can continue over each other despite the different types. If you would like to see my full version, you can contact me via my linkedin/ig/discord in the home page.