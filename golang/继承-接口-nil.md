
# 继承

匿名成员变量



为什么先说这个东西，因为 GO 里严格来说就没<继承>这个东西，而是通过匿名成员变量间接实现了大家认知里的：继承

```go

type Animal struct {
	Name string
	Category int
}

type Rabbit struct {
	Animal
	Tail string
	Category int//这里定义了一个跟Animal一样的成员变量名

}

```

Rabbit 结构 继承了 Animal 结构

同时Rabbit 和 Animal 还共有一个相同名称的成员变量：Category，且互不影响...

>rabbit.Category
>rabbit.Animal.Category

函数重写

```go

func (animal *Animal)Eat(){

	fmt.Println("im Animal Eat")

}

func (rabbit *Rabbit)Eat(){
	fmt.Println("im rabbitKid Eat")
}

```

Rabbit 和 Animal 同时还拥有一个相同的函数： Eay()

依然互不影响

子结构也可以同时继承多个父结构体，但是多个父结构体之间有相同名字的函数时，子结构调用的时候得加上标识，如：标识A结构体.函数1,如果直接调用，会报错。

从JAVA角度看：也可以间接理解JAVA里为啥不允许多继承，从GO的角度看：≈JAVA的OOP那套体系被GO这种实现方式直接给扒个精光....

继承的作用？

1. 简少重复写代码
2. 统一入口、统一管理（父类的构造函数为公共的）

>想了很久，现实应用中就这么2个作用....

# interface 接口


定义一个接口，只要实现了此接口的所有方法，就算实现了些接口
>没有 implament



定义一个空接口（内部无任何函数名）

```go
type Animal interface{}
var a Animal
a = 1
a = "imstr"
```

一个变量即可以是整形又可以是字符串... 这就是GO里接口的强大用法。

```go
func PPP(x interface{})
```

定义一个函数，参数x为<空接口>类型，即：该函数可以接收任意类型的参数...

再变化一下

```go
func PPP(x ...interface{})
```

函数可以接收N个参数，且任意类型....这就是传说中的：多态

现实中大概率不会这么用：
1.  虽然你可以传进来任意类型，但是你使用的时候得先断言
2. 毕竟是强类型语言，传进来的东西还得再判断

>他不会给你真的提供弱类型语言特性，只是加了一点便捷而以....


#### 类型

1. interface 为空
2. interface 包含若干方法

runtime.iface：带方法的接口
runtime.eface：表示不带任何方法的空接口interface{}。


```go
type eface struct { // 16 字节
	_type *_type
	data  unsafe.Pointer
}
```


a := interface{}

a = nil

此时：
type = nil
data = nil



# nil

nil在Go语言中只能被赋值给指针和接口
>指针、channel、func、interface、map或slice类型的变量 (非基础类型) 


```
var nil Type
type Type int
```

目测：nil 最底层是 int .......