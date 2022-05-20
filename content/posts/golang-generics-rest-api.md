---
title: "REST APIs with Go Generics"
date: 2022-03-14T16:26:05+01:00
draft: false
---

We're on the verge of Go 1.18 being released, with the headline feature of Generics going mainstream after many years of discussion. But what are generics?

I am primarily involved in the development of networked services. These consist of a mixture of HTTP REST and gRPC services. A nice element of working with a gRPC service is the method signature of your endpoints are generated for you, you simply provide the implementation. We have no such thing for the HTTP REST services.

Do we replicate the code generation benefits of gRPC, but for the HTTP REST services? Or, could we use generics?

Consider a hypothetical service that generates friendly greetings for users, it may define an request and response structure, accepting a name and returning the greeting:

```go
type GreetRequest struct {
	Name string `json:"name"`
}

type GreetResponse struct {
	Greeting string `json:"greeting"`
}
```

If we continue with principles derived from gRPC, the ideal method signature of the function handling this request may look something like:

```go
func Greet(ctx context.Context, req *GreetRequest) (*GreetResponse, error) {
	return &GreetRes{Greeting: "Hello " + req.Name + "!"}, nil
}
```

However the above does not conform to any of interfaces and types routinely expected by HTTP routers, in particular `http.Handler` and `http.HandlerFunc`.

Prior to Go 1.18 and Generics, you are left with two options: write glue logic for every one of your API endpoints or use runtime reflection and empty interfaces, to adapt your endpoint to your chosen router.

Now with generics, we can implement a function that can wrap any function implementing a similar method signature into a JSON API, inheriting the compile-time type safety of Go. What might that look like? Consider the following generic function:

```go
func JSON[REQ, RES any](fn func(context.Context, *REQ) (*RES, error)) http.HandlerFunc
```

Let's analyze what we just did.

The first obvious difference is the square brackets, these identify to the compiler we are writing a generic function. Specifically, what are the names and the types the compiler will substitute in our generic function for each instance we use our generic function. `REQ` and `RES` correspond to our request and response structures, but can be anything.

However we can now implement the rest of our function as we otherwise would, except using `REQ` and `RES` in place of our actual request and response structures. In a simplified implementation, let's unmarshal JSON from the HTTP request, execute our endpoint's function and marshal JSON back to the HTTP client:

```go
return func(w http.ResponseWriter, r *http.Request)
	req := new(REQ)

	json.NewDecoder(r.Body).Decode(req)

	res, err := fn(r.Context(), req)
	if err != nil {
		// handle error
	} else {
		json.NewEncoder(w).Encode(res)
	}
}
```

That's it. Our generic `JSON` function can be used to adapt any of our API endpoint functions to our HTTP router. i.e.

```go
router.Post("/v1/greeting", JSON(Greet))
```

And now, assuming everything is correct, calling the endpoint, we are greeted with the expected response:

```
POST /1/greeting HTTP/1.1
{"name": "James"}

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"greeting": "Hello James!"}
```

This post only scratches the surface of where Go Generics can help us reduce repetition and introduce compile-time type safety to our code. I will continue expanding on this example in future blog posts on how we can use Go Generics to improve building APIs.
