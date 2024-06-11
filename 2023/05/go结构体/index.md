# Go结构体


# 结构体

## 声明结构体

>  基本语法

```go
type 结构体名称 struct{
    field1 type
    field2 type
}
```

## 字段/属性

>  基本介绍

1. 结构体字段 = 属性 = field
2. 字段是结构体的一个组成部分，一般是基本数据类型、数组，也可是引用类型

> 注意事项

1. 字段声明语法同变量，示例：`Name string`
2. 字段的类型可以为：基本类型、数组或引用类型
3. 在创建一个结构体变量后，如果没有给字符赋值，都对应一个零值(默认值)
    1. 布尔类型是`false`，数值是`0`，字符串是`""`
    2. 数组类型的默认值和它的元素类型相关，比如：`score[3] int` 默认值：`[0,0,0]`
    3. 指针、slice和map的零值都是`nil`，即还没有分配空间，需要`make`后使用
4. 不同结构体变量的字段是独立的，互不影响，一个结构体变量字段的更改，不影响另外一个

## 创建结构体变量

> 方式一：直接声明

```go
var person Person
```

> 方式二：`{}`

```go
var person Person = Person{}
```

> 方式三：`new`

```go
var person *Person = new(Person)
```

> 方式四：`&`

```go
var person *Person = &Person{}
```

**说明**

1. 第3、4种方式返回的是**结构体指针**。
2. 结构体指针访问字段的标准方式：`(*结构体指针).字段名`，比如：`(*person).Name = "tom"`
3. 对于结构体指针访问字段，golang做了简化操作，也支持 `结构体指针.字段名`，比如：`person.Name = "tom"`。这样更加符合程序员使用习惯，golang底层进行了一步转化操作。

## 结构体内存分配机制

### 结构体变量在内存中存在示例图

> 变量总是存在内存中的，那么结构体变量也是一样的

```go
var p1 Person
p1.Name = "Joker"
p1.Age = 18

var p2 Person = p1

fmt.Println(p2.Age) //18

p2.Name = "Tommy"
fmt.Printf("p2.Name=%v, p1.Name=%v\n", p2.Name, p1.Name) //p2.Name=Tommy, p1.Name=Joker

```

**示例图：**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305191249149.png" alt="image-20230519124928045" style="zoom:25%;" />

### 结构体指针变量在内存中存在示例图

```go
var p1 Person
p1.Name = "Joker"
p1.Age = 18

var p2 *Person = &p1

fmt.Println((*p2).Age) //18
fmt.Println(p2.Age)    //18

p2.Name = "Tommy"
fmt.Printf("p2.Name=%v, p1.Name=%v\n", p2.Name, p1.Name)    //p2.Name=Tommy, p1.Name=Tommy
fmt.Printf("p2.Name=%v, p1.Name=%v\n", (*p2).Name, p1.Name) //p2.Name=Tommy, p1.Name=Tommy

```

**示例图：**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305191254300.png" alt="image-20230519125449214" style="zoom: 25%;" />

## 结构体使用注意事项和细节

> 1. 结构体的所有字段在内存中是连续的

```go
package main

import "fmt"

type Point struct {
	x int
	y int
}

type Rect struct {
	leftUp, rightDown Point
}

type Rect2 struct {
	leftUp, rightDown *Point
}

func main() {
	r1 := Rect{Point{1, 2}, Point{3, 4}}
	// r1有4个int，在内存中是连续分布的
	fmt.Printf("r1.leftUp.x的地址=%p r1.leftUp.y的地址=%p r1.rightDown.x的地址=%p r1.rightDown.y的地址=%p\n",
		&r1.leftUp.x, &r1.leftUp.y, &r1.rightDown.x, &r1.rightDown.y)
	// r1.leftUp.x的地址=0x1400012c000 r1.leftUp.y的地址=0x1400012c008 r1.rightDown.x的地址=0x1400012c010 r1.rightDown.y的地址=0x1400012c018

	r2 := Rect2{&Point{10, 20}, &Point{30, 40}}
	// r2有2个*Point，在内存中2个*Point本身地址也是连续分布的
	fmt.Printf("r2.leftUp 本身地址=%p r2.rightDown 本身地址=%p\n", &r2.leftUp, &r2.rightDown) //r2.leftUp 本身地址=0x1400008e210 r2.rightDown 本身地址=0x1400008e218
	// 但是它们指向的地址不一定是连续的
	fmt.Printf("r2.leftUp 指向地址=%p r2.rightDown 指向地址=%p\n", r2.leftUp, r2.rightDown) //r2.leftUp 指向地址=0x140000a8010 r2.rightDown 指向地址=0x140000a8020
}
```

