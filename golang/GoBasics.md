#technotes #go #concept 

# Go Basics

Notes from [The Little Go Book](https://github.com/karlseguin/the-little-go-book)



**Go** is compiled, statically typed language with **C** like syntax and garbage collection

## How to execute

```bash
go run main.go			# execute the code
go run --work main.go		# execute the code and show temp dir
gi build main.go		# compile the code
```

## Variables and Declarations

Declare a variable `power`  of type `int` and assign value `9000` to variable

```go
package main

import (
	"fmt"
)

func main() {
	var power int
	power = 9000
	fmt.Printf("It's over %d\n", power)
}
```

Other ways to defind the same variable

```go
// above two lines can be combined into one line
var power int = 9000

// using ':=', this will infer the type
// but can only be used inside funtions as local variable
power := 9000
```

 `:=` can also be used to assign funtion to a variable
 
 ```go
func main(){
	power := getPower()
}

func getPower() int {
	return 9001
}
```

A variable cannot be declared twice

```go
func main() {
	power := 10000
	fmt.Printf("Default Power is %d\n", power)
	
	// COMPILER ERROR
	// no new variables on left side of :=
	// error says so because we can declare multiple variables together 
	// so if `power` needs to be redeclared, it can be done along with another variable
	power := 9000
	fmt.Printf("Power is %d\n", power)
	
	// this works as `power` is redeclared with another variable `name`
	// but the type of variable cannot be changed
	name, power := "Goku", 9000
	fmt.Printf("bname is %s  and Power is %d\n", name, power)
}
```

> **NOTE**
> When a value is not assigned to a variables, By default, Go assigns a `zero` value to variables.

## Function Declarations

In Go, funtions can return multiple values including a funtion itself

```go
// takes 1 variable ass input, `message` which is string
func log(message string) {
}

// takes 2 ints and returns 1 int
func add(a int, b int) int {
}
// same can also be written as
func add(a, b int) int {
}

// takes a string and returns a int and bool
func power(name string) (int, bool) {
}

```

> **NOTE**
> `_` can be used to ignore one or more variables returned by a funtion

```go
// use both the vaues returned
value, exists := power("goku")
if exists == false {
 // handle this error case
}

// ignore one of the variable
_, exists := power("goku")
if exists == false {
 // handle this error case
}
```

## Structures

Declare and initialize a struct

```go
// define a struct
type Saiyan struct{
	Name string
	Power int
}

// declare and initialize a struct
goku := Saiyan{
	Name: "Goku",
	Power: 9000,
}

// decalre zero value struct
goku := Saiyan{}

// assign value
goku := Saiyan{Name: "Goku"}
goku.Power = 9000
// or
goku := Saiyan{"Goku", 9000}
```

### Why use Pointers

Go passes arguments to functions **as copies** instead of passing the actual variables

```go
func main(){
	goku := Saiyan{"Goku", 9000}
	Super(goku)
	fmt.Println(goku.Power)	// OUTPUT : 9000
}
func Super(s Saiyan){
	s.Power += 10000
}
```

Above example will have output as `9000` as *s* in *Super* was just a copy and does not affect the original variable. This is why Pointer is used to pass the address of the variable so the changes made by the funtion is refelected

```go
func main(){
	// goku has the address of the struct
	goku := &Saiyan{"Goku", 9000}
	Super(goku)
	fmt.Println(goku.Power)	// OUTPUT : 19000
}
// Super receives pointer as input
func Super(s *Saiyan){
	s.Power += 10000
}
```

### Functions on Structures

Methods can be associated with Structs

```go
type Saiyan struct {
	Name string
	Power int
}

// *Saiyan is called as `receiver` of the Super method
func (s *Saiyan) Super() {
	s.Power += 10000
}

goku := &Saiyan{"Goku", 9001}
goku.Super()
fmt.Println(goku.Power) // will print 19001

```

### Constructor / Factory

Funtions are used to construct to construnct values and return instance of desired type (like a factory)

```go
// factory returns pointer to the struct constructed
func NewSaiyan(name string, power int) *Saiyan {
	return &Saiyan{
		Name: name,
		Power: power,
	}
}

// factory can also return value instead of a pointer
func NewSaiyan(name string, power int) Saiyan {
	return Saiyan{
		Name: name,
		Power: power,
	}
}
```

### new

`new` can be used to allocate memory required by a type

```go
goku := new(Saiyan)
// same as
goku := &Saiyan{}

// comparing
goku := new(Saiyan)
goku.Name = "goku"
goku.Power = 9001
//vs
goku := &Saiyan {
	Name: "goku",
	Power: 9000,
}
```

### Fields of a Structure

Fiels of structure can be of any type including a struct

```go
type Saiyan struct {
	Name string
	Power int
	Father *Saiyan
}

gohan := &Saiyan{
	Name: "Gohan",
	Power: 1000,
	Father: &Saiyan {
		Name: "Goku",
		Power: 9001,
		Father: nil,
	},
}
```

### Composition

Go supports *composition*, which is the act of including one structure into another.

```go
type Person struct{
	Name string
}

func (p *Person) Introduce(){
	fmt.Printf("Hi, I am %s\n", p.Name)
}

type Saiyan struct {
	*Person
	Power int
}

func main() {
	goku := &Saiyan{
		Person: &Person{"Goku"},
		Power:  9001,
	}
	goku.Introduce()	// prints "Hi, I am Goku"
	
	// both the lines below will print "Goku"
	fmt.Println(goku.Name)
	fmt.Println(goku.Person.Name)

}
```

> **Note**
> Go doesn't support **overloading**.

##  Maps, Arrays and Slices

### Arrays

Declaring an array requires that we specify the size, and once the size is specified, it cannot grow and is fixed.

```go
// save up to 10 scores using indexes scores[0] through scores[9].
var scores [10]int
scores[0] = 350

// declare and initialize
scores := [4]int{9001, 9333, 212, 33}

// find the length of Array using `len`
len(scores)

// iterate over Array using `range`
for index, value := range scores {

}
```

Arrays are efficient but rigid. We often don't know the number of elements we'll be dealing with upfront. For
this, we turn to slices.

### Slices

In Go, you rarely, if ever, use arrays directly. Instead, you use slices. A slice is a lightweight structure that wraps
and represents a portion of an array.

```go
// no size mentioned inside the brackets
scores := []int{1,4,293,4,9}

// using `make` keyword
scores := make([]int, 10)

// create slice with length of 5 but with capacity of 10
scores := make([]int, 5, 10)
```

> **new vs make**
> *new* only allocates memory
> *make* allocates memory and initializes the Slice

```go
// ERROR : panic: runtime error: index out of range [7] with length 0
// because slice has a length of 0
func main() {
	scores := make([]int, 0, 10)
	scores[7] = 9033
	fmt.Println(scores)
}

// explicitly expand slice
// appending to a slice of length 0 will set the first element.
func main() {
	scores := make([]int, 0, 10)
	scores = append(scores, 5)
	fmt.Println(scores) // prints [5]
}

// re-slice slice
func main() {
	scores := make([]int, 0, 10)
	scores = scores[0:8]
	scores[7] = 9033
	fmt.Println(scores) // primts "[0 0 0 0 0 0 0 9033]"
}
```

We can resize a Slice till its capacity is met. After that `append` can be used to grow the slice. It creates a new larger array and copies the values over from the underlaying array in Slice. Go grows array with **2x** algorithm. 

```go
func main() {
	// Slice is initialized with length of 5	
	scores := make([]int, 5)
	fmt.Println(scores) // prints [0 0 0 0 0]
	// append a value to a slice that already holds 5 values
	scores = append(scores, 9332)
	fmt.Println(scores)	// prints [0 0 0 0 0 9332]
}
```

4 common ways to initialize a Slice

```go
names := []string{"leto", "jessica", "paul"}
checks := make([]bool, 10)
// nil slice
var names []string
scores := make([]int, 0, 20)
```

Slice is a datastructure of array
```go
func main() {
	scores := []int{1, 2, 3, 4, 5}
	slice := scores[2:4]
	slice[0] = 999
	fmt.Println(scores) // prints [1 2 999 4 5]
}
```

Slice of Slice

```go
func main() {
	scores := []int{1, 2, 3, 4, 5}
	fmt.Println(scores[2:])  // prints [3 4 5]
	fmt.Println(scores[1:2]) // prints [2]
	fmt.Println(scores[:3])  // prints [1 2 3]
}
```

***copy*** function can be used to copy Slices

### Maps

Maps in Go are what other languages call hashtables or dictionaries. 
Define a *key* and *value*, and can *get*, *set* and *delete* values from it.
Maps, like slices, are created with the *make* function. 

```go
func main() {
	lookup := make(map[string]int)
	lookup["goku"] = 9001

	power, exists := lookup["goku"]
	fmt.Println(power, exists) // prints 9001, true

	power1, exists1 := lookup["vegeta"]
	fmt.Println(power1, exists1) // prints 0, false

	fmt.Println(len(lookup)) // prints 1

	// has no return, can be called on a non-existing key
	delete(lookup, "goku")
	fmt.Println(len(lookup)) // prints 0

	// to set initial size of map to 100
	lookup1 := make(map[string]int, 100)
}
```

Map with struct

```go
type Saiyan struct {
	Name    string
	Friends map[string]*Saiyan
}

func main() {
	goku := &Saiyan{
		Name:    "Goku",
		Friends: make(map[string]*Saiyan),
	}

	goku.Friends["krillin"] = "..."

	lookup := map[string]int{
		"goku":  9001,
		"gohan": 2044,
	}

	for key, value := range lookup {
	}
}
```

## Code Organization and Interfaces

### Package

In Go, package names follow the directory structure of the Go workspace - `$GOPATH/src/<workspace-name>/`

> **TIP**
> Instead of having all the code in the same directory, it is better to seperate a logic into its own sub-directory
> eg: database logic can be put into  `$GOPATH/src/<workspace-name>/db`

#### Cyclical Imports

Cyclical imports happens when package A imports package B but package B imports package A (either directly or indirectly through another package). This is something the compiler won't allow.

#### Visibility

Go uses a simple rule to define what types and functions are visible outside of a package. If the name of the
type or function starts with an uppercase letter, it's visible. If it starts with a lowercase letter, it isn't.

This also applies to structure fields. If a structure field name starts with a lowercase letter, only code within the
same package will be able to access them.

#### Package Management

The *go* command along with  *run* and *build* has a *get* subcommand which is used to fetch third party libraries. go get supports various protocols to install packages from websites

```bash
go get github.com/mattn/go-sqlite3
```

#### Dependency Management

`go get` within a project scans all the files, looking for imports to third-party libraries and will download them.
 `go get -u` will update all the packages 
 `go get -u FULL_PACKAGE_NAME` will update specific package
 
 There's no way to specify a revision, it always points to the master/head/trunk/default. To solve this, we can use a third-party dependency management tool like *goop* and *godep*
 
### Interfaces
 
 Interfaces are types that define a contract but not an implementation. Interfaces help decouple the code from specific implementations. For example, we might have various types of loggers.
 
 ```go
// define interface
type Logger interface {
	Log(message string)
}

//  The above mean, if a struct has a function name `Log` with a `string` parameter and no return value, then it can be used as a Logger.
// We can define multiple loggers

// SqlLogger
type SqlLogger struct {}
func (l SqlLogger) Log(message string) {
	fmt.Println(message)
}
// ConsoleLogger
type ConsoleLogger struct {}
func (l ConsoleLogger) Log(message string) {
	fmt.Println(message)
}
// FileLogger
type FileLogger struct {}
func (l FileLogger) Log(message string) {
	fmt.Println(message)
}


// Interface can be used as a type for a variable
type Server struct {
	logger Logger
}
// Interface can also be used as a funtion parameter or return value
func process(logger Logger) {
	logger.Log("hello!")
}
```

## Features

### Error Handling

Go's preferred way to deal with errors is through return values, not exceptions.

We can create our own Error type by using the built-in error interface, which is:

```go
type error interface {
	Error() string
}
```

More commonly, we can create our own errors by importing the errors package and using it in the New function:

```go
package main

import (
	"errors"
)

func process(count int) error {
	if count < 1 {
		return errors.New("Invalid count")
	}
 ...
 return nil
}
```

Go does have `panic` and `recover` functions. `panic` is like throwing an exception while `recover` is like catch; they are rarely used.

### Defer

Some resources that we use require that we explicitly release them. For example, we need to Close() files after we're done with them. For example, as we're writing a function, it's easy to forget to Close something that we declared 10 lines up or a function might have multiple return points. Go's solution is the defer keyword. Whatever we *defer* will be executed after the enclosing function (in below example, it will be main()) returns, even if it does so violently. This lets us  release resources near where it’s initialized and takes care of multiple return points.

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	file, err := os.Open("a_file_to_read")
	if err != nil {
		fmt.Println(err)
		return
	}
	defer file.Close()
	// read the file
}
```

### go fmt

Most programs written in Go follow the same formatting rules, namely, a tab is used to indent and braces go
on the same line as their statement. We can apply the formatting rule to it and all sub-projects using 

```bash
go fmt ./..
```

### Initialized If

Go supports a slightly modified if-statement, one where a value can be initiated prior to the condition being
evaluated

```go
if err := process(); err != nil {
	return err
}
```

### Empty Interface and Conversions

Go has option to have an empty interface with no methods: `interface{}`

Since every type implements all 0 of the empty interface's methods, and since interfaces are implicitly implemented, every type fulfills the contract of the empty interface.

```go
// add funtion with empty interface
func add(a interface{}, b interface{}) interface{} {
 ...
}

