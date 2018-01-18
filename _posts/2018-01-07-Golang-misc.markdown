---
layout:     post
title:      "Golang misc part1"
subtitle:   "Composition and receiver"
date:       2018-01-07 12:32:50
author:     "Jim Ma"
header-img: "img/bg1.jpg"
---
If you are java developer and start to look at Golang, you may curious to know
what does the inheritance in Golang look like ? Is the Golang interface type
is something like jave's interface type ? How can I define methods/functions to
type struct in Golang(equivalent thing java object)? In this post, I am trying to
explain this with some example code and result.
## Inheritance
Actually Golang doesn't have the same object oriented inheritance like java's
extends keyword. But this important object oriented feature can be archived with
composition. Let's take a simple Person, Employee example. In object oriented
wolrd, the thing comes in your mind that employee should extend person. To implement
this with Go, Employee struct type will be composed with a Person type like the
following code snippet demonstrates:
``` go
type Person struct {
	Name   string
	Gender string
	Id     int32
}

type Employee struct {
	Person
	Company    string
	EmployeeID int32
}

```
The some different for Person field here , there is no type defined. This is called emdedded or embedding :
> A field declared with a type but no explicit field name is an anonymous field, also called an embedded field or an embedding of the type in the struct. An embedded type must be specified as a type name T or as a pointer to a non-interface type name *T, and T itself may not be a pointer type. The unqualified type name acts as the field name.

Is Person type explicitly defined here OK ? The answer is NO and Golang doesn't need you type more thing. If you insist to define the type for Person, you will finally you can't invoke the function defined for Person type and lost inheritance.

## Receiver
The receiver is following with go keyword **func** struct type you want to add the function to:  
``` go
func (e *Employee) GetEmployeeID() int32 {
	return e.EmployeeID
}

func (e *Employee) PrintID() {
	fmt.Printf("Employee's ID : %d", e.EmployeeID)
}
```
Receiver can be a pointer or a value receiver. To find out what's the difference between these two receive type, go to look at [this page](https://nathanleclaire.com/blog/2014/08/09/dont-get-bitten-by-pointer-vs-non-pointer-method-receivers-in-golang/). Pointer reciver allows function to modify the object which it points to and in function invocation it doesn't need to copy this object:
``` go
func (p *Person) ChangeName() string {
	p.Name = "Changed"
	return p.Name
}
func (p Person) NotReallyChangeName() string {
	p.Name = "Changed"
	return p.Name
}

func (p Person) PrintID() {
	fmt.Printf("Person's ID : %d \n", p.Id)
}

func (e *Employee) PrintID() {
	fmt.Printf("Employee's ID : %d", e.EmployeeID)
}
```
Given the second receiver will be copied in invocation, so if Person is a large object,
don't do this.
So from the above code , you'll find receive is the answer to define function/methods for
a type(object).

## Interface type
If you want to make a java object implement interface, you have to use java key word **implements**. In Go, interface is defined outside of function or object :
```go
type PrintIDPerson interface {
	PrintID()
}

func PrintPersonID(pidPerson PrintIDPerson) {
	pidPerson.PrintID()
}
```
This several lines code tells all object has PrintID() function is a PrintIDPerson, and PrintPersonID function can accept a PrintIDPerson object to call the interface method PrintID. So all Employee objects are PrintIDPerson type, and Person object too. Well it depends. For example:
``` go
1 person := object.Person{"John", "male", 100010}
2 employee := object.Employee{person, "MS", 10001}
3 PrintPersonID(&person)
4 PrintPersonID(person)
5 //PrintPersonID(employee) compiation error
6 PrintPersonID(employee)
```
The line 6 can't be compiled, because it isn't a correct PrintIDPerson. Why? Go back to look
at Employee's PrintID() function, the receiver is a pointer. **employee** here is a value object. But it's strange thing is line 3 and line 4 both work. This is all because the PrintID()'s receive is a value object Person.
How can we know if passed object is other type of interface ? Go's reflection is the good thing to help figure this out :
```go
func PrintPersonIDToConsole(in interface{}) {
	pidPerson, ok := in.(object.PrintIDPerson)
	if ok {
		pidPerson.PrintID()
	} else {
		fmt.Printf("Input : %v isn't a PrintIDPerson\n", in)
	}
}
```
Type assertion is the easy thing to tell you: **in.(object.PrintIDPerson)**  if **in** is a type of object.PrintIDPerson.
```go
person := object.Person{"John", "male", 100010}
employee := object.Employee{person, "MS", 10001}
PrintPersonIDToConsole(person)
PrintPersonIDToConsole(&person)
PrintPersonIDToConsole(employee)
PrintPersonIDToConsole(&employee)
```
Run pass the interface to the function, the result will tell you employee is not a object.PrintIDPerson type. It is simply because the passed receive is not a pointer, but Employee's PrintID() expects a pointer receiver:
```
Person's ID : 100010
Person's ID : 100010
Input : {{Changed male 100010} MS 10001} isn't a PrintIDPerson
Employee's ID : 10001
```

## Summary
* Go has composition and it can compose as many structs as you want. It's powerful. The only thing you need to take care is do this with embedded field and **DO NOT** add a type.
* Interface is defined by function.
* Function's receiver can be value or pointer. If it's a large struct, use pointer receiver.
  Or if you don't know too much, pointer receiver could be good(?).
* Interface is a type. It will check the passed type, if passed object is a pointer and the function's receiver is a pointer. Then this passed object is a type of this interface. If the function's receiver is value object, the passed pointer object or value object can both be casted to an interface type. 
