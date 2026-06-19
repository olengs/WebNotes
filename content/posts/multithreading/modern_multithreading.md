---
title: "Modern C++ Multithreading"
summary: "C++ Multithreading for C++17 and later"
date: 2026-06-12
tags: ["C++", "Multithreading"]
author: ["JC"]
draft: false
weight: 0
ShowToc: true
---

### Summary

Notes on C++17+ Multithreading

## std::thread

Create a thread via std::thread via a entry point. This entry point can be any Callable Object. (Fn, FuncPtr, Functor (Func Obj), lamdba, member function, static member func)
```c++{linenos=true}
#include <thread>
#include <iostream>

void hello() {
    //need to lock cout but will cover later
    std::cout << "Hello World!" << std::endl;
}

int main() {
    std::thread thr(hello);

    //error without next line
    thr.join();
}
```

std::thread must own its arguments, so if the arguments are passed by:
lvalue -> passed by value
rvalue -> passed by move
lref -> use std::ref
```c++{linenos = true}
#include <string>
#include <thread>
#include <iostream>

void hello(std::string first, const std::string& second, std::string& third, std::string&& fourth) {
    std::cout << first << " " << second << " " << third << " " << fourth << std::endl
}

int main() {
    std::string second = "second";
    std::string third = "third";
    std::string fourth = "fourth"
    std::thread thr(hello, "first", std::cref(second), std::ref(third), std::move(fourth));

    thr.join();
}
```

Class member functions works slight differently
```c++{linenos = true}
#include <thread>

class greeter {
    void hello() {}
    static void statichello() {}
};

int main() {
    greeter mygreeter{};
    std::thread thr(&greeter::hello, &mygreeter);
    std::thread thr2(&greeter::statichello);

    thr.join();
    thr2.join();
}
```

Lambda can capture local members
```c++ {linenos = true}
int main() {
    std::string hello = "hello";

    std::thread([&]() {
        //hello is a ref here
        std::cout << hello << std::endl;
    });
}
```