---
title:  "Goroutine: process, thread, goroutine"
date:   2020-10-13T00:43:00+0800
tags: [golang, goroutine, backend]
categories: technique
---

Goroutine is a interesting concept, but how does it relate to the concept of thread? or other traditional OS stuff?

As programming in Go, this question occasionally comes up to my mind. After some research, I decide to write it down :).

### Basic OS knowledge: Process, Thread, Time slicing

In OS's aspect, it's all about scheduling. OS has the power to decides how to utilize CPU. Naturally there will be a lot of program running at the same time. Such as browser, PDF reader, Spotify, and a lot of background process. When you ask the OS to run a program, a process will be created for you.

`// TODO: figure here`

Let's say there are 100 process existing. What OS does is to schedule them, make sure all of them get a limited time to access the CPU, that is called time slicing, the OS slice the time for each process.

`// TODO: figure here`

What about thread? 
Thread is helpful since a program may be complicated. Take browser as an example, you may have multiple tabs browsing different web pages. If the program processes purly sequentially, you can do nothing on a new tab until all the old tab done their jobs, which is not a smooth user experience. When thread comes into play, every thread can get a partial time slice to execute. In that the task that should be run parallel could be run in different thread, but within the same process. 

Initially when a process is created, within that process, a thread is created as well.
So, it's basically time slicing again, but in different level.

`// TODO: figure here`

### Goroutine
Now, what goroutine does is to push this time slicing concept furthur.

`// TODO: `