// to convert an interface variable to an explicit type, use `.(TYPE)`
// if the underlying type is not int, the below will result in an error
return a.(int) + b.(int)
```

### Strings and Byte Array

Strings and byte arrays are closely related. We can easily convert one to the other:

```go
func main() {
	// string is array of bytes
	stra := "the spice must flow"
	fmt.Println(stra)	// prints: the spice must flow
	
	// create slice of bytes
	byts := []byte(stra)
	fmt.Println(byts) // prints: [116 104 101 32 115 112 105 99 101 32 109 117 115 116 32 102 108 111 119]
	
	// convert bytes to string
	strb := string(byts)
	fmt.Println(strb) // prints: the spice must flow
}
```

When we use `[]byte(X)` or `string(X)`, we're creating a copy of the data. This is necessary because strings are immutable.

Strings are made of *runes* which are unicode code points. If we take the length of a string, we might not get
what we expect.

```go
fmt.Println(len("椒"))	// prints: 3
```

If we iterate over a string using range, we'll get runes, not bytes. When we turn a string into a `[]byte` we'll get the correct data.

### Function Type

Functions are first-class types which can be used anywhere -- as a field type, as a parameter, as a return value.

```go
package main

import (
	"fmt"
)

type Add func(a int, b int) int

func main() {
	fmt.Println(process(func(a int, b int) int {
		return a + b
	}))
}