**内存分布图：**

![image-20230520155653436](https://raw.githubusercontent.com/XD825/picgo/main/img/202305201556497.png)

>2. 结构体是用户单独定义的类型，和其它类型进行转换时需要有完全相同的字段（名字、个数和类型）

```go
package main

import "fmt"

type A struct {
	Num int
}
type B struct {
	Num int
}

func main() {
	var a A
	var b B
	a = A(b) // 可以进行转换
	fmt.Println(a, b)
}
```

>3. 结构体进行type重新定义（相当于取别名），Golang认为是新的数据类型，但是相互间可以强转

```go
type Student struct {
    Name string
    Age  int
}
type Stu Student

var stu1 Student
var stu2 Stu
//stu2 = stu1 //cannot use stu1 (variable of type Student) as type Stu in assignment
stu2 = Stu(stu1)
fmt.Println(stu1, stu2)
```

> 4. struct的每个字段上，可以写上一个tag，该tag可以通过反射机制获取，常见的使用场景就是序列化和反序列化

![image-20230520165613667](https://raw.githubusercontent.com/XD825/picgo/main/img/202305201656766.png)

# 方法

Golang中的方法是作用在指定的数据类型上（即：和指定的数据类型绑定），因此自定义类型，都可以有方法，而不仅仅是struct

## 声明方法

```go
func (recevier type) methodName (参数列表) （返回值列表）{
    // 方法体
    return 返回值
}
```

1. `recevier type`：表示这个方法和type这个类型进行绑定，或者说该方法作用于type类型
2. `recevier type`：type可以是结构体，也可以是其它的自定义类型
3. `recevier` ：就是type类型的一个变量（实例），
4. 参数列表：方法输入
5. 返回值列表：返回值，可以有多个
6. 方法体：逻辑代码
7. `return`：不是必须的

**示例：**

![image-20230520172049576](https://raw.githubusercontent.com/XD825/picgo/main/img/202305201720656.png)

## 方法的调用和传参机制原理

> 说明：方法的调用和传参机制和函数基本一样，不一样的地方是方法调用时，会将调用方法的变量，当做实参也传递给方法。

![image-20230520181112805](https://raw.githubusercontent.com/XD825/picgo/main/img/202305201811884.png)

## 方法注意事项和细节

> 1、结构体类型是值类型，在方法调用中，遵守值类型的传递机制，是值拷贝传递方式。

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305202308079.png" alt="image-20230520230818988" style="zoom: 50%;" />

> 2、如果希望在方法中修改结构体变量的值，可以通过结构体指针的方式进行处理。（这种方式用的比较多，因为**指针传递效率高**）

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305202320497.png" alt="image-20230520232054408" style="zoom:50%;" />

> 3、Golang中的方法作用在指定的数据类型上（即：和指定的数据类型绑定），因此自定义类型，都可以有方法，而不仅仅是struct。

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305202329721.png" alt="image-20230520232957639" style="zoom: 50%;" />

> 4、方法的访问范围控制的规则，和函数一样。方法名首字母小写，只能在本包访问，方法首字母大写，可以在本包和其他包访问。

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305202348561.png" alt="image-20230520234833458" style="zoom:50%;" />

> 5、如果一个变量实现了`String()`这个方法，那么`fmt.Println`默认会调用这个变量的`String()`进行输出。

![image-20230520234552577](https://raw.githubusercontent.com/XD825/picgo/main/img/202305202345688.png)

## 方法 VS 函数

> 1、调用方式不一样

函数的调用方式：函数名(实参列表)

方法的调用方式：变量.方法名(实参列表)

> 2、普通函数，接收者为值类型时，不能将指针类型的数据直接传递

![image-20230521083614455](https://raw.githubusercontent.com/XD825/picgo/main/img/202305210836632.png)

> 3、对于方法，接收者为值类型时，可以直接用指针类型的变量调用方法，反过来也可以。
>
> 注：不管调用形式如何，真正决定是值拷贝还是地址拷贝还是看方式是和哪种类型绑定

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305210842691.png" alt="image-20230521084208591" style="zoom:50%;" />

# 工厂模式

Golang的结构体没有构造函数，通常可以使用工厂模式来解决这个问题。

**使用工厂模式解决结构体首字母小写在别的包调用问题：**

![image-20230521091158468](https://raw.githubusercontent.com/XD825/picgo/main/img/202305210911549.png)

 

