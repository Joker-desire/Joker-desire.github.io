# Go面向对象三大特性


# 基本介绍
Golang仍然有面向对象编程的继承、封装和多态的特性，只是实现的方式和其他OOP语言不一样。

# 封装

## 封装的介绍

封装（encapsulation）就是把抽象出的**字段**和**字段的操作**封装在一起，数据被保护在内部，程序的其他包只有通过被授权的操作（方法），才能对字段进行操作。

## 封装的好处

1. 隐藏实现细节
2. 可以对数据进行验证，保证安全合理

## 如何体现封装

1. 对结构体中的属性进行封装
2. 通过方法、包实现封装

## 封装的实现步骤

1. 将结构体、字段（属性）的首字母小写（类似private）

    ```go
    type student struct {
    	name string
    	age  int
    }
    ```

2. 给结构体所在包提供一个工厂模式的函数，首字母大写。类似一个构造函数

    ```go
    func NewStudent() *student {
    	return &student{}
    }
    ```

3. 提供一个首字母大写的Set方法，用于对属性判断并赋值

    ```go
    func (s *student) SetName(name string) {
    	s.name = name
    }
    
    func (s *student) SetAge(age int) {
    	if age > 0 && age < 150 {
    		s.age = age
    	}
    }
    ```

4. 提供一个首字母大写的Get方法，用户获取属性的值

    ```go
    func (s *student) GetName() string {
    	return s.name
    }
    
    func (s *student) GetAge() int {
    	return s.age
    }
    ```

5. 调用

    ```go
    student := model.NewStudent()
    student.SetName("Joker")
    student.SetAge(19)
    fmt.Println(student.GetName(), student.GetAge()) //Joker 19
    ```

# 继承

## 继承的介绍

继承可以**解决代码复用**，让编程更加靠近人类思维。

当多个结构体存在相同的属性（字段）和方法时，可以从这些结构体中抽象出结构体，在该结构体中定义这写相同的属性和方法。其他的结构体不需要重新定义这些属性和方法，只需嵌套一个抽象出来的匿名结构体即可。

也就是说：在Golang中，如果一个struct嵌套了另一个匿名结构体，那么这个结构体可以直接访问匿名结构体的字段和方法，从而实现了继承特性。

## 嵌套匿名结构体的基本语法

```go
type Person struct {
	Name string
	Age  int
}

type Student struct {
	Person // 这就是嵌套匿名结构体
	Score  float64
}
```

## 继承的深入讨论

1、结构体可以使用嵌套匿名结构体所有的字段和方法，即：首字母大写或小写的字段、方法都可以使用

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305211011070.png" alt="image-20230521101152939" style="zoom:50%;" />

2、匿名结构体字段访问可以简化