func process(adder Add) int {
	return adder(1, 2)
}
```

## Concurrency

Go is often described as a concurrent-friendly language. The reason for this is that it provides a simple syntax
over two powerful mechanisms: goroutines and channels

### Goroutines

A goroutine is similar to a thread, but it is scheduled by Go, not the OS. Code that runs in a goroutine can run
concurrently with other code.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("start")
	// we start a goroutine using the `go` keyword followed by the function we want to execute
	go process()
	// Sleep for a few milliseconds because the main process exits before the goroutine gets a chance to execute - To solve this, we need to coordinate the code.
	// the process doesn't wait until all goroutines are finished before exiting
	time.Sleep(time.Millisecond * 10) // this is bad, don't do this!
	fmt.Println("done")
}
func process() {
	fmt.Println("processing")
}

// If we just want to run a bit of code, such as the above, we can use an anonymous function.
go func() {
	fmt.Println("processing")
}()
```

### Synchronization

Below example uses go routine to print 1 to 20. But the problem is as multiple go routines work at the same time, the order of numbers will not be the same everytime

```go
package main

import (
	"fmt"
	"time"
)

var counter = 0

func main() {
	for i := 0; i < 20; i++ {
		go incr()
	}
	time.Sleep(time.Millisecond * 10)
}
func incr() {
	counter++
	fmt.Println(counter)
}
```

