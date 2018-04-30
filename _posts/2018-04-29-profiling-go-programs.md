---
title: Testing and profiling golang program
layout: post
published: true
category: programming
tags: [golang, profiling]
comments: false
---


## Testing and profiling golang program

A very simple application, we will use for demo. 

{% highlight golang %}

package main

import (
	"bytes"
	"fmt"
	"strings"
	"sync"
)

func main() {
	sentence := "The quick brown fox jumps over the lazy dog"
	words := Map(sentence)
	rwords := process(words)
	fmt.Println(reduce(rwords))
}

func process(words []string) []string {
	nosOfWords := len(words)
	buffChannel := make(chan string, nosOfWords)
	task := new(sync.WaitGroup)
	task.Add(nosOfWords)

	for _, word := range words {
		go func(word string) {
			defer task.Done()
			buffChannel <- reverse(word)
		}(word)
	}
	task.Wait()
	close(buffChannel)

	rwords := make([]string, 0)
	for rword := range buffChannel {
		rwords = append(rwords, rword)
	}
	return rwords
}

func Map(sentence string) []string {
	return strings.Split(sentence, " ")
}

func reduce(reverseWords []string) string {
	return strings.Join(reverseWords, " ")
}

func reverse(word string) string {
	var buff bytes.Buffer
	for index := len(word) - 1; index >= 0; index-- {
		buff.WriteString(string(word[index]))
	}
	return buff.String()
}
{% endhighlight %}

### Run It!
    go run main.go

or

    go build

## Test
Create a file and save it as `main_test.go`
Execute test case as 

	go test

Its time to add first test case, before we move ahead let me tell there are three types of test that we can write in golang

* Unit Test
* Benchmark Test
* Example Test

#### Unit Test

Lets write first test case 

{% highlight golang %}
func TestMap(t *testing.T) {
	
	//Arrange
	phrase := "The quick brown fox jumps over the lazy dog"
	
	//Act
	slicedWord := Map(phrase)
	
	//Assert
	if len(slicedWord) != 9 {
		t.Log("Test failed, nos of words returned are incorrect")
		t.Fail()
	}
}
{% endhighlight %}

A short representaion dipicting effect of log and Fail/Failnow combinations


  Log()    | Fail()/FailNow()  | Net Effect  | Desc                                    
  -------- | ----------------- | ----------- |:--------------------------------------
  t.Log    |  t.Fail()         |  t.Error()  | signals failure                         
  t.Log    |  t.FailNow()      |  t.Fatal()  | abort further execution of test cases   

<br/>  

{% highlight golang %}

func TestMapForBlankSentence(t *testing.T) {
	words := Map("")
	if len(words) != 0 {
		t.Error("TestMapForBlankSentence failed")
	}
}

func TestReverse(t *testing.T) {
	if reverse("Hello") != "olleH" {
		t.Fatal("Reverse is incorrect")
	}
}

{% endhighlight %}	

#### Skipping long test
{% highlight golang %}
func TestReverse(t *testing.T) {
	if testing.short(){
		t.Skip()
	}
	if reverse("Hello") != "olleH" {
		t.Fatal("Reverse is incorrect")
	}
}

func TestReverse(t *testing.T) {
	if testing.Verbose(){
		b.Skip("Reverse skipped")
	}
	if reverse("Hello") != "olleH" {
		t.Fatal("Reverse is incorrect")
	}
}
{% endhighlight %}

#### Executing tests in parallel

Add t.Parallel() in test case, to flag test runner that execute this test in parallel.

{% highlight golang %}
func TestReverse(t *testing.T) {
	t.Parallel()
	if reverse("Hello") != "olleH" {
		t.Fatal("Reverse is incorrect")
	}
}
{% endhighlight %}

#### Table Driven Test

Create a slice of anonymous type and use iteratively to check for series of inputs.

{% highlight golang %}
func TestReverseForMultipleInput(t *testing.T){
	if testing.Short() {
		t.Skip("TestReverseForMultipleInput skipped")
	}
	 tests := []struct{
		input string
		output string
	}{
		{"Hello", "olleH"},
		{"benchmarks", "skramhcneb"},
		{"provide","edivorp"},
		{"flag","galf"},
	}
	for _,test := range tests {
		if reverse(test.input) != test.output {
			t.Fatal("TestReverseForMultipleInput failed")
		}
	}
}
{% endhighlight %}

#### Code Coverage

	 go test -cover

or generate a coverage profile and view it in web.

	go test -coverprofile=cover.out
	go tool cover -html=cover.out

## Benchmark

A stand alone benchmark incorporated in a main program, but it is always nice to have benchmark in a test suite

{% highlight golang %}
br := testing.Benchmark(func( b *testing.B){
	/*
		is a stand alone function that runs independtly of test runner.
	*/
})
// here we can get the benchmark metrics
{% endhighlight %}

Lets write a first bechmark to see the metrics.

{% highlight golang %}
func BenchmarkMap(b *testing.B) {
	b.ReportAllocs()
	for i := 0; i < b.N; i++ {
		Map("There is no flag you can provide, that will run only benchmarks")
	}
}
{% endhighlight %}

Add `b.ReportAllocs()` in code as above for getting allocation metrics, while performing the benchmarking alternatively we get this from a
`benchmem` flag. Also, there is no flag you can provide, that will run only benchmarks (or only one benchmark). 

The only flags related to these are:

* `bench regexp` Run benchmarks matching the regular expression. By default, no benchmarks run. 
* `run regexp` Run only those tests and examples matching the regular expression.

using these flags in association with other flags will fire on specfified benchmark

	go test -bench=Map$ -run=^$ -benchmem
	BenchmarkMap-4            100000             15030 ns/op           41056 B/op         37 allocs/op
	PASS
	ok      github.com/VimleshS/go_technext 5.062s


`Timers` in benchmark. Moment we kick off benchmark test, becnhmark start time is logged, but there are certain time that we need to reset/pause monitoring time. e.q while arranging for benchmark test. There are properties provided by testing package to do this set of operation.

* b.ResetTimer()
* b.StopTimer()
* b.StartTimer()


{% highlight golang %}
func BenchmarkProcess(b *testing.B) {
	b.StopTimer()
	words := []string{"There", "is", "no", "flag", "you", "can", "provide", ",",
		"that", "will", "run", "only", "benchmarks"}
	b.StartTimer()
	for i := 0; i < b.N; i++ {
		process(words)
	}
}
{% endhighlight %}

### Generating profiles

#### CPU profiling

	go test -bench=Map$ -run=^$ -cpuprofile=cpu.prof
	go tool pprof Word_reversal.test.exe cpu.prof

#### Memory profile

| Flag           |  Desc                             | 
| ---------------|-----------------------------------|
| inuse_space    |  Display in-use memory size       |
| inuse_objects  |  Display in-use object counts     |
| alloc_space    |  Display allocated memory size    |
| alloc_objects  |  Display allocated object counts  | 

<br/>

	go test -bench=Reverse$ -run=^$  -memprofile=prof.mem
	go tool pprof --alloc_space Word_reversal.test.exe prof.mem

#### Block profile	

	go test -run=^$ -bench=Process$ -blockprofile=bloc.prof
	go tool pprof Word_reversal.test.exe bloc.prof

#### Contention profile

{% highlight golang %}
func BenchmarkReduce(b *testing.B){
	words := []string{"olleH", "skramhcneb" }
	b.SetParallelism(30)
	b.RunParallel(func (pb *testing.PB){
		for pb.Next(){
			reduce(words)
		}
	})
}	
{% endhighlight %}