![image-20230521101636484](https://raw.githubusercontent.com/XD825/picgo/main/img/202305211016572.png)

3、当结构体和匿名结构体有相同的字段或方法时，编译器采用就近访问原则访问，如希望访问匿名结构体的字段和方法，可以通过匿名结构体名来区分

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305211744516.png" alt="image-20230521174451406" style="zoom: 50%;" />

4、结构体嵌入两个（或多个）匿名结构体，如两个匿名结构体有相同的字段和方法（同时结构体本身没有同名的字段和方法），在访问时，就必须明确指定匿名结构体名字，否则编译报错

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305211755304.png" alt="image-20230521175559175" style="zoom:50%;" />

5、如果一个struct嵌套了一个有名结构体，这种模式就是组合，如果是组合关系，那么在访问组合的结构体的字段或方法时，必须带上结构体的名字

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305211801386.png" alt="image-20230521180115283" style="zoom:50%;" />

6、嵌套匿名结构体后，也可以在创建结构体变量（实例）时，直接指定各个匿名结构体字段的值

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305212138685.png" alt="image-20230521213829577" style="zoom:50%;" />

# 接口（interface）

## 基本介绍

interface类型可以定义一组方法，但是这些方法不需要实现。并且interface不能包含任何变量。

## 基本语法

```go
type 接口名 interface {
    method1(参数列表) 返回值列表
    method2(参数列表) 返回值列表
}
```

**实现接口所有方法**

```go
func (c 自定义类型) method1(参数列表) 返回值列表 {
    // 方法实现
}

func (c 自定义类型) method2(参数列表) 返回值列表 {
    // 方法实现
}
```

**说明：**

1. 接口里的所有方法都没有方法体，即接口的方法都是没有实现的。接口体现了程序设计的多态和高内聚低耦合的思想。
2. Golang中的接口，不需要显式的实现。只要一个变量，含有接口类型中的所有方法，那么这个变量就实现了这个接口。

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305221243522.png" alt="image-20230522124356338" style="zoom:50%;" />

## 注意事项和细节

**1、接口本身不能创建实例，但是可以指向一个实现了该接口的自定义类型的变量（实例）**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231230115.png" alt="image-20230523123002964" style="zoom:50%;" />

**2、接口中所有的方法都没有方法体，即都是没有实现的方法**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231235743.png" alt="image-20230523123507646" style="zoom:50%;" />

**3、在Golang中，一个自定义类型需要将某个接口的所有方法都实现，那么这个自定义类型就实现了该接口**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231233498.png" alt="image-20230523123357406" style="zoom:50%;" />

**4、一个自定义类型只有实现了某个接口，才能将该自定义类型的实例（变量）赋给接口类型**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231238740.png" alt="image-20230523123832626" style="zoom:50%;" />

**5、只要是自定义数据类型，就可以实现接口，不仅仅是结构体类型**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231242349.png" alt="image-20230523124247256" style="zoom:50%;" />

**6、一个自定义类型可以实现多个接口**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231249482.png" alt="image-20230523124938318" style="zoom:50%;" />

**7、Golang接口中不能有任何变量**

![image-20230523125101715](https://raw.githubusercontent.com/XD825/picgo/main/img/202305231251820.png)

**8、一个接口可以继承多个别的接口，如果要实现接口，也必须将接口继承的别的接口的方法也全部实现**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231253006.png" alt="image-20230523125310877" style="zoom:50%;" />

![image-20230523173841088](https://raw.githubusercontent.com/XD825/picgo/main/img/202305231738298.png)

**9、interface类型默认是一个指针（引用类型），如果没有对interface初始化就使用，那么会输出nil**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231337771.png" alt="image-20230523133748670" style="zoom:50%;" />

**10、空接口interface{}没有任何方法，所以所有类型都实现了空接口，即可以把任何一个变量赋值给空接口**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231306237.png" alt="image-20230523130622113" style="zoom:50%;" />

## 接口经典案例（结构体排序）

```go
package main

import (
	"fmt"
	"math/rand"
	"sort"
)

type Student struct {
	Name string
	Age  int
}

type StudentSlice []Student

func (s StudentSlice) Len() int {
	// 获取切片长度
	return len(s)
}

func (s StudentSlice) Less(i, j int) bool {
	// 按照年龄升序
	return s[i].Age < s[j].Age
}

func (s StudentSlice) Swap(i, j int) {
	// 交换顺序
	s[i], s[j] = s[j], s[i]
}
func main() {

	slice := StudentSlice{}
	for i := 0; i < 10; i++ {
		student := Student{Name: fmt.Sprintf("Joker-%d", rand.Intn(100)), Age: rand.Intn(150)}
		slice = append(slice, student)
	}
	fmt.Println("原结构体顺序：")
	for _, stu := range slice {
		fmt.Println(stu)
	}

	sort.Sort(slice)
	fmt.Println("排序后结构体顺序：")
	for _, stu := range slice {
		fmt.Println(stu)
	}

}
```

## 接口VS继承

### 案例

```go
package main

import "fmt"

type IBird interface {
	Fly()
}

type IFish interface {
	Swim()
}

type Monkey struct {
	Name string
}

func (m *Monkey) ClimbTree() {
	fmt.Println(m.Name, "会爬树...")
}

type SmallMonkey struct {
	Monkey // 继承了Monkey结构体
}

// Fly 给SmallMonkey延伸Fly功能
func (m SmallMonkey) Fly() {
	fmt.Println(m.Name, "学会了飞行...")
}

// Swim 给SmallMonkey延伸Swim功能
func (m SmallMonkey) Swim() {
	fmt.Println(m.Name, "学会了游泳...")
}

func main() {
	smallMonkey := SmallMonkey{Monkey{Name: "悟空"}}
	smallMonkey.ClimbTree()
	smallMonkey.Fly()
	smallMonkey.Swim()

	var b IBird = smallMonkey
	b.Fly()

	var f IFish = smallMonkey
	f.Swim()

}
```

### 总结

1. 当A结构体继承了B结构体，那么A结构体就自动的继承了B结构体的字段和方法，并且可以直接使用
2. 当A结构体需要扩展功能，同时不希望去破坏继承关系，则可以去实现某个接口即可，因此可以认为，实现接口是对继承机制的补充

### 接口 VS 继承

1. 接口和继承解决的问题不同
    1. 继承的价值主要在于：解决代码的复用性和可维护性
    2. 接口的价值主要在于：设计，设计好各种规范（方法），让其它自定义类型去实现这些方法。

2. 接口比继承更加灵活
    1. 接口比继承更加灵活，继承是满足`is - a`的关系，而接口只需满足`like - a`的关系


3. 接口在一定程度上实现代码解耦

# 多态

## 基本介绍

变量（实例）具有多种形态。在Golang中，多态特性是通过接口实现的。可以按照统一的接口来调用不同的实现。这时接口变量就呈现不同的形态。

## 接口体现多态的两种形式

1. 多态参数

    <img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231818141.png" alt="image-20230523181834047" style="zoom:50%;" />

2. 多态数组

    <img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305231821563.png" alt="image-20230523182138445" style="zoom:50%;" />

## 类型断言

类型断言，由于接口是一般类型，不知道具体类型，如果要转成具体类型，就需要使用类型断言。

```go
func main() {
	// 类型断言
	var x interface{}
	var b float64 = 29.9
	x = b                                 // 空接口，可以接受任意类型
	y := x.(float64)                      // 使用类型断言
	fmt.Printf("y 的类型是 %T 值是 %v\n", y, y) // y 的类型是 float64 值是 29.9
}
```

**进行断言时进行检测：**

```go
func main() {
	// 类型断言
	var x interface{}
	var b float64 = 29.9
	x = b                         // 空接口，可以接受任意类型
	if f, ok := x.(float32); ok { // 进行检测
		y := f                                // 使用类型断言
		fmt.Printf("y 的类型是 %T 值是 %v\n", y, y) // y 的类型是 float64 值是 29.9
	} else {
		fmt.Println("convert fail")
	}

}
```