It's true that if we run the above code, we'll sometimes get that output. However, the reality is that the behavior is undefined. Why? Because we potentially have multiple (two in this case) goroutines writing to the same variable, counter, at the same time. Or, just as bad, one goroutine would be reading counter while another writes to it.

We can have as many readers as we want, but writes need to be synchronized. There are various ways to do this, the most common approach is to use a **mutex**

A mutex serializes access to the code under lock. The reason we simply define our lock as `lock sync.Mutex` is because the default value of a `sync.Mutex` is unlocked.


```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	counter = 0
	lock    sync.Mutex
)

func main() {
	for i := 0; i < 20; i++ {
		go incr()
	}
	time.Sleep(time.Millisecond * 10)
}
func incr() {
	lock.Lock()
	defer lock.Unlock()
	counter++
	fmt.Println(counter)
}
```

There's another common mutex called a read-write mutex. This exposes two locking functions: one to lock for reading and one to lock for writing. In Go, sync.RWMutex is such a lock. In addition to the Lock and Unlock methods of a sync.Mutex, it also exposes RLock and RUnlock methods; where R stands for Read.

Furthermore, part of concurrent programming isn't so much about serializing access across the narrowest possible piece of code; it's also about coordinating multiple goroutines. For example, sleeping for 10 milliseconds isn't a particularly elegant solution. What if a goroutine takes more than 10 milliseconds? What if it takes less and we're just wasting cycles? Also, what if instead of just waiting for goroutines to finish, we want to tell one hey, I have new data for you to process!? These are all things that are doable without channels. Certainly for simpler cases, I believe you should use primitives such as sync.Mutex and sync.RWMutex, but as we'll see in the next section, channels aim at making concurrent programming cleaner and less error-prone.

