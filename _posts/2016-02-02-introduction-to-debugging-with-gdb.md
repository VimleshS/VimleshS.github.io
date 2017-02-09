---
title: Quick introduction to debugging golang application with gdb.
layout: post
published: true
category: programming
tags: [golang]
comments: true
---


I have spent most of my time coding in Delphi and .Net with has fantastic IDEs and debugging capabilities.

When I shifted to Golang I was debugging code by putting a detailed log, untill I learned to use gdb. This post is all about providing a consise steps so as to start using gdb. One can find detailed explanations at [golang/gdb](https://golang.org/doc/gdb) and [Gnu debugger](https://sourceware.org/gdb/current/onlinedocs/gdb/)

Before I move ahead let me borrow statement from golang website which reads “….the instructions below should be taken only as a guide to how to use GDB when it works, not as a guarantee of success.” and [<span style="color: red"> known issue and limitations</span>](https://golang.org/doc/gdb#Known_Issues)

For a sake a simplicity and demo purpose I’m choosing a small web application developed in gin-gonic as below

<script src="https://gist.github.com/VimleshS/7dc4085ca8d0d2773094.js"></script>

**How to build go code**

	go build -gcflags "-N -l"

This option prevents the compiler to optimize go-code and helps in debugging with gdb

**Few command operations**

1. To list a function or file use “list”/”l”
   - This works well either with function name or line number
2. To set a breakpoint use “break”/”b”
   - This also works well either with function name or line number
3. To starts a program use “run”
4. To executes next line use “next”/”n”
5. To prints value of expression use “print”/”p”

**Start debugging in 4 steps**

Step 1
Build program with the flag provided at the top and launch gdb debugger

<p align="middle">
    <img src="/assets/images/debugging_with_gdb/step-1.png" alt="Build" class="img-responsive img-thumbnail">
</p>

Step 2
“list” the code portion if that is required or set directly break point
<p align="middle">
    <img src="/assets/images/debugging_with_gdb/step-2.png" alt="Build" class="img-responsive img-thumbnail">
</p>

Step 3
Now start the debugger to begin with debugging
<p align="middle">
    <img src="/assets/images/debugging_with_gdb/step-3.png" alt="Build" class="img-responsive img-thumbnail">
</p>

Step 4
I have used curl to post a request to web server

	curl -iv -XPOST -d'{"User": "username", "Password": "password"}' -H"Content-type: application/json" http://localhost:8080/login

In second half of screen, execution has paused at the break point, use “next”/”step” or “continue” what ever is your desired debugging choice

<p align="middle">
    <img src="/assets/images/debugging_with_gdb/step-4.png" alt="Build" class="img-responsive img-thumbnail">
</p>



