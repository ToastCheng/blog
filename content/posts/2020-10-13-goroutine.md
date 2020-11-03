---
title:  "Goroutine: process, thread, goroutine"
date:   2000-10-13T00:43:00+0800
tags: [golang, goroutine, backend]
categories: technique
---

Goroutine is a interesting concept, but how does it relate to the concept of thread? or other traditional OS stuff?

As programming in Go, this question occasionally comes up to my mind. After some research, I decide to write it down :).

### Basic OS knowledge: Process, Thread, Time slicing
In OS's aspect, it's all about scheduling. OS has the power to decides how to allots CPU to processes. Naturally there will be a lot of processes running at the same time in your computer. When you ask the OS to run a program, a process will be created for you. Such as browser, PDF reader, Spotify, and a lot of background processes. 

`// TODO: figure here`

Let's say there are 100 process existing. What OS does is to schedule them, make sure all of them get a limited time to access the CPU, that is called time slicing, the OS slice the time for each process.

`// TODO: figure here`

What about thread? 
Thread is helpful since a program may be complicated. Take browser as an example, you may have multiple tabs browsing different web pages. If the program processes purly sequentially, you can do nothing on a new tab until all the old tab done their jobs, which is not a smooth user experience. When thread comes into play, every thread can get a partial time slice to execute. In that the task that should be run parallel could be run in different thread, but within the same process. 

Initially when a process is created, within that process, a thread is created as well.
So, it's basically time slicing again, but in different level.

`// TODO: figure here`

A thread is lighter than a process, by lighter, it means it is faster to create, to terminate, and it is more space efficient. In other word, you can have things done more efficiently.

What if there is something that is more lighter than a thread?

### Goroutine
Now, what goroutine does is to push this time slicing concept further.
Before jumping into it, there is some concept worth to take a look.

`P`: Processor, (may be logical).
`M`: Machine, OS thread. 
`G`: Goroutine.

When does context switch take place?
1. Keyword `go`
2. Garbage Collection
3. System call
4. Blocked by atomic, mutex or channel operation

Net poll

Work stealing

https://stackoverflow.com/questions/5440128/thread-context-switch-vs-process-context-switch#:~:text=Process%20context%20switching%20involves%20switching%20the%20memory%20address%20space.&text=Thread%20switching%20is%20context%20switching,processes%20is%20just%20process%20switching).

`// TODO: `
