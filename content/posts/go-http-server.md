---
title: "secretProject part 3 - http.HandleFunc in Go"
date: 2022-11-01T16:03:06-05:00
draft: false
toc: false
images:
tags: 
  - dev
  - go
---

>Oh, what a tangled web we weave, when first we practice to deceive!!.
>
>-Sir Walter Scott
---

# Previously on codedinsugar.com

Now that we have our secretProject [Kubernetes cluster built](https://codedinsugar.com/posts/kubernetes-cluster-pt-2/), let's throw some custom apps in there!

# Keep it simple and sweet

This is NOT a Kubernetes (k8s) tutorial!  The k8s ecosystem is massive if you're into container orchestration, read their docs [here](https://kubernetes.io/docs/home/).  I use k8s everyday at work and at home and even though it's almost become second nature for me, I still don't know everything about it...

The purpose of this post is to:

* Build a simple http server in Go using only the standard library
* Deploy that app to k8s

# Listen, Serve, Handle

The end result of our http server is this:

```
// Test with curl -d 'data' localhost:9190
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

func main() {
	//HandleFunc is a convenience method that creates an http handler to respond to http requests
	http.HandleFunc("/", func(res http.ResponseWriter, req *http.Request) {
		data, err := ioutil.ReadAll(req.Body)
		if err != nil {
			http.Error(res, "Uh oh something didn't go right...", http.StatusBadRequest)
		} else {
			fmt.Fprintf(res, "Hello %s\n", data)
			log.Printf("Data: %s\n", data)
		}
	})

	http.HandleFunc("/goodbye", func(res http.ResponseWriter, req *http.Request) {
		data, err := ioutil.ReadAll(req.Body)
		if err != nil {
			http.Error(res, "Uh oh something didn't go right...", http.StatusBadRequest)
		} else {
			fmt.Fprintf(res, "Goodbye %s\n", data)
			log.Printf("Data: %s\n", data)
		}
	})

	http.ListenAndServe(":9190", nil) // 9090 is taken by Cockpit UI
}
```

But let's break down what's happening in `main()` even further...

```
http.HandleFunc("/", func(res http.ResponseWriter, req *http.Request)
```

If you traceback the references from [http.HandleFunc](https://pkg.go.dev/net/http#HandleFunc) ultimately you'll see that the [ServeMux](https://pkg.go.dev/net/http#ServeMux) type is inferred.  ServeMux is an HTTP request multiplexer. It matches the URL of each incoming request against a list of registered patterns and calls the handler for the pattern that most closely matches the URL.  The pattern that we're matching is the first argument of the function which is `"/"` aka the root path after the base domain.  The second argument is an interface to the [Handler](https://pkg.go.dev/net/http#Handler) type with a single method called `ServeHTTP` that accepts a [ResponseWriter](https://pkg.go.dev/net/http#ResponseWriter) interface type and a [Request](https://pkg.go.dev/net/http#Request) struct type.  These guys make the request magic happen!


```
data, err := ioutil.ReadAll(req.Body)
if err != nil {
	http.Error(res, "Uh oh something didn't go right...", http.StatusBadRequest)
} else {
	fmt.Fprintf(res, "Hello %s\n", data)
	log.Printf("Data: %s\n", data)
}
```

So so now we need to handle what's actually in the request.  The `.Body` method is of interface type [ReadCloser](https://pkg.go.dev/io#ReadCloser) which we'll assign to the `data` and `err` variable identifiers.  If `err` is satisfied then the `.Error` method will use `res` to return a specific string and the response code in `.StatusBadRequest`.  Simple enough yeah?

To test different request paths we'll create another handle but with a different signature using the `"/goodbye"` path.

```
http.ListenAndServe(":9190", nil)
```

Request and response, check.  Now what?  We have to tell the system running the http server (my local box) what port to actually listen on which is where [http.ListenAndServe](https://pkg.go.dev/net/http#Server.ListenAndServe) comes in at TCP port 9190.

# Run and Build

Coolio, let's see this bad boy in action:

```
go run main.go
// produces no output

curl -d 'codedinsugar.com' localhost:9190
Hello codedinsugar.com
// ran from a different terminal

2022/11/01 17:47:39 Data: codedinsugar.com
// returned from original terminal

---

curl -d 'testingGoodbye' localhost:9190/goodbye
Goodbye testingGoodbye
...
2022/11/01 17:48:30 Data: testingGoodbye
```

Beautiful and working as expected.  I wonder what else we can pass to `-d`?

```
curl -d $(curl ipinfo.io/ip) localhost:9190
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    13  100    13    0     0     36      0 --:--:-- --:--:-- --:--:--    36
Hello 78.255.xxx.xxx
...
2022/11/01 17:49:15 Data: 78.255.xxx.xxx
```

:)