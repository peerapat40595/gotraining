

1. allocate
2. read
3. write

Consistent Accurate


```go
// All material is licensed under the Apache License Version 2.0, January 2004
// http://www.apache.org/licenses/LICENSE-2.0

// Sample program to show the basic concept of pass by value.
package main

func main() {

	// Declare variable of type int with a value of 10.
	count := 10

	// Display the "value of" and "address of" count.
	println("count:\tValue Of[", count, "]\tAddr Of[", &count, "]")

	// Pass the "value of" the count.
	increment(count)

	println("count:\tValue Of[", count, "]\tAddr Of[", &count, "]")
}

// increment declares count as a pointer variable whose value is
// always an address and points to values of type int.
//go:noinline
func increment(inc int) {

	// Increment the "value of" inc.
	inc++
	println("inc:\tValue Of[", inc, "]\tAddr Of[", &inc, "]")
}
```

AF => active frame

Mechanics
semantic => behavior

pointer -> indirect access

escape analysis => dicide how to allocate memory

# comparing 2 factory function

V1

```go

// createUserV1 creates a user value and passed
// a copy back to the caller.
//go:noinline
func createUserV1() user {
	u := user{
		name:  "Bill",
		email: "bill@ardanlabs.com",
	}

	println("V1", &u)

	return u
}
```
 main get copy of value
 
 memory in stack below active frame (no integrity)
 
 ```go
 // createUserV2 creates a user value and shares
 // the value with the caller.
 //go:noinline
 func createUserV2() *user {
 	u := user{
 		name:  "Bill",
 		email: "bill@ardanlabs.com",
 	}
 
 	println("V2", &u)
 
 	return &u // best practice
 }
```

*** memory in stack below active frame (no integrity)
Because of that
we cannot use stack anymore
Compiler will inject value to the heap
And pointer infer to value in the heap

Performance
Stack > Heap

Share up pointer -> heap

Guideline => return as &u better than assign address to variable
return &u or u only *** no *u

Decide how to keep memory at compile time

2 goroutine
have own stack, can not access each other
but can access stack

because some time address in stack change

```bash
# escape analysis
go build -gcflags "-m -m"
```
