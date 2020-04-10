## 随机数

你可以使用 `Seed(value)` 函数来提供伪随机数的生成种子，一般情况下都会使用当前时间的纳秒级数字。随机方法 `Math/rand` 必须结合 `Seed` 才可以正常获得随机值，否则只会是同一个值。

## 变量传递

Go 方法和函数默认是按值传递，每次将一个变量作为参数传递，都会创建一个新的变量并将其传递给所调用的函数和方法，副本分配在不同的内存地址，**修改变量数据不会影响原来的变量**。

下面例子表示，update 并没有修改 p1，而是复制了一份 p1 并进行了修改，所以存在 **内存的浪费**。

```go
package main

import (
	"fmt"
)

type Person struct {
	Id   int64  `json:"id"`
	Name string `json:"name"`
}

func update(p Person, name string) Person {
	p.Name = name
  return p
}

func main() {
	p1 := Person{
		Id:   1,
		Name: "person1",
	}

	fmt.Println(p1.Name) // persion1
	p2 := update(p1, "person2")
	fmt.Println(p1.Name) // persion1
	fmt.Println(p2.Name) // persion2
}

```

修改一下例子，将值传递改为引用传递，问题就解决了。

```go
package main

import (
	"fmt"
)

type Person struct {
	Id   int64  `json:"id"`
	Name string `json:"name"`
}

func update(p *Person, name string) {
	p.Name = name
}

func main() {
	p1 := Person{
		Id:   1,
		Name: "person1",
	}

	fmt.Println(p1.Name) // persion1
	update(&p1, "person2")
	fmt.Println(p1.Name) // persion2
}

```

注意点：

1. 变量是一个**大的结构**，建议使用指针传递，避免在内存中复制整个结构。
2. 变量是 map , chan, slice，这三种都属于引用类型。但是 slice 的内存地址可以直接通过 `%p` 打印，不需要 `&` 取地址符转换。
3. 通过在 update 方法里面打印变量的内存地址，可以确认所有的传参都是值传递，都是一个副本。当参数是引用类型的时候，即可修改参数数据。**引用类型和传引用是两个概念**
