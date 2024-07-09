---
title: "Async continuation for c++"
summary: "My take on continuation for c++"
date: 2024-07-08
tags: ["c++", "async", "multithreading"]
author: ["JC"]
draft: true
weight: 0
ShowToc: true
---



### Summary

In this post, I will be talking about the process of me making a continuation function for std::async to chain async calls and things that I have learnt while doing this. Why did I make this? I wanted to chain async functions using std::async like c# where u can do task.then().then(); Hence, I gave it a try in hopes of learning something new.



