# Introduction to Go
Go was published in 2009 by Google engineers as an alternative to C++, but took 3 years till the first release - no connection with Bitcoin :)  There are several Go tutorials online, this one is tailored to our blockchain development course, focused on Hyperledger fabric. Corrections and additional requests are welcomed.

## Contents
* [Environment](#environment)
* [Language basics](#language-basics)
 * [Source file structure](#source-file-structure)
   * [Variables](#variables)
   * [Functions](#functions)
   * [Methods](#methods)
   * [Interfaces](#interfaces)
   * [Loops](#loops)
   * [Arrays](#arrays)
   * [Mappings](#mappings)
   * [Structures](#structures)
   * [Error handling](#error-handling)

Init, Invoke, Query
stub, shim

# Environment
`go run <program_name>` runs the program
`go build <package_name>`  compiles the package and provides binary executables
`go install <package name>` compiles the package and installs it in `/bin` folder

# Language basics
>>*, function string, args []string) ([]byte, error)
>> log.Errorf
>> json.Marshal, json.Unmarshal

## Source file structure
Programs in Go reside in packages. We must declare a package as a first statement in the program. Package `main` is the entry point to our program.  Next comes the declaration of external packages and possibly any global variables.
```
package main
import (
	"fmt"
)

/* global variable declaration */
var g_name string
```
> Note that structs starting with a capital letter are *exported* and can be used in other packages.

## Variables
Some basic data types are available (ints, floats, bools, , type inference is possible, as well as multiple declarations and initialization in-line. Besides variable declared with `var`, there are also constants declared with `const`, whose values must be known at the compilation.
Strings are defined with double quotes and are by default encoded in UTF-8. They are slices of bytes.
> Default value (or "nothing") is `nil`.

Example `hello.go`:
```
package main
import (
	"fmt"
)

func main() {
	var city string = "Cyberjaya" //declaring with the initial value
	//city := "Cyberjaya" //short-hand declaration is also possible

	fmt.Println("string: ", city)
	
	fmt.Println("first slice: ", city[0]) //prints 67, decimal value of the UTF-8 char
	fmt.Printf("char %c, hex value: %x\n", city[0], city[0])
}
```
would output:
```
> go run hello.go 
string:  Cyberjaya
first slice:  67
char C, hex value: 43
```
> Since a "character" is such an ambiguous thing, Go has **runes**, which stand for elements in the Unicode table (a rune is a code point). See the next example to see it in action.

> A **blank identifier** is a catchall for something that we want to ignore. It can be used for any value/type. See the next example to see it in action.

## Functions
Functions can have 0 or many input/output parameters. If output parameters are named, then we don't need to declare them in the function (*formal parameters*) and can also simply `return`.
```
package main
import (
	"fmt"
)

func getChar(instring string, idx int) (char string, result string) {
	runes := []rune(instring) //convert to an array of runes
	char = string(runes[idx]) //get the char at the corresponding index in runes
	result = "OK" //dummy output to showcase multiple outputs
	return
}

func main() {
	var testchar string
	testchar, _ = getChar("Cyber", 1) //the second output of the function is catched with _ and ignored
	fmt.Println("testchar:", testchar) //would output "testchar: y"
}
```

## Methods
Method is a function with the additional *receiver type*: `func (receiver <Type>) methodName(<parameters>) {  ... }`. Since Go doesn't have classes, we can use methods to manipulate types like structs etc. and we can also use two methods with the same name, who operate on different receiver types.
Go supports value and pointer receivers and converts automatically.
```
package main

import (
	"fmt"
	"strings"
)

type User struct {
	firstName, lastName string
}

func (u User) GetFullName() string {
	u.firstName = "Yolo"
	return u.firstName + " " + u.lastName
}

func (u *User) GetFormattedName() string {
	u.firstName = "Anji"
	return strings.ToUpper(u.lastName) + ", " + u.firstName
}

func main() {
	u := User{"Joe", "Doe"}
	fmt.Println(u.GetFullName())
	fmt.Println(u.firstName) //the type was copied, so no changes are kept

	fmt.Println(u.GetFormattedName())
	fmt.Println(u.firstName) //the type was referenced, so changes are made to the base
}

```
... would output:
```
Yolo Doe
Joe
DOE, Anji
Anji
```

## Interfaces
Interface is a collection of method signatures. It specifies the methods a particular type should have when implemented. The actual implementation is implicit, so we don't need to declare that we are implementing particular interfaces, we just need to match all methods' names and input/output parameters.
```
package main

import (
	"fmt"
	"time"
)

type User struct {
	userName string
	lastLogin time.Time
}

func (u *User) GetLastLogin() string {
	return fmt.Sprintf("%s, your last login was at: %s", u.userName, u.lastLogin.Format("2006-01-02 15:04:05"))
}

type Administrator struct {
	id       int
	lastLogin time.Time
}

func (a *Administrator) GetLastLogin() string {
	return fmt.Sprintf("%d, your last login was at: %s", a.id, a.lastLogin.Format("2006-01-02 15:04:05"))
}

type ILastLogin interface {
	GetLastLogin() string //this interface only has one method
}

func LastLoginMsg(i ILastLogin) string {
	return fmt.Sprintf("Dear %s", i.GetLastLogin()) //LastLoginMsg is generic and supports both User and Administractor types
}

func main() {
	u := &User{"Joe", time.Date(2018, 1, 29, 18, 42, 59, 0, time.UTC)}
	fmt.Println(LastLoginMsg(u))
	a := &Administrator{42, time.Date(2018, 12, 15, 10, 30, 0, 0, time.UTC)}
	fmt.Println(LastLoginMsg(a)) //we use the same interface with two different types
}
```
... would output:
```
Dear Joe, your last login was at: 2018-01-29 18:42:59
Dear 42, your last login was at: 2018-12-15 10:30:00
```

&NewLine;
In Hyperledger fabric, there are two important interfaces for us to use:
1. [`Chaincode`](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode) for received transactions with two necessary methods:
	* `Init(stub ChaincodeStubInterface) pb.Response`
	* `Invoke(stub ChaincodeStubInterface) pb.Response`
2. [`ChaincodeStubInterface`](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStubInterface) for reading/modifying the ledger with several methods that we could implement:
	* `GetArgs() [][]byte`
	* `GetStringArgs() []string`
	* `GetFunctionAndParameters() (string, []string)`
	* `GetArgsSlice() ([]byte, error)`
	* `GetTxID() string`
	* `GetChannelID() string`
	* `InvokeChaincode(chaincodeName string, args [][]byte, channel string) pb.Response`
	* `GetState(key string) ([]byte, error)`
	* `PutState(key string, value []byte) error`
	* `DelState(key string) error`
	* ...
## Loops
Loops are pretty standard, though `{}` are compulsory. The code can be made in various ways to provide for better readability. Example:
```
package main
import (
	"fmt"
)

func main() {
	for i:=0; i<3; i++ {
		fmt.Println(i)
	}
	
	//is the same as
	var j int = 0;
	for ;j<3; j++ {
		fmt.Println(j)
	}
}
```

## Arrays
Arrays are value types (changes to copies of arrays are not reflected back) and can be declared in several ways, short-hand declaration with incomplete specification is also possible. 
```
package main
import (
	"fmt"
)

func main() {
	var arr1 [5]int;
	arr2 := [...]int{1,2,3,4,5}; //declared short-hand, with the size omitted
	
	fmt.Println(arr1[0]) //outputs 0
	fmt.Println(arr2[0]) //outputs 1
	fmt.Println(len(arr2)) //outputs 5
}
```
Fancy **iterating with range**?
```
package main
import (
	"fmt"
)

func main() {
	arr := [...]int{10,20,30};
	
	for index,value := range arr {
		fmt.Printf("index: %d, value: %d\n", index, value)
	}
}
```

**Slices** are pointers to a subset of arrays, they don't hold value (mind this when passing slices via function calls). If we change a slice, the underlying array will be changed. A slice has two handy methods: length (number of elements the slice is pointing to) and capacity (number of elements in the whole array). 
```
package main
import (
	"fmt"
)

func main() {
	arr := [...]string{"ab","cd","ef","gh"};
	slice := arr[2:3]
	
	for index,value := range slice {
		fmt.Printf("index: %d, value: %s\n", index, value)
	} //outputs "index: 0, value: ef"
	
	slice[0] = "xy"
	fmt.Println("Modified slice: ", slice) //outputs "Modified slice:  [xy]"
	fmt.Println("Slice length: ", len(slice), ", capacity: ", cap(slice)) //outputs "Slice length:  1 , capacity:  2"
}
```
Slices have 3 methods: `make` for creating a new slice, `append` to increase the capacity, `copy` for copying one slice to another.
```
package main
import (
	"fmt"
)

func main() {
	smallslice := make([]int, 1,1) //that is: make([]Type,Len,Cap)
	fmt.Println(smallslice[0])     //output "0"
	
	smallslice = append(smallslice, 1,2,3)
	fmt.Println((smallslice))      //output "[0 1 2 3]"
	
	largeslice := make([]int,5,5)
	copy(largeslice, smallslice)  //output "[0 1 2 3 0]"
	
	fmt.Println((largeslice))
}
```

## Mappings
Mappings map values to corresponding keys. A map can be created by `make(map[key_type]value_type)`. We can check the existence of a key by assigning the key access to two variables, the second one is the existence flag.
```
package main
import (
	"fmt"
)

func main() {
	cities := make(map[int]string) //postcode:city
	cities[62050] = "Putrajaya"
	cities[63000] = "Cyberjaya"	
	fmt.Println((cities))
	
	queriedPost, exists := cities[62000]	
	if exists {
		fmt.Println((queriedPost))
	} else {
		fmt.Println("Doesn't exist.")
	}
}
```
## Structures
To create a more complex data structure, we can define a `struct`. Arrays of structs are often handy.
```
package main
import (
	"fmt"
)

type User struct {  
	id int
	name, email string
}

func main() {  
	usr := User {
		id: 0,
		name: "Arik",
		email: "arik@email.me",
	}
	fmt.Println("Email of the first user is: ", usr.email)
	
	users := [2]User{}
	users[0] = usr
	users[1] = User {
		id: 1,
		name: "Tarik",
		email: "tarik@mail.you",
	}
	fmt.Println("Name of the second user is: ", users[1].name)
}
```


## Error handling
error, panic, defer
Error type is a simple string struct.
```
package main
import (
	"fmt"
	"errors"
)

func getChar(instring string, idx int) (char string, result error) {
	if (idx == 666) {
		return "", errors.New("Forbidden number.")
	}
	runes := []rune(instring)
	char = string(runes[idx])
	result = nil
	return
}

func main() {	
	testchar, err := getChar("Cyber", 666)
	if (err == nil) {
		fmt.Println("testchar:", testchar)
	} else {
		fmt.Println(err)		
	}
}
```

Or we can use the formatted `Errorf` of the `fmt` package:
```
package main
import (
	"fmt"
)

func getChar(instring string, idx int) (char string, result error) {
	if (idx == 666) {
		return "", fmt.Errorf("Forbidden number: %d", idx) //formatted error	
	}
	runes := []rune(instring)
	char = string(runes[idx])
	result = nil
	return
}

func main() {	
	testchar, err := getChar("Cyber", 666)
	if (err == nil) {
		fmt.Println("testchar:", testchar)
	} else {
		fmt.Println(err)	
	}
}
```
&NewLine;
If errors don't pull us out of the tar pit, we can `panic` to stop the program (that is, after executing any possibly remaining `defer`red function calls, think housekeeping). A `recover` inside a `defer` function can stop the `panic` :)

Example of panic:
```
package main
import (
	"fmt"
)

func main() {	
	panic("Mayday!")
}
```
... would output:
```
panic: Mayday!

goroutine 1 [running]:
main.main()
	/tmp/sandbox112133275/main.go:9 +0xa0  Program exited: status 2.
```

&NewLine;
Example of a deferred call, recovering from panic:
```
package main
import (
	"fmt"
)

func clean() {
	fmt.Println("Cleaning the mess")
        if r := recover(); r != nil {
            fmt.Println("Recovering from: ", r)
        }	
}

func makemess() {
	fmt.Println("Making a mess")
	panic("Mayday in making a mess!")
}

func main() {	
	defer clean()
	
	fmt.Println("Start in main ...")
	makemess()
	fmt.Println("Final housekeeping ...")
}
```
... would output:
```
Start in main ...
Making a mess
Cleaning the mess
Recovering from:  Mayday in making a mess!
```
When would we use such things? Think of closing files, connections etc.