### Channels

The challenge with concurrent programming stems from sharing data. If the goroutines share no data, we needn't worry about synchronizing them. But many systems are built to share data acreoss multiple requests.

A channel is a communication pipe between goroutines which is used to pass data. In other words, a goroutine that has data can pass it to another goroutine via a channel. The result is that, at any point in time, only one goroutine has access to the data.

A channel, like everything else, has a type. This is the type of data that we'll be passing through our channel. 

For example, to create a channel which can be used to pass an integer around, we'd do:

```go
// The type of this channel is `chan int`.
c := make(chan int)

// To pass this channel to a function, the signature looks like below
func worker(c chan int) { ... }
```

Channels support two operations: receiving and sending. 

```go
// We send to a channel by doing:
CHANNEL <- DATA

// We receive from channel by doing:
VAR := <-CHANNEL
```

The arrow points in the direction that data flows. When sending, the data flows into the channel. When
receiving, the data flows out of the channel

> **NOTE**
> - Receiving and Sending to and from a channel is blocking
> - That is, when we receive from a channel, execution of the goroutine won't continue until data is available. Similarly, when we send to a channel, execution won't continue until the data is received.

Consider a system with incoming data that we want to handle in separate goroutines. This is a common
requirement. If we did our data-intensive processing on the goroutine which accepts the incoming data, we'd
risk timing out clients.

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// defind Worker struct
type Worker struct {
	id int
}

// worker waits until data is available then "processes" it
func (w *Worker) process(c chan int) {
	for {
		data := <-c
		fmt.Printf("worker %d got %d\n", w.id, data)
	}
}

func main() {
	// start the worker
	c := make(chan int)
	for i := 0; i < 5; i++ {
		worker := &Worker{id: i}
		go worker.process(c)
	}
	// write data to the worker
	for {
		c <- rand.Int()
		time.Sleep(time.Millisecond * 50)
	}
}
```

What happens if we receive more data than we can handle? In cases where we need high guarantees that the data is being processed, we probably will want to start blocking the client. In other cases, you might be willing to loosen those guarantees. There are a few popular strategies to do this.

The first is to buffer the data. If no worker is available, we want to temporarily store the data in some sort of queue. 

### Buffered Channel

*Channels have buffering capability built-in.* When we created our channel with make, we can give our channel a length.

```go
c := make(chan int, 100)
```

We can make this change, but we'll notice that the processing is still choppy. Buffered channels don't add
more capacity; they merely provide a queue for pending work and a good way to deal with a sudden spike.

Buffering can be checked by looking at the channel's `len(c)`

```go
c := make(chan int, 100)
c <- rand.Int()
fmt.Println(len(c))
```

Even with buffering, there comes a point where we need to start dropping messages. We can't use up an
infinite amount of memory hoping a worker frees up. For this, we use Go's `select`.

```go
c := make(chan int)
for {
 select {
 case c <- rand.Int():
 //optional code here
 default:
 //this can be left empty to silently drop the data
 fmt.Println("dropped")
 }
 time.Sleep(time.Millisecond * 50)
}
```

Another popular option is to `timeout`. We're willing to block for some time, but not forever. To block for a maximum amount of time, we can use the `time.After` function.

```go
for {
 select {
 case c <- rand.Int():
 case <-time.After(time.Millisecond * 100):
 fmt.Println("timed out")
 }
 time.Sleep(time.Millisecond * 50)
}
```

`time.After` returns a channel, so we can select from it.

here's what an implementation of after could look like:

```go
func after(d time.Duration) chan bool {
 c := make(chan bool)
 go func() {
 time.Sleep(d)
 c <- true
 }()
 return c
}
```

`time.After` is a channel of type `chan time.Time`. In the previous select example, we simply discard the value that was sent to the channel. If you want though, you can receive it

```go
case t := <-time.After(time.Millisecond * 100):
 fmt.Println("timed out at", t)
```

it's common to see a `select` inside a `for`

```go
for {
 select {
 case data := <-c:
 fmt.Printf("worker %d got %d\n", w.id, data)
 case <-time.After(time.Millisecond * 10):
 fmt.Println("Break time")
 time.Sleep(time.Second)
 }
}
```
