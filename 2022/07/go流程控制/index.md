# Go流程控制


## 顺序控制

程序从上到下逐行地执行，中间没有任何判断和跳转。

## 分支控制

分支控制有三种：单分支、双分支和多分支

### 单分支

```go
var age int
fmt.Println("请输入年龄：")
fmt.Scanln(&age)

if age > 18{
    fmt.Println("你已成年")
}
```

### 双分支

```go
if age > 18{
    fmt.Println("你已成年")
} else {
    fmt.Println("未成年")
}
```

### 多分支

```go
if score == 100{
    fmt.Println("很优秀")
} else if score >=80 && score <100 {
    fmt.Println("优秀")
} else if score >=60 && score <80 {
    fmt.Println("良好")
} else{
    fmt.Println("不及格")
}
```



## 循环控制

### switch分支结构

1. `switch`语句用于基于不同条件执行不同动作，每一个`case`分支都是唯一的，从上到下逐一测试，直到匹配到为止。
2. `case`后是一个表达式（即：常量值、变量、一个有返回值的函数等）
3. `case`后的各个表达式的值的数据类型，必须和`switch`的表达式数据类型一致
4. `case`后面可以带多个表达式，使用逗号间隔，
5. `case`后的表达式如果是常量值（字面量），则要求不能重复
6. 匹配项后面不需要再加`break`
7. `default`语句不是必须的
8. `Type Switch`: `switch`语句还可以被用于`type-switch`来判断某个`interface`变量中实际指向的变量类型

```go
var x interface{}
var y = 10.0
x = y
switch i := x.(type) {
    case nil:
    fmt.Printf("x 的类型：%T", i)
    case int:
    fmt.Printf("x 的类型：int")
    case float32:
    fmt.Printf("x 的类型：float32")
    case float64:
    fmt.Printf("x 的类型：float64")
    default:
    fmt.Printf("未知")
}
```

### for循环控制

```go
for i := 0; i <= 10; i++ {
    fmt.Println(i)
}
```

#### for-range

- 可以方便的遍历字符串和数组

```go
arr1 := [4]int{1, 2, 3, 4}
for index, val := range arr1 {
    fmt.Println(index, "-", val)
}
```

#### "while"循环

- 在Go语言中是没有while循环的，可以使用for循环进行替换
- `break`跳出循环
- `continue`跳过当循环，进入下一次循环

```go
for {
    fmt.Println("while循环")
}
```



## goto

1. Go语言的`goto`语句可以无条件地转移到程序中指定的行
2. `goto`语句通常与条件语句配合使用。可用来实现条件转移，跳出循环体等功能。
3. 在Go程序设计中一般不主张使用`goto`语句，以免造成程序流程的混乱，使理解和调试程序都产生困难。

```go
fmt.Println("run 001")
fmt.Println("run 002")
goto label1
fmt.Println("run 003") // 不会被打印
fmt.Println("run 004") // 不会被打印
fmt.Println("run 005") // 不会被打印
label1:
fmt.Println("run 006")
```


