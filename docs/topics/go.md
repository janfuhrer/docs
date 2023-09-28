tags: #go

# go Basics

links: [[200 Programming Languages MOC|Programming Languages]] - [[000 Index|Index]]

---
## variables
- Go is a statically typed language. This means that each variable gets a type on declaration which can’t be changed later
- If you declare a variable and do not assign a value it is initialized with the zero value of its type (`0`, `false` or `""`)
```go
func main() {
	// declare variable
	name := "foo"
	
	// assign value after declaring
	name = "bar"

	// single variable
	var name string

	// multiple variables
	var (
		size int
		isFile bool
	)
}
```

## flow control
```go
func main() {
	// if, else
	x := 10
	if x>= 5 && x<0 {
		// if
	} else {
		// else
	}

	// switch
	dayOfWeek := 2
	switch dayOfWeek {
	case 1:
		// case 1
	case 2:
		// case 2
	default:
		// default
	}

	// loops
	for i := 0; i < 5; i++ {
		// for i
	}

	for i < 5 {
		// for i
		i++
	}

	for {
		// "while true"
		// use "break" or "next" to control flow
	}
}
```

## functions
```go
// without return value
func sayHello(name string) {
	fmt.Println("Hello", name)
}

// with return value
func add(a int, b int) int {
	return a + b
}

// with multiple return values
func sub(a int, b int) (int, bool) {
	result := a - b
	isNegative := result < 0
	return result, isNegative
}

func main() {
	sayHello(name)
	result := add(2, 3)
	result, negative := sub(2,3)
}
```

- In Go it is an error to declare variables without using them. If we only need a single value we can discard variables by assigning them to `_`
```go
func add(a int, b int) (int, bool) { ... }

result, _ := add(2, 3)
```

- In Go functions are first-class values. This means that you can pass functions around like ordinary values.
```go
func run(a int, b int, myfunc func(int, int) int) int {
	return myfunc(a, b)
}

func main() {
	add := func(a int, b int) int { return a + b }
	result := run(2, 3, add)
}
```

## error handling
- Go does not have exceptions, so if a function can fail, it returns an error value
```go
// function signature of "ReadFile"
func ReadFile(name string) ([]byte, error)


content, err := os.ReadFile("test.txt")
if err != nil {
	// handle error
}
```

## pointers
- In addition to the basic data types Go also have pointers. A pointer contains a memory address of an actual value or `nil` if they do not point to anything. The zero value of a pointer is `nil`.
- If you dereference a pointer you should always be sure that it is not `nil`. Otherwise your program crashes with an error `invalid memory address or nil pointer dereference` (same as NullPointerException in Java).

```go
// value to pointer (int to *int)
// -> with "&" you can create a pointer to a value
myPointer = &myInt

// value from pointer (*int to int)
// -> with "*" you can obtain the actual value from a pointer (dereference)
myInt = *myPointer
```

## structs
In Go function arguments are passed by value. This means they are copied in memory and a new variable is created. For large variables like structs this can be a performance problem. To overcome this we can use pointers. With pointers only the memory address to the variable value is passed and the variable itself is not copied. This is called passing values by reference.

- When createing a new instance you don’t have to specify all struct fields. If a field is not set it is initialized to the zero value of its type.
```go
// declare struct type
type User struct {
	Name         string
	FailedLogins int
	Locked       bool
}

// create instance
myUser := User{
	Name:         "admin",
	FailedLogins: 0,
	Locked:       false,
}

// we don't need to specify the struct field names
myUser2 := User{
	"admin",
	0,
	false,
}

// access struct fields
fmt.Printf("hello %s", myUser.Name)
myUser.FailedLogins += 1
myUser.Locked = true
```

### structs as function arguments
```go
func Reset(user *User) {
	user.FailedLogins = 0
	user.Locked = false
}

func main() {
	// call function with pointer of struct
	Reset(&myUser)

	// directly obtain a pointer to a User object
	myUser := &User{
		Name: "admin",
	}
	Reset(myUser)
}
```

### constructors
- Go does not have constructors. It is common to use a function `NewXxx` where `Xxx` is the struct name.
```go
type person struct {
	Name string
	Age  int
}

func NewPerson(name string, age int) person {
	return person{
		Name: name,
		Age:  age,
	}
}

func main() {
	pers := NewPerson("bob", 45)
	fmt.Println(pers)
}
```

### methods
- Instead of passing structs to functions we can attach methods to structs
```go
myUser.Reset()

// instead of
Reset(myUser)
```

- A method declaration looks like a function declaration. The only difference is that you specify to which struct you want to attach the method between the `func` keyword and the argument list
```go
func (user *User) Reset() {
	user.FailedLogins = 0
	user.Locked = false
}
```

### nested structs
- In comparision with other languages Go does not support inheritance. However something similiar can be achieved by using composition
```go
type person struct {
	firstName string
	lastName  string
	age       int
}

type blogPost struct {
	title   string
	content string
	author  person
}

func (p person) fullName() {
	fmt.Printf("%v %v\n", p.firstName, p.lastName)
}

func main() {
	bob := person{
		"Bob",
		"Müller",
		28,
	}
	blogPost1 := blogPost{
		"Inheritance in Go",
		"Go supports composition instead of inheritance",
		bob,
	}
	blogPost1.author.fullName()
}
```

---
links: [[200 Programming Languages MOC|Programming Languages]] - [[000 Index|Index]]