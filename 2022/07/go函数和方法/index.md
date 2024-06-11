# Go函数和方法


## Go包

### 包的基本概念

go的每一个文件都是属于一个包的，也就是说go是以包的形式来管理文件和项目目录结构的

### 包的三大作用

1. 区分相同名字的函数、变量等标识符
2. 当程序文件很多时，可以很好的管理项目
3. 控制函数、变量等访问范围，即作用域

### 包的使用

#### 打包基本语法

`package util`

#### 导包基本语法

`import "包的路径"`



#### 注意事项

1. 在给一个文件打包时，该包对应一个文件夹

2. 当一个文件要使用其他包函数或变量时，需要先引入对应的包

3. 为了让其他包的文件，可以访问到本包的函数，则该函数名的首字母需要大写

    ```go
    func Call(...){
        ...
    }
    ```

    

4. package指令在文件第一行，然后是import指令

    ```go
    package error
    
    import (
    	"errors"
    	"fmt"
    	"strconv"
    	"testing"
    )
    ```

5. 在访问其他包函数变量时，其语法是：包名.函数名

    ```go
    m := cm.CreateConcurrentMap(99)
    m.Set(cm.StrKey("key"), 10)
    t.Log(m.Get(cm.StrKey("key")))
    ```

6. 如果包名较长，Go支持给包取别名，注意细节：取别名后，原来的包名就不能使用了

    ```go
    import (
    	cm "github.com/easierway/concurrent_map"
    	"testing"
    )
    ```

    

7. 在同一包下，不能有相同的函数名。

8. 如果要编译成一个可执行程序文件，就需要将这个包声明为`main`，即`package main`这个就是一个语法规范，如果是写一个库，包名可以自定义

## Go函数

为完成某一功能的程序执行（语句）的集合，称为函数。

在Go中，函数分为：自定义函数、系统函数

### 基本语法

```go
func 函数名 (形参列表) (返回值列表) {
    执行语句...
    return 返回值列表
}
```

1. 形参列表：表示函数的输入
2. 函数中的语句：表示为了实现某一功能代码块
3. 函数可以有返回值，也可以没有

### 示例

```go
func compute(a int, b int, operator string) int {
	var result int
	switch operator {
	case "+":
		result = a + b
	case "-":
		result = a - b
	case "*":
		result = a * b
	case "/":
		result = a / b
	default:
		panic("运算符错误")

	}
	return result

}
```

### 函数参数的传递方式

1. 值传递：基本数据类型int系列、float系列、bool、string、数组和结构体struct
2. 引用传递：指针、slice切片、map、管道chan、interface等

不管是值传递还是引用传递，传递给函数的都是变量的副本，不同的是，值传递的是值的拷贝，引用传递的是地址的拷贝，一般来说，地址拷贝效率高，因为数据量小，而值拷贝决定拷贝的数据大小，数据越大，效率越低。

### 变量作用域

1. 函数内部声明/定义的变量叫局部变量，作用域仅限于函数内部
2. 函数外部声明/定义的变量叫全局变量，作用域在整个包都有效，如果其首字母为大写，则作用域在整个程序有效
3. 如果变量是在一个代码块，比如for/if中，那么这个变量的作用域就在改代码块

### 函数注意事项

1. 函数的形参列表可以是多个，返回值列表也可以是多个

2. 形参列表和返回值列表的数据类型可以是值类型和引用类型

3. 函数的命名遵循标识符命名规范，首字母不能是数字，首字母大写该函数可以被本包文件和其他包文件使用，类似public，首字母小写，只能被本包文件使用，其他包文件不能使用，类似private

4. 函数中的变量是局部的，函数外不生效

5. 基本数据类型和数组默认都是值传递的，即进行值拷贝。在函数内修改，不会影响到原来的值

6. 如果希望函数内的变量能修改函数外的变量，可以传入变量的地址&，函数内以指针的方式操作变量。从效果上看类似引用。

7. Go函数不支持重载

