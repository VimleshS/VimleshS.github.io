---
title: Context, pipeline and cancellation
layout: post
published: true
category: programming
tags: [golang, context]
comments: true
---

This is a brief note on how to setup and handle cancellation in a context. Lets start by simulating arbitrary problem
statement of squaring a number and multiplying it with result obtained from upstream pipeline. Here, I am using 
fibonacci just to replicate long running process in upstream pipeline.

{% highlight golang %}
func fib(n int) int {
	if n == 1 || n == 2 {
		return 1
	}
	return fib(n-1) + fib(n-2)
}
{% endhighlight %}

Lets, create a first task which will square up a number and depends upon result of a subsequent pipeline.

{% highlight golang %}
func task1(cxt context.Context, num, fibOfNum int) int {
	intC := make(chan int)
	var r int

	t := num * num
	go func(cxt context.Context) {
		task2Res := task2(cxt, fibOfNum)
		intC <- t * task2Res
	}(cxt)

	select {
	case r = <-intC:
		return r
	case <-cxt.Done():
		fmt.Printf("task 1 done, %v\n", cxt.Err())
	}
	return r
}	
{% endhighlight %}

Lets, create another task which will be time consuming 

{% highlight golang %}
func task2(cxt context.Context, num int) int {
	intC := make(chan int)
	var r int

	go func(num int) {
		intC <- fib(num)
	}(num)

	select {
	case r = <-intC:
		return r
	case <-cxt.Done():
		fmt.Printf("task 2 done, %v \n", cxt.Err())
	}
	return r
}
{% endhighlight %}


In main, create a cancellable context and pass it to pipeline so that it can be cancelled or timedout.

{% highlight golang %}
	ctx := context.Background()
	d := time.Duration(20 * time.Second)
	ctx, cancel := context.WithTimeout(ctx, d)
	go func() {
		s := bufio.NewScanner(os.Stdin)
		s.Scan()
		cancel()
	}()

	result := task1(ctx, 5, 40)
	fmt.Println(result)

{% endhighlight %}

#### Output when pipeline exits gracefully,

   > 2558353875

   > #goroutines: 2


#### Output when pipeline is cancelled or timedout,

   > task 1 done, context canceled

   > 0

   > #goroutines: 3

:accept: Suggestion and changes are wellcome.