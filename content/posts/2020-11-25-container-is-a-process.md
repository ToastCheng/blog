---
title: "Docker: Container is a process"
date: 2020-11-25T18:43:00+0800
tags: [note, docker, container]
categories: technique
---

When I learn something new, I tend to link it to what I've already learned. So when I first heard docker, container, this kind of virtualizing technique. They just look like magic to me and they don't seem related to the text book at first glance. So how about making some connection between?

Some facts:
- Everything running on my computer is a process
- Operating system manages thoes process


- Mount: filesystems, / in the container will be different from / on the host.
- PID: process id's, pid 1 in the container is your launched application, this pid will be different when viewed from the host.
- Network: containers run with their own loopback interface (127.0.0.1) and a private IP by default. Docker uses technologies like Linux bridge networks to connect multiple containers together in their own private lan.
- IPC: interprocess communication
- UTS: this includes the hostname
- User: you can optionally shift all the user id's to be offset from that of the host




references:
[Container vs Process](https://sites.google.com/site/mytechnicalcollection/cloud-computing/docker/container-vs-process)
[docker implementation](https://philipzheng.gitbook.io/docker_practice/underly)