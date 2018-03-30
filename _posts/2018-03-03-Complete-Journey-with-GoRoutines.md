---
layout: post
title: Complete Journey with Goroutines 
date: 2018-03-03 03:30:09
comments: true
categories:
- Computer Science 
tags:
    - Programming 
    - Go
    - Concurrency
    - GoNewbie
---

GoRoutines are mainly related to the concurrency model in Golang. Whenever we want things to be done concurrently we use goroutines. 
Wait. What is concurrency? 
In very simple words, concurrency means dealing with a lot of things at once(does not need to be at the same time). For an example, consider, If I was both typing and thirsting, then I will stop typing then drink water and then start typing again. Here I am dealing with two jobs(typing and drinking water) by some time slice which is said to be concurrent jobs.

## Overview  

In this blog, I will mainly be talking about 
* [What are Go Routines?](#what-are-go-routines)
* [How are they different from threads?](#how-are-they-different-from-threads)
* [Scheduling of Go Routines](#scheduling-of-go-routines)
* [Common mistakes while using Go Routines and how to avoid them?](#common-mistakes-while-using-go-routines-and-how-to-avoid-them)
* [Keywords](#keywords)

## What are Go Routines?
Go Routines are the way of doing the jobs concurrently in golang. They allow us to create and run the multiple methods or functions concurrently in the same address space inexpensively. We can say that the idea of GoRoutines are inspired by [CoRoutines](https://en.wikipedia.org/wiki/Coroutine). In my opinion the only difference is that CoRoutines supports the explicit mean of transferring the control to other CoRoutines while GoRoutines have it implicitly. (This point will get clearer in the scheduling section of this blog). GoRoutines are lightweight abstraction over threads, because their creation and destruction are very cheap as compared to threads and they are scheduled over OS threads. 
Executing the methods in the background is as easy as prepending the word `go` in a function call. Let’s see with a simple example.

```go
package main

import (  
    "fmt"
    "time"
)

func learning() {  
    fmt.Println("My first goroutine")
}
func main() {  
    go learning()
    /* we are using time sleep so that the main program does not terminate before the execution of goroutine.*/
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}

```
[Try](https://play.golang.org/p/UlpPlcIsyW1)

The output of the above program will be:

```shell
My first goroutine
main function
```

Remove the time.Sleep and then run the program again. You will notice that the output will be like this:

```shell
main function
```

To understand this we need to know how goroutines are executed? In Golang unlike normal functions, the control does not wait for the Goroutine to finish executing. Control immediately returns to the next line of the code after Goroutine call. The main Goroutine(main function is also a Goroutines) should be running for any other Goroutines to run. If the main Goroutine terminates then the program will be terminated and no other Goroutine will run. Hence when we are not using time.Sleep() then the controller immediately execute the line fmt.Println("main function) just after the Goroutine call and after this the main program gets terminated and we do not get the output of the Goroutine on the terminal.

*Using sleep in the main Goroutine is a hack we are using here just to understand how Goroutines work. Mainly we use [channels](https://www.sohamkamani.com/blog/2017/08/24/golang-channels-explained/) to block the main Goroutine until all other Goroutines finish their execution.*

## How are they different from threads?

Many people think that Goroutines are faster than threads. This is not entirely correct. It’s not any faster,it also allows you to do things concurrently but if things are right, the runtime will also split the work across multiple processors. So you_may_get parallelism out of it, too. But think of it as a way of doing things concurrently. If task A is blocked on something (waiting on I/O), the scheduler understands that another goroutine that’s ready to run can be executed while waiting for the I/O to return.

GoRoutines have benefit over threads in:

**Memory consumption:** The creation of go routines requires much less memory as compared to threads. It requires 2kb of memory while threads requires 1Mb(~500 times as compared to goroutines). They are designed in the way that stack size of go routines can grow and shrink according to need of the application. There might be only one thread in the program with thousands of goroutines.

**Setup and Teardown cost:** Thread have significant setup and teardown costs because it has to request resources from the OS and return it once its done. While go routines are created and destroyed by the go runtime(it manages scheduling, garbage collection, and the runtime environment for goroutines) and those operations are pretty cheap.

![go runtime](/assets/images/runtime.png)
<p align="center">
  From: <a href = "http://www.cs.columbia.edu/~aho/cs6998/reports/12-12-11_DeshpandeSponslerWeiss_GO.pdf">Analysis of Go Runtime scheduler</a>
</p>


**Switch cost:** This difference is mainly because of the difference in the scheduling of goroutines and threads. Threads are scheduled *preemptively* (If a process is running for more than a scheduler time slice, it would preempt the process and schedule execution of another runnable process on the same CPU), the schedular needs to save/restore all registers i.e. 16 general purpose registers, PC (Program Counter), SP (Stack Pointer), segment registers etc. While Go routines are scheduled *cooperatively*,(explained in the scheduling section) they do not directly talk to the OS kernel. When a goroutine switch occurs very few registers(say 3) like program counter and stack pointer need to be saved/restored. For more details refer [this](https://electronics.stackexchange.com/questions/115286/what-processor-registers-are-saved-and-recovered-in-a-context-switch). 

## Scheduling of Go Routines

As I have mentioned in the last paragraph that goroutines are cooperatively scheduled. In cooperative scheduling there is no concept of scheduler time slice. In such scheduling goroutines yield the control periodically when they are idle or logically blocked in order to run multiple goroutines concurrently.
The switch between goroutines happens only at well defined points, when an explicit call is made to the Go runtime scheduler. And those well defined points are like:
- Channels send and receive operations, if those operations would block.
- The Go statement, although there is no guarantee that new goroutine will be scheduled immediately.
- Blocking syscalls like file and network operations.
- After being stopped for a garbage collection cycle.

Now lets see how they are scheduled internally. Go uses three entities to explain the goroutine scheduling.
- Processor (P) 
- OSThread (M)
- Goroutines (G)

In any particular Go application the number of threads available for go routines to run is equal to the GOMAXPROCS, which by default is equal to the number of cores available for that application. We can use the `runtime` package to change this number at runtime as well. OSThreads are scheduled over processors and goroutines are scheduled over OSThreads as explained in the Fig 2.

`Go` has an M:N scheduler that also utilizes multiple processors. At any time, M goroutines need to be scheduled on N OS threads that runs on at most [GOMAXPROCS](#gomaxprocs) numbers of processors(`N <= GOMAXPROCS`). Go scheduler distribute runnable goroutines over multiple worker OS threads that runs on one or more processors.


![go runtime](/assets/images/GoRoutines_Scheduling.png)
<p align="center">
  Fig 2 
</p>


Every P has a local goroutine queue and also there is a global goroutine queue which contains runnable goroutines. Each M should be assigned to a P. Ps may have no Ms if they are blocked or in a system call. At any time, there are at most GOMAXPROCS number of P and only one M can run per P. More Ms can be created by the scheduler if required.

![go runtime](/assets/images/stealing.png)
<p align="center">
  Fig 3 
</p>

In each round of scheduling scheduler finds a runnable G and execute it. Once a runnable G is found, it gets executed until it is blocked(as explained above). How does scheduler search for a runable goroutine. 

```
        Search in the local queue
        if not found 
            Try to steal from other Ps' local queue //see Fig 3
            if not found 
                Search in the global queue 

        Also periodically it searches in the global queue (every ~ 1/70)

```

## Common mistakes while using Go Routines and how to avoid them?

**Not checking the FD limit and memory limit:**  You should always check the resources (like file descriptor and memory) limit required for your go program when you are creating huge number of goroutines and inside each goroutines either you are allocating large memory or dealing with files. Otherwise if memory limit will exceeds then memory leak may occur and if FD limit will exceed then goroutines may get stuck. 

**Running goroutines with infinite loops:**  Try executing the following golang code snippet.
```go
package main
import "fmt"
import "time"
import "runtime"

func main() {
    var x int
    processors := runtime.GOMAXPROCS(0)
    for i := 0; i < processors; i++ {
        go func() {
            for { x++ }
        }()
    }
    time.Sleep(time.Second)
    fmt.Println("x =", x)
}
```
Lets execute the program:

```
$ GOMAXPROCS=8 go run test.go
```

We see that program never terminates. If we write the same program in C/C++, we will never observe such an issue. Now lets rerun the program with changing the line: `processors = runtime.GOMAXPROCS(0) -1`. Now the program will terminate properly and prints the result. This is surprising isn't it? you can read this [blog](http://www.sarathlakshman.com/2016/06/15/pitfall-of-golang-scheduler) to read more about it.


## Keywords 

*GOMAXPROCS*

In current version of go, GOMAXPROCS is used to control the number of threads available for goroutine execution to a particular go program. The hard limit is still on the number of CPU cores presented to the OS. The `GOMAXPROCS` option allows you to tune it down. By default, as of 1.5+ or 1.6+, `GOMAXPROCS` is set to `runtime.NumCPU()`. Fun trivia: in older versions of Go it was set to `1`, because the scheduler wasn’t as smart and GOMAXPROCS > 1 was extremely detrimental to performance.
 
