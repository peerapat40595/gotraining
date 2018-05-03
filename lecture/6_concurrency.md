# Concurrency

Thread state
1. executing
2. runable
3. waiting

1 core

context switching 
    -> expensive
    -> save state
    
multi core

have caching system
    L1, L2, L3
    
Best thread pool number depend!
    ex. the best is 3
    4 slower => context switching
    2 slower => not eff

GO scheduler => run time

kernel mode
user mode

Go -> keep thread busy
Process already pool

No need thread
but need
-> Sync
-> Orchestration

5 Kids
go routine -> add more kid

### Orchestration Pattern

Go routine

var wg sync.WaitGroup
wg.Add(number of go routine)

wg.Wait() -> until everything done

```go
func init() {

	// Allocate one logical processor for the scheduler to use.
	runtime.GOMAXPROCS(1)
}
```

Orchestration vs Synchronization => diff concept

Channel -> slow
Use mutex

Get right answer -> when you sync everything but
It take longer than it should

Use sharing var at least at possible

# Channel

is not data structure
is not a queue

pointer -> sharing
channel -> signaling to 1 or more go routine

send and receive signal
like IO read and write

Do you need guarantee signal?
Unbuffered => Guaranteed but costly => need to wait and ensure
Buffer => not guaranteed

Signal with data
- unbuffered => guarantee
- buffer > 1 => no guarantee
- buffer = 1 => delayed guarantee

Signal without data => cancellation

# Channel Pattern

```go
//unbuffered (no size)
ch := make(chan string)
```

```go
// waitForTask: Think about being a manager and hiring a new employee. In
// this scenario, you want your new employee to perform a task but they need
// to wait until you are ready. This is because you need to hand them a piece
// of paper before they start.
func waitForTask() {
	ch := make(chan string)

	go func() {
		p := <-ch // wait until get signal
		fmt.Println("employee : recv'd signal :", p)
	}()

	time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
	ch <- "paper"
	fmt.Println("manager : sent signal")

	time.Sleep(time.Second)
	fmt.Println("-------------------------------------------------------------")
}
```


# Fan out pattern

```go
// waitForResult: Think about being a manager and hiring a new employee. In
// this scenario, you want your new employee to perform a task immediately when
// they are hired, and you need to wait for the result of their work. You need
// to wait because you need the paper from them before you can continue.
func waitForResult() {
	ch := make(chan string)

	go func() {
		time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
		ch <- "paper"
		fmt.Println("employee : sent signal")
	}()

	p := <-ch
	fmt.Println("manager : recv'd signal :", p)

	time.Sleep(time.Second)
	fmt.Println("-------------------------------------------------------------")
}
```

# without data

```go
// waitForFinished: Think about being a manager and hiring a new employee. In
// this scenario, you want your new employee to perform a task immediately when
// they are hired, and you need to wait for the result of their work. You need
// to wait because you can't move on until you know they are but you don't need
// anything from them.
func waitForFinished() {
	ch := make(chan struct{})

	go func() {
		time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
		close(ch)
		fmt.Println("employee : sent signal")
	}()

	_, wd := <-ch // or <-ch
	fmt.Println("manager : recv'd signal :", wd)

	time.Sleep(time.Second)
	fmt.Println("-------------------------------------------------------------")
}
```

# fanout but not for webservice because cannot connect 10K go routine to db at the same time
```go
// fanOut: Think about being a manager and hiring a team of employees. In
// this scenario, you want your new employees to perform a task immediately when
// they are hired, and you need to wait for all the results of their work.
// You need to wait because you need the paper from each of them before you
// can continue.
func fanOut() {
	emps := 20
	ch := make(chan string, emps) // buffered

	for e := 0; e < emps; e++ {
		go func(emp int) {
			time.Sleep(time.Duration(rand.Intn(200)) * time.Millisecond)
			ch <- "paper"
			fmt.Println("employee : sent signal :", emp)
		}(e)
	}

	for emps > 0 {
		p := <-ch
		fmt.Println(p)
		fmt.Println("manager : recv'd signal :", emps)
		emps--
	}

	time.Sleep(time.Second)
	fmt.Println("-------------------------------------------------------------")
}
```

```go
// drop: Think about being a manager and hiring a new employee. In
// this scenario, you want your new employee to perform a task but they need
// to wait until you are ready. As the employee finishes their task you don’t
// care to know they are done. All that’s important is whether you can or can’t
// send new work. If you can’t perform the send, then you know your employee is
// at capacity. At this point the new work needs to be discarded so things can
// keep moving.
func drop() {
	const cap = 5
	ch := make(chan string, cap)

	go func() {
		for p := range ch {
			fmt.Println("employee : recv'd signal :", p)
		}
	}()

	const work = 20
	for w := 0; w < work; w++ {
		select {
		case ch <- "paper":
			fmt.Println("manager : sent signal :", w)
		default:
			fmt.Println("manager : dropped data :", w)
		}
	}

	close(ch)
	fmt.Println("manager : sent shutdown signal")

	time.Sleep(time.Second)
	fmt.Println("-------------------------------------------------------------")
}

// cancellation: Think about being a manager and hiring a new employee. In
// this scenario, you want your new employee to perform a task immediately when
// they are hired, and you need to wait for the result of their work. This time
// you are not willing to wait for some unknown amount of time for the employee
// to finish. You are on a discrete deadline and if the employee doesn’t finish
// in time, you are not willing to wait.
func cancellation() {
	ch := make(chan string, 1)

	go func() {
		time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
		ch <- "paper"
		fmt.Println("employee : sent signal")
	}()

	tc := time.After(100 * time.Millisecond)

	select {
	case p := <-ch:
		fmt.Println("manager : recv'd signal :", p)
	case t := <-tc:
		fmt.Println("manager : timedout :", t)
	}

	time.Sleep(time.Second)
	fmt.Println("-------------------------------------------------------------")
}

```
