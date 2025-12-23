---
title: Go Starter
description: A starter guide on using Go for whatever fancies your heart
date: 2023-09-07T22:20:32Z
image: https://www.gravatar.com/avatar/24a44e8bc08cd0c6e2fd43378eeb25b2?s=192&d=identicon&r=PG&f=y&so-version=2
math: false
tags:
  - Tutorial
  - Guide
  - go
categories:
  - Tutorial
toc: true
license: 
hidden: false
comments: false
draft: false
links:
  - title: Installing go using snap/apt or binary
    description: Installing the go compiler using snap or apt for debian
    website: https://www.cyberciti.biz/faq/how-to-install-gol-ang-on-ubuntu-linux/
    images: https://www.cyberciti.biz/media/new/category/go-lang-logo.png
  - title: Go Language Tour
    description: A tour of the Go programming language
    website: https://go.dev/tour
    image: https://go.dev/images/go-logo-white.svg 
  - title: Go Tutorials
    description: Officially made Go tutorials
    website: https://go.dev/doc/tutorial
    image: https://go.dev/images/go-logo-white.svg
---

## Go Starter

Go is a programming language designed for the open source community, it is
supported by Google and has a simlar way of operating to C programming, it
depends on its own environment variable system for most to all of its operations
Go has a garbage collector which handles most memory errors effectively, it
doesn't have null based errors but rather handles them in a way which would be
discussed in [Error Handling](#Error Handling) Now before we get started on the
syntax of Go and how it differs from other programming languages lets look at
how Go operates with its files

## Installing and using Go

- The guides of downloading and installing the go compiler and its standard
  library and tools can be found [here](https://go.dev/dl/)
- For linux systems, you can use your distributions package manager to install
  go, for example for debian and Arch distros

### Debian/Ubuntu

```bash
sudo apt update 
sudo apt install golang-go
```

### Arch Linux

```bash
sudo pacman -Sy go
```

## Go env, package and modules

### go env

Go has a lot of go based environment variables to look through all of them,
these only work within a go project and cannot be access in a bash environment
directly, you can use the `go env` command to view all go environment variables
Currently the environment variable within go are

```shell
$ go env
	GO111MODULE=''
	GOARCH='amd64'
	GOBIN=''
	GOCACHE='/home/user/.cache/go-build'
	GOENV='/home/user/.config/go/env'
	GOEXE=''
	GOEXPERIMENT=''
	GOFLAGS=''
	GOHOSTARCH='amd64'
	GOHOSTOS='linux'
	GOINSECURE=''
	GOMODCACHE='/home/user/go/pkg/mod'
	GONOPROXY=''
	GONOSUMDB=''
	GOOS='linux'
	GOPATH='/home/user/go'
	GOPRIVATE=''
	GOPROXY='https://proxy.golang.org,direct'
	GOROOT='/usr/lib/go'
	GOSUMDB='sum.golang.org'
	GOTMPDIR=''
	GOTOOLCHAIN='auto'
	GOTOOLDIR='/usr/lib/go/pkg/tool/linux_amd64'
	GOVCS=''
	GOVERSION='go1.21.1'
	GCCGO='gccgo'
	GOAMD64='v1'
	AR='ar'
	CC='gcc'
	CXX='g++'
	CGO_ENABLED='1'
	GOMOD='/dev/null'
	GOWORK=''
	CGO_CFLAGS='-O2 -g'
	CGO_CPPFLAGS=''
	CGO_CXXFLAGS='-O2 -g'
	CGO_FFLAGS='-O2 -g'
	CGO_LDFLAGS='-O2 -g'
	PKG_CONFIG='pkg-config'
	GOGCCFLAGS='-fPIC -m64 -pthread -Wl,--no-gc-sections -fmessage-length=0 -ffile-prefix-map=/tmp/go-build3927296176=/tmp/go-build -gno-record-gcc-switches'
```

- You can modify these using `go env -w <VARIABLE>` for example say, I need to
  change the C compiler used from gcc to clang, I can do

```shell
go env -w CC='clang'
```

- When checking the go environments

```shell
$ go env | grep 'CC='
CC='clang'
```

- And reverse it back with `go env -u CC`

- You can also view it as json using `go env -json`

### modules and workspaces

- Go modules are like project files, a module is the root of all packages when
  you look at the GOMOD variable, you could see it contains the value
  `/dev/null` this automatically changes when there is a go.mod file in the
  directory used
- For example a directory named gostart has a `go.mod` file when running
  `go env | grep 'GOMOD='` we get

```shell
$ go env | grep 'GOMOD='
GOMOD='/home/user/Documents/gostart/go.mod'
```

- Go modules represent a local project module that can be accessed when called
- To create a new go module, create a directory and name with your preferred
  name and change to that directory
- Run `go mod init <module name>` for example `go mod init hello_go`, the module
  name can be different from the directory name but it is most preferred for
  some cases
- The file created would contain the following

```go
// hello_go/go.mod
module hello_go
go 1.21.1
```

- Go module names are kinda unique you can even give the name of a repository
  link as a module name for example

```shell
$ go mod init github.com/Android-Jester/gostart
```

- ? So what if you want to import modules from other projects how can you do
  that
- Well the best way would be to `go get <module link>`, for example, I want a
  special way to log my files, a module I can use is
  `github.com/sirupsen/logrus`
- so after running `go get github.com/sirupsen/logrus`, the go.mod file becomes

```go
// hello_go/go.mod
module hello_go
  
go 1.21.1

require (
	github.com/sirupsen/logrus v1.9.3 // indirect
	golang.org/x/sys v0.0.0-20220715151400-c0bba94af5f8 // indirect
)
```

### Packages

- A package unlike a module is a subfolder and its files
- For every go program, the first file which contains the main function must
  have `package main` at the first line
- To have another package create another directory inside the project location
- all go files in that directory must include `package <directory name>` and can
  be called as part of the module
- For a project containing the following

```
.
├── common
│   └── random_func.go
├── go.mod
└── hello.go
```

```go
// hello.go
package main // main package must have main function
import "hello_go/common" // import code from the common folder

func main() {
	common.PrintHelloWorld()
}
```

Inside `common/helloWorld.go`

```go
package common // name of the directory
import "fmt"
func PrintHelloWorld() {
	fmt.Println("Hello World")
}
```

- Here demonstrates a simple function named `PrintHelloWorld`, don't worry too
  much about the code as it would be explained in detail later but here you
  could see that all go files in the directory common is taken and shown

## Go coding

Well that is enough of the modules, let's try to understand some go syntax

### Variables, constants and Functions
#### Data Types
- The current data types of Go are
	- `bool` 
	- `string` 
	- Signed Integers: `int` `int8`  `int16`  `int32`  `int64`
	- Unsigned Integers: `uint` `uint8` `uint16` `uint32` `uint64` `uintptr`
	- `byte` (alias for `uint8`)
	- `rune` (alias for `int32`, represents a Unicode code point)
	- float: `float32` `float64`
	- complex: `complex64` `complex128` (For complex numbers)
#### declaration

##### Variable
- Variables are declared in two ways
```go
var helloString string
helloString = "Hello"
// OR
var helloString string = "Hello"
// OR
var helloString = "Hello" // Type can be omitted
// OR
helloString := "Hello"
```
- The first way is using the `var` keyword and optionally adding the type after the name of the variable then initializing it with a value using `=`, you can choose to declare before initializing or initialize the variable while declaring it
- The second way doesn't require the `var` keyword and uses this weird `:=` to directly initialize it to the value
- you can actually declare multiple variables or similar type
```go
var i,j int
i,j = 1, 2
// OR
var i, j int = 1, 2
// OR
var i, j = 1, 2
// OR
var (
	i = 1
	j = 2
)
// OR
var (
	i int = 1
	j int = 2
)
// OR
i, j := 1, 2
```
##### Functions
- Functions are declared with the `func` keyword, and carry arguments, the arguments can be declared with the same time and a function can return more than one value
```go
import "math"
func hypot(x float64, y float64) float64 {
	return math.Sqrt(x*x + y*y)
}


func hypot(x, y float64) float64 {
	return math.Sqrt(x*x + y*y)
}

func split(sum int) (int, int) {
	
	x := sum * 4 / 9
	y := sum - x
	return x, y
}
```
- To make the split function much simpler, you can even parse the variables directly in the declaration and learn the return expression with just `return`
```go
func split(sum int) (x,y int) {
	x := sum * 4 / 9
	y := sum - x
	return
}
```
##### Access Level
- Variables and functions in Go are private when their identifiers are in camel case and limited only to that package meaning no matter how many go files you create, those functions and variables cannot escape
- For example 
```
.
├── common
│   ├── priv.go
│   └── random_func.go
├── go.mod
└── hello.go
```

```go
// hello.go
package main
import "hello_go/common"
func main() {
	common.PrintHelloWorld()
	fmt.Println(GuessWhatNumber)
	common.helloGame() // this would panic
}
```

```go
// common/priv.go
package common
import "fmt"

var GuessWhatNumber = 1.21

func helloGame() {
	fmt.Println("HH")
}
```

```go
// common/random_func.go
package common
import "fmt"

func PrintHelloWorld() {
	helloGame() // but this would work
	fmt.Println("Hello World")
}
```
### defer
The `defer` keyword halts and stacks function calls that you want to be called last and runs that function at the end
These deferred function calls can be stacked in a First In First Out way
```go
func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}

```

### Loops and decisions
#### For
Go contains only one keyword for loops and that is `for`, this compared to other languages does the functionality of both `for` and `while` and this is demonstrated below
```go
sum := 0
for i := 0; i < 10; i++ {
	sum += i
}

// While instance
sum := 1
	for sum < 1000 {
		sum += sum
	}


// pre-post optional
sum := 0 
for ;sum < 10; {
	sum += 1
}

// Forever loop
sum := 0
for {
	sum += 1
}


```
With [arrays and slices](#Array and Slices)(described after this section), for loops can use the `range` expression to loop through values in the array or slice
```go
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}
```


#### Decisions
##### If/else
- `If/else` statements are basically the same for other languages and used to handle decisions in code
```go
package main

import "fmt"

var x int8 = 10
if x > 10 {
	fmt.Println(x)
} else if x == 10 {
	fmt.Println("x is ", x)
} else {
	fmt.Println("X is less")
}
```

If statements can be shorter by a line by putting the initialization of a  variable inside the conditional section
```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}
```

##### Switches
Switches can be used to replace the conditions of `if` statements and can be used relatively differently from the usual switches in other languages

Switches in Go, parse data top to bottom meaning that it doesn't matter what the conditional is, either the result being checked for the value to compare against, it will always read and compare the value from the top to the bottom and compare it to whatever is provided
You can also skip the conditional if you want
```go
package main

import (
	"fmt"
	"time"
)


// Top down parsing
func main() {
	fmt.Println("When's Monday?")
	today := time.Now().Weekday()
	
	fmt.Println(time.Now())
	switch time.Tuesday {
	case today:
		fmt.Println("Today.")
	case today + 1:
		fmt.Println("Tomorrow.")
	case today + 2:
		fmt.Println("In two days.")
	default:
		fmt.Println("Too far away.")
	}
}

// Normal Switch
func main() {
	fmt.Println("What day is today?")
	
	
	fmt.Println(time.Now())
	switch today := time.Now().Weekday() {
	case time.Monday:
		fmt.Println("Monday.")
	case time.Tuesday:
		fmt.Println("Tuesday.")
	case time.Wednesday:
		fmt.Println("Wednesday.")
	default:
		fmt.Println("Towards the end")
	}
}


// No conditional
func main() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}

```

### Array and Slices
#### Array
- Arrays in Go are a special datatype that holds a **limited** amount of data limited to a specific datatype
- They are declared using the `[n]T` where n is the length of the array and T is the type, you can access the members of an array by specifying the array identifier and their index, you can even modify each member of the array
- Each member corresponds to a specific memory address within the array, thus you can say Go arrays are like pointers but instead of pointing to a specific memory space, it is pointing a limited number of memory spaces
```go
 func main() {
	var a [2]string
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	fmt.Println(primes)
}
```
#### Slice
- Slices on their own are almost the opposite of an array, they hold no strict length on their own but behave similar to an array
- They are declared similar to an array but the length isn't included
```go
func slice_trouble() {
	var a_slice []int
	describe(a_slice)
	a_slice = []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
	describe(a_slice)
	a_slice[10] = 100 // panics
	describeMember(a_slice[10])

}
```
- However, the dynamic nature of slices aren't always the case especially with the amount of memory when a slice is formed from an array or another slice, in this situation, the slice is rather a pointer pointing to the array memory space and not to a memory space holding a value
 ```go
func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names) // prints: [John Paul George Ringo]
	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b) // prints: [John Paul] [Paul George]
	b[0] = "XXX"
	fmt.Println(a, b) //prints: [John XXX] [XXX George]
	fmt.Println(names) // prints: [John XXX George Ringo]
}
```
- Slices have both a _length_ and a _capacity_.
	- The length is the number of elements it contains.
	- The capacity is the number of elements in the underlying array, counting from the first element in the slice.
	
- The length and capacity of a slice `s` can be obtained using the expressions `len(s)` and `cap(s)`.
	- Also slices can be created with the `make()` expression
		- This also allows one to preset the length and capacity
```go
b := make([]int, 0, 5) //len(b)=0 cap(b)=5
```
However this can change if you want to increase the length of the slice, this can be done using
using the the `apppend()` function
```go
var s []int // arrays have fixed sizes, this is a slice
s = append(s, 2, 3, 4)
```

Slices can be defined in many ways depending on the amount of data that you want to carry
```go
//for the array 
var a [10]int
// you can have
a[0:10] // This takes between 0 and 10
a[:10] // this takes from 9 backwards
a[0:] // this takes from the index 0 to the end
a[:] // this just takes all
```

### Structs and Interfaces and Maps
#### Structs
Structs are like a type system that are used to contain a set of data and also can contain their own functions, in Go, they are defined using the `type` keyword
```go
type MyStruct struct {
	structValue string
	anyVariable string
}
```
However you can initialize a struct in many ways as well, for example
```go
type Vertex struct {
	X,Y int
}

var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)
```
Here, we can see that the vertex struct can be initialized with both or one of the values, without any value or as a pointer

Functions that require a struct are called methods and they are defined using pointers to a struct and have a special syntax for that
```go
package main

import "fmt"

type MyStruct struct {
	structValue string
	anyVariable string
}

// Methods
func (st MyStruct) NewStruct() MyStruct {
	st.structValue = "[+] Structure Value"
	st.anyVariable = "[+] ANy"
	return st
}

// Same but with pointers
func (st *MyStruct) Initial() {
	st.structValue = "Structure Value"
	st.anyVariable = "ANy"
}

func main() {
	var newStruct MyStruct = MyStruct{"Hello", "Hi"}
	fmt.Println(newStruct) // prints: {Hello Hi}
	newStruct.Initial()	
	fmt.Println(newStruct) // prints: {Structure Value ANy}
	newStruct.NewStruct()
	fmt.Println(newStruct)	// prints: {Structure Value ANy}
}


```
In this example, methods are created from the struct `MyStruct`, but if you look closely, they don't behave the same, the `initial()` method behaves like a method you would find in Java or C# but the `NewStruct()` method doesn't, it doesn't have a pointer reference and thus only takes the value of the struct but doesn't update the value but rather just takes data from that struct as if another struct has been created
#### Interfaces
Interfaces are basically a set of method signatures that are defines the structs that implement them, they work similarly to implementation of interfaces in some languages like C# or JAVA but differ since they act as place holders for struct values that implement their methods
The value of an interface type can hold any value that implements those methods
```go
package main

import (
	"fmt"
	"math"
)

type Abser interface {
	Abs() float64
}

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}


func main() {
	var a Abser
	v := Vertex{3, 4}
	a = &v // a *Vertex implements Abser
	// a = v  panics because, v is a Vertex (not *Vertex) and does NOT implement Abser.

	fmt.Println(a.Abs())
}






```

##### Empty Interface
Interface can be made without any value and they can hold any value of any type it is instantiated with
```go
func main() {
	var i interface{}
	describe(i)

	i = 42
	describe(i)

	i = "hello"
	describe(i)
}

func describe(i interface{}) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```
##### Type assertion
A _type assertion_ provides access to an interface value's underlying concrete value, it is done using `t = i.(<Type>)` or `t, ok := i.(<Type>)`
- This statement asserts that the interface value `i` holds the concrete type `T` and assigns the underlying `T` value to the variable `t`.
- When using the expression `t, ok := i.(T)`, `ok` is a `bool`, that checks if the type the interface has is correct
```go
func main() {
	var i interface{}
	i = "Hello"
	t := i.(string)
	fmt.Println(t) // this prints Hello
	t2, ok := i.(float64)
	if ok {
		fmt.Print(t2)
	}
}
```
- Switches can be used to check if the interface is of the type provided by the cases
```go
func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	do(21)
	do("hello")
	do(true)
}
	```


### Maps
- Maps are like hash-maps, they contain a key and value pair, they are defined using the expression `map[T1]T2`, where T1 is the type that acts as a key and T2 represents the value, you can create a map using the `make()` expression similar to how slices are created
- Maps like slices don't have a defined length thus you can continuously make key-value pairs in a already instantciated map
```go
type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}
	m["Bell Labs2"] = Vertex{
		40.68433, -74.39,
	}

	fmt.Println(m["BellLabs2"])
}

	```
- You can delete a map using the `delete()` expression
```go
delete(m, "Bell Labs")
```

### Null references
When a variable isn't instantiated it contains a null value
Null values are represented by the zeros or nil of their types
- `0` for numeric types,
- `false` for the boolean type, and
- `""` (the empty string) for strings.
- nil is returned from others such as structs and interfaces
### Error Interfaces
- This is a built interface that is used to handle errors
- Error methods can be used to handle situations were an error would exist
- Errors can be using a simple if expression
```go
type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s",
		e.When, e.What)
}

func run() error {
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err)
	}
}
```

I hope this little Go language tour helps a lot for those who are either starting to code or who just want to learn a little about the language, although there is a much more detailed one at the [Go Language Tour](https://go.dev/tour) if there is any problem, you can contact me [here](mailto:duah14@outlook.com) and I would make the necessary changes and fixes to this blog
With that said, thank you for reading
