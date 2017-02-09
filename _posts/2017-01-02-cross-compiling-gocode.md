---
title: Cross compiling go code
layout: post
published: true
category: programming
tags: [golang]
comments: true
---

Cross compiling go code is very easy, with the go compiler installed we can cross compile binaries by bootstraping the go toolchain.

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello from cross compiled binary"))
	})

	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatalln("Program failed to started")
	}
}
```

**windows to linux**

To cross compile for linux from window system

	SET GOOS=linux

and then fire a normal build command

	go build

**linux to windows**

	GOOS=windows go build


