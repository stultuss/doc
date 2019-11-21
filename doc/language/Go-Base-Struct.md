Golang 基础-结构体
=========================
> 学习笔记

划重点：定义的 Struct 的 Field，首字母必须大写，否则类似 json.Marshal 的处理方法会忽略不处理。另外的，类似API，HTTP之类的英文缩写，必须要全部大写，而非首字母大写。参考：[CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)

## 介绍

创建一个结构体：

```go
type Person struct {
	Id   int64  `json:"id"` // 后面的是Struct Tag，表示序列化后的字段名为 id，序列化 => json.Marshal
	Name string `json:"name"`
}
```

初始化一个结构体：

```go
// new 关键字创建
p1 := new (Person)
p1.Id = 1
p1.Name = "p1"

// 指针创建
p2 := &Person{1, "p2"}
```

通常情况下，为每个结构体定义一个构建函数，并推荐使用构建函数初始化结构体，所以

```go
func NewPerson(id int64, name string) *Person {
	return &Person{id, name}
}

p3 := NewPerson(1, "p3")
```

## 方法

可以通过以下方式为结构体添加一个方法

eg: **person.go**

```go
package person

type Person struct {
	Id   int64  `json:"id"`
	Name string `json:"name"`
  status bool		// 首字母小写不能导出，作用与 private 相同，无法直接通过 p.status 获得
}

func NewPerson(id int64, name string) *Person {
	return &Person{id, name, true}
}

func (p *Person) SetStatus(status bool) {
	p.status = status
}

func (p Person) GetStatus() bool {
	return p.status
}
```

如果想要方法改变接收者的数据，就在接收者的指针类型上定义该方法 `func (p *Person) SetStatus(){ ... }`。否则，就在普通的值类型上定义方法 `func (p Person) GetStatus() { ... }`。

## 面向对象

Go 语言没有明确提出面向对象概念，但是基于现有语法设计，还是可以写出面向对象的代码。主要是通过借助结构体实现的。

### 接口

Go 语言中没有 `class` 关键字来表示类，但却有 `interface` 来表示接口。Go 语言中的接口非常灵活，通过它可以实现很多面向对象的特性。

创建一个接口：按照约定，接口的名字由方法名加 `[e]r` 后缀组成，例如 `Printer`、`Reader`、`Writer` 等。还有一些不常用的方式（当后缀 `er` 不合适时），接口名以 `able` 结尾，或者以 `I` 开头。

```go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```

实现一个接口：

```go
package main

import (
	"fmt"
)

type IPhone interface {
	call()
}

type NokiaPhone struct {

}

func (p NokiaPhone) call() {
	fmt.Println("I am NokiaPhone, I can call you!")
}

func main() {
	var phone IPhone // phone 可以接收任何实现 IPhone 接口的结构体

	phone = new(NokiaPhone)
	phone.call()
}
```

### 反射

反射可以在运行时检查类型和变量，例如它的大小、方法和 `动态` 的调用这些方法。这对于没有源代码的包尤其有用。除非真得有必要，否则应当避免使用或小心使用。在 Go 语言中，通过使用 `reflect` 包进行反射。

**举例：通过反射修改值 **

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var x float64 = 3.4
	v := reflect.ValueOf(x)
	// setting a value:
	// v.SetFloat(3.1415) // Error: will panic: reflect.Value.SetFloat using unaddressable value
	fmt.Println("settability of v:", v.CanSet())
	v = reflect.ValueOf(&x) // Note: take the address of x.
	fmt.Println("type of v:", v.Type())
	fmt.Println("settability of v:", v.CanSet())
	v = v.Elem()
	fmt.Println("The Elem of v is: ", v)
	fmt.Println("settability of v:", v.CanSet())
	v.SetFloat(3.1415) // this works!
	fmt.Println(v.Interface())
	fmt.Println(v)
}
```

**举例：反射结构 **

```go
package main

import (
	"fmt"
	"reflect"
)

type NotknownType struct {
	s1, s2, s3 string
}

func (n NotknownType) String() string {
	return n.s1 + " - " + n.s2 + " - " + n.s3
}

// variable to investigate:
var secret interface{} = NotknownType{"Ada", "Go", "Oberon"}

func main() {
	value := reflect.ValueOf(secret) // <main.NotknownType Value>
	typ := reflect.TypeOf(secret)    // main.NotknownType
	// alternative:
	//typ := value.Type()  // main.NotknownType
	fmt.Println(typ)
	knd := value.Kind() // struct
	fmt.Println(knd)

	// iterate through the fields of the struct:
	for i := 0; i < value.NumField(); i++ {
		fmt.Printf("Field %d: %v\n", i, value.Field(i))
		// error: panic: reflect.Value.SetString using value obtained using unexported field
		//value.Field(i).SetString("C#")
	}

	// call the first method, which is String():
	results := value.Method(0).Call(nil)
	fmt.Println(results) // [Ada - Go - Oberon]
}
```

### 三要素

####封装

即：数据隐藏，Go语言中将访问层次简化为两层 (public 和 private)，参考可见性规则：

> 当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 private ）。

#### 继承

用组合实现：内嵌一个（或多个）包含想要的行为（字段和方法）的类型；多重继承可以通过内嵌多个类型实现

```go
package main

import (
	"fmt"
)

type Person struct {
	Id   int64
	Name string
	status bool
}

func (p *Person) GetId() int64 {
	return p.Id
}

// 继承了 Person
type Student struct {
	Person 
	score int64
}

// 父类 GetId 方法重写
func (s *Student) GetId() int64 {
	return 1000 + s.Id
}

func (s *Student) GetScore() int64 {
	return s.score
}

func (s *Student) SetScore(score int64) {
	s.score = 100
}

func main() {
	student := Student{Person{1, "mars", true}, 99} // 初始化
	fmt.Printf("%v's id= %d, score = %d\n", student.Name, student.GetId(), student.GetScore())
	student.SetScore(100)
	student.GetScore()
	fmt.Printf("%v's id= %d, score = %d\n", student.Name, student.GetId(), student.GetScore())
}
```

#### 多态

用接口实现：某个类型的实例可以赋给它所实现的任意接口类型的变量。类型和接口是松耦合的，并且多重继承可以通过实现多个接口实现。Go 接口不是 Java 和 C# 接口的变体，而且接口间是不相关的，并且是大规模编程和可适应的演进型设计的关键。

```go
package main

import (
	"fmt"
)

type IPhone interface {
	call()
}

type NokiaPhone struct {

}

func (p NokiaPhone) call() {
	fmt.Println("I am NokiaPhone, I can call you!")
}

type ApplePhone struct {

}

func (p ApplePhone) call() {
	fmt.Println("I am ApplePhone, I can call you!")
}

func main() {
	var phone IPhone

	phone = new(NokiaPhone)
	phone.call()

	phone = new(ApplePhone)
	phone.call()
}
```

## 并发访问

对象的字段（属性）不应该由 2 个或 2 个以上的不同线程在同一时间去改变。如果在程序发生这种情况，为了安全并发访问，可以使用包 `sync` 包中的方法。

