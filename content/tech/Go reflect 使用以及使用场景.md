---
title: "Go reflect 使用以及使用场景
"
date: 2020-03-20
tags: ["go"]
categories: ["技术"]
isCJKLanguage: true
---

# Go reflect 使用以及使用场景

🌿本文源自各文章的总结，加上自己的一些理解与修改



## reflect 是什么

**定义**

> reflect（反射），在计算机学中是指计算机程序在[运行时](https://zh.wikipedia.org/wiki/运行时)（runtime）可以访问、检测和修改它本身状态或行为的一种能力。用比喻来说，反射就是程序在运行的时候能够“观察”并且修改自己的行为。



没懂。。。



**历史背景**

早期计算机的原生汇编语言本质上就具有反射特性。因为它是由定义编程指令作为数据，如动态修改指令或对它们进行分析等等的反射功能是很平常的。编程发展到如C语言等高抽象层次的语言时，这种实践消失了，带有反射特性的高级编程语言要到更晚的时候才出现。



**为什么需要反射**

强类型语言在编译期间会对对象（变量）作类型，接口，字段，方法等作合法性检测，反射技术则允许将对需要调用的对象的信息检查工作从编译期间推迟到运行期间再现场执行。这样一来，可以在编译期间先不明确目标对象的接口名称、字段（fields，即对象的成员变量）、可用方法，然后在运行根据目标对象自身的信息决定如何处理。它还允许根据判断结果进行实例化新对象和相关方法的调用。

反射主要用途就是使给定的程序动态地适应不同的运行情况。利用面向对象建模中的多态性也可以简化编写分别适用于多种不同情形的功能代码，**但是反射可以解决多态性并不适用的更普遍情形，从而更大程度地避免硬编码（即把代码的细节“写死”，缺乏灵活性）的代码风格**。



**总结**

反射并不是语言必须实现的，是为了更灵活的编码。

没看懂的话，可以结合下面的参考原文以及reflect使用场景理解。。。



## Go reflect 如何使用

### 原理

**类型设计**

Go的变量包括

- value
- type



而type有两种

- static type

- concrete type

  

如int， float， string这些类型的type为static type，在声明时便已经确定；

而interface的type为concrete type，runtime才看得见的类型，故只有inteface类型才有反射一说。



**interface**

interface包括

- value
- (concrete)type

 故可以根据这type实现reflect



### 使用

**`TypeOf`**

获取intaface的类型



**`ValueOf`**

获取interface的具体值



example

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var num int = 233
	var i interface{} = num

	fmt.Println("type: ", reflect.TypeOf(i))
	fmt.Println("value: ", reflect.ValueOf(i))
}


// output:
// type:  int
// value:  233
```



**类型转换**

简单类型， 如int， float64， string

example

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var i int = 233
	
	p := reflect.ValueOf(&i)
	v := reflect.ValueOf(i)

	cp := p.Interface().(*int)
	cv := v.Interface().(int)

	fmt.Println(cp, cv)
}

// output
// 0xc000016070 233
```



注意，该类型断言如若类型不符合，例如将int类型转换的interface断言为*int，会panic。

可以尝试用`v, ok := xx.(xx)`预先判断或者用`switch xx.(type)`来循环判断。



ps： 演示代码不具有实际意义



**复合类型获取字段类型，方法**

example

```go
package main

import (
	"fmt"
	"reflect"
)

type t1 struct {
	A string
	B int
}

func (t t1) Fn() {
	fmt.Println("this is t1's method")
}

func main() {

	st := t1{
		A: "ss",
		B: 233,
	}

	var t interface{} = st
	
	rt := reflect.TypeOf(t)
	fmt.Println("t's type", rt.Name())

	rv := reflect.ValueOf(t)
	fmt.Println("t's value", rv)

	for i:=0; i<rt.NumField(); i++{
		field := rt.Field(i)
		value := rv.Field(i).Interface()
		fmt.Printf("field's name: %s, type: %v, value: %v\n", field.Name, field.Type, value)
	}
	
	// 反射似乎不检测非导出方法
	for i:=0; i<rt.NumMethod(); i++{
		m := rt.Method(i)
		fmt.Printf("method'name: %s, type: %v\n", m.Name, m.Type)
	}

}

// output
// t's type t1
// t's value {ss 233}
// field's name: A, type: string, value: ss
// field's name: B, type: int, value: 233
// method'name: Fn, type: func(main.t1)

```



**修改原来的值**

example

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	var i int = 233

	// 指针类型才可以修改值
	p := reflect.ValueOf(&i)
  // 获取原始反射对象
	pe := p.Elem()

	fmt.Println("type of pe: ", pe.Type())
	fmt.Println("can set?", pe.CanSet())

	pe.SetInt(23)
	fmt.Println("after change", i)
}

// output
// type of pe:  int
// can set? true
// after change 23

```



**动态调用方法**

example

```go
package main

import (
	"fmt"
	"reflect"
)

type t1 struct {
	A string
	B int
}

func (t t1) Fn() {
	fmt.Println("this is t1's method")
}

func main() {

	st := t1{
		A: "ss",
		B: 233,
	}

	var t interface{} = st

	rv := reflect.ValueOf(t)

	method1 := rv.MethodByName("Fn")
	// Fn方法不需要参数，这里传len为0的[]reflect.Value既可
	args := make([]reflect.Value, 0)
	method1.Call(args)

}

// output
// this is t1's method

```



## reflect 使用场景

Emm, 上面的代码很抽象，好像并没有什么卵用。。。所以说反射不是语言实现的必须，只是为了更加灵活，减少一些硬编码。

这里总结下reflect的使用场景，或许更好理解



**struct tag解析**

json解析， orm框架可以根据结构体的tag信息动态映射字段



**抽象类型**

如fmt一系列方法都使用了interface来传参数，大大减少了代码行数



**动态调用函数**

还没想到实际用处，但感觉很有用

如gRPC框架会用到？



**判读是否实现了某接口**

example

```go
type IT interface {
    test1()
}

type T struct {
    A string
}

func (t *T) test1() {}

func main() {
    t := &T{}
    ITF := reflect.TypeOf((*IT)(nil)).Elem()
    tv := reflect.TypeOf(t)
    fmt.Println(tv.Implements(ITF))
}
```



## 什么时候使用reflect

> 1. 为了降低多写代码造成的bug率，做更好的归约和抽象
> 2. 为了灵活、好用、方便，做动态解析、调用和处理
> 3. 为了代码好看、易读、提高开发效率，补足与动态语言之间的一些差别



## 注意

reflect具有性能问题，但是实际使用中可以先实现功能，后面再考虑优化。

毕竟性能是个伪命题？



## 参考

- [为什么语言里要提供「反射」功能？](https://www.zhihu.com/question/28570203)

- [Golang的反射reflect深入理解和示例](https://juejin.im/post/5a75a4fb5188257a82110544)
- [Go Reflect 高级实践](https://segmentfault.com/a/1190000016230264)

