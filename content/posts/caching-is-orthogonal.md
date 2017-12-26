+++
title = "Caching Is Always Orthogonal"
date = 2017-10-22T20:43:20-02:00
draft = true
summary = "I've found that caching is always an orthogonal responsability when comparing it to the main responsability of the module and using composition we can design better components."
+++


<!--more-->

I'm writing this post to share something I've come to realise about caching in terms of software design.
This might seem pretty obvious to some, and it certainly feels this way to me now, but I decided to write it anyhow.
Caching an expensive operation is a common strategy to squeeze out performance from a program. From a code organization perspective, it's easy to pollute the original code with caching instructions. I've found that caching is always an orthogonal responsability when comparing it to the main responsability of the module and using composition we can design better components.

**BAD**

```go
type Client struct {
	cache map[string][string]
}

func (c *Client) Fetch(s) string {
	if cached, ok := cache[s]; !ok {
		cache[s] = expensiveOperation() 
	}
	return cache[s]
}
```

** GOOD **

```go
type Fetcher interface {
	Fetch() string
}

type Client struct {
}

func (c *Client) Fetch(s) string {
	expensiveOperation()
}

type CachingClient struct {
	cache map[string][string]
	client Fetcher
}

func (c *CachingClient) Fetch(s) string {
	if cached, ok := cache[s]; !ok {
		cache[s] = c.Client.Fetch() 
	}
	return cache[s]
}
```

Comparing the two versions, we immediately see some obvious disadvantages:

* More code
* Two additional types
* Necessity for boilerplate code to initialize the complete functional client

Now let's analyse what we gained:

The introduction of an interface type adds flexibility to system. The upper levels of the code can depend on the interface instead of a specific implementation. We take advantage of this flexibility to introduce a Fetcher that decorates any other Fetcher, not just this specific one. Because the two are now independent, they are now composeable and therefore easier to reuse and to test. To sum it up:

* Flexibility
* Inversion of dependencies
* Composeability
* Easier to test

The tradeoff is pretty clear and the final decision depends on your specific context.

While this post focus on caching, we can apply the same pattern on a number of different scenarios to obtain flexible and composeable modules.
