# Mechanic

Predictable access pattern
array
index++ -> make your program faster

Watch c c++

CPU Caches and Why You Care - Video - Scott Meyers
NUMA Deep Dive Series - Frank Denneman

for range
    copy of string
    value semantic

```go
// Using the pointer semantic form of the for range.
	five := [5]string{"Annie", "Betty", "Charley", "Doug", "Edward"}
	fmt.Printf("Bfr[%s] : ", five[1])

	for i := range five {
		five[1] = "Jack"

		if i == 1 {
			fmt.Printf("Aft[%s]\n", five[1])
		}
	}

	// Using the value semantic form of the for range.
	five = [5]string{"Annie", "Betty", "Charley", "Doug", "Edward"}
	fmt.Printf("Bfr[%s] : ", five[1])

	for i, v := range five {
		five[1] = "Jack"

		if i == 1 {
			fmt.Printf("v[%s]\n", v)
		}
	}

	// Using the value semantic form of the for range but with pointer
	// semantic access. DON'T DO THIS.
	five = [5]string{"Annie", "Betty", "Charley", "Doug", "Edward"}
	fmt.Printf("Bfr[%s] : ", five[1])

	for i, v := range &five {
		five[1] = "Jack"

		if i == 1 {
			fmt.Printf("v[%s]\n", v)
		}
	}
```

```go
	var data []string -> nil -> null
	data := []string{} -> pointer -> []
```

append
    value semantic mutation api

slice -> ref 
    
Mem leak
    -> go routine
    -> map as cache, need delete key
    -> append use diff assignment