8. 在Go中，函数也是一种数据类型，可以赋值给一个变阿玲，则该变量就是一个函数类型的变量了。通过该变量可以对函数调用。

9. 函数既然是一种数据类型，因此在Go中，函数可以作为形参，并且调用

10. 为了简化数据类型定义，Go支持自定义数据类型

    1. 基本语法：`type 自定义数据类型名 数据类型 // 相当于一个别名`

11. 支持函数返回值命名

    ```go
    func call(n1 int, n2 int) (sum int, sub int) {
    	sum = n1 + n2
    	sub = n1 - n2
    	return
    }
    ```

12. 使用`_`表示符，忽略返回值

    ```go
    func TestFunc(t *testing.T) {
    	sum, _ := call(1, 2)
    	fmt.Println(sum)
    }
    ```

13. Go支持可变参数

    1. args是slice，通过args[index]可以访问到各个值
    2. 如果一个函数的形参列表中有可变参数，则可变参数需要放在形参列表最后

    ```go
    // 支持0到多个参数
    func call2(args ...int) int {
    	sum := 0
    	for _, arg := range args {
    		sum += arg
    	}
    	return sum
    }
    // 支持1到多个参数
    func call3(n1 int, args ...int) int {
    	sum := n1
    	for _, arg := range args {
    		sum += arg
    	}
    	return sum
    }
    
    func TestFunc(t *testing.T) {
    	println(call2(1, 2, 3, 4))
    	println(call3(10, 1, 2, 3, 4))
    }
    ```



## `init`函数

每一个源文件都可以包含一个`init`函数，该函数会在main函数执行前，被Go运行框架调用，也就是说`init`会在`main`函数前被调用

```go
package main

import "fmt"

func init() {
	fmt.Println("main init...")
}

func main() {
	fmt.Println("main ...")
}
```

### 注意细节

1. 如果一个文件同时包含全局变量定义，`init`函数和`main`函数，则执行的流程是变量定义->init函数->main函数
2. init函数最主要的作用，就是完成一些初始化的工作



## 匿名函数

Go支持匿名函数，如果我们某个函数只是希望使用一次，可以考虑使用匿名函数，匿名函数也可以实现多次调用。

### 使用方式1

在定义匿名函数时就直接调用

```go
result := func(a int, b int) int {
    return a + b
}(1, 2)
fmt.Println(result)

```

### 使用方式2

将匿名函数赋给一个变量（函数变量），在通过该变量来调用匿名函数

```go
sum := func(a int, b int) int {
    return a + b
}
fmt.Println(sum(1, 2))
```



### 全局匿名函数

如果将匿名函数赋给一个全局变量，那么这个匿名函数，就成为一个全局匿名函数，可以在程序有效



## 闭包

闭包就是一个函数和与其相关的引用环境组合的一个整体（实体）

```go
func AddUpper() func(int) int {
	var n int = 30
	return func(x int) int {
		n = n + x
		return n
	}
}

func main() {
	f := AddUpper()   //返回一个闭包
	fmt.Println(f(1)) //31
	fmt.Println(f(2)) //33
	fmt.Println(f(3)) //36
}
```



## defer延时机制

在函数中，程序员经常需要创建资源（比如：数据库连接、文件句柄、锁等），为了在函数执行完毕后，及时的释放资源，Go的设计者提供了`defer`（延时机制），类似于finally。

```go
func TestDefer(t *testing.T) {
	defer func() { //匿名函数
		t.Log("Clear resources.")
	}()
	fmt.Println("Started")
	panic("Fatal error") //抛出异常，defer仍会执行

}
```

### 细节说明

1. 当go执行到一个defer时，不会立即执行defer后的语句，而是将defer后的语句压入到一个栈中，然后继续执行函数下一个语句
2. 当函数执行完毕后（或者抛出异常后），在从defer栈中，一次从栈顶取出语句执行（注：遵守栈先入后出的机制）
3. 在defer将语句放入到栈时，也会将相关的值拷贝同时入栈


