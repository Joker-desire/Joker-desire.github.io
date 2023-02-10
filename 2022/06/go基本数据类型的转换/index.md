# Go基本数据类型的转换


### 介绍

Go在不同类型的变量之间赋值时需要显式转换。Go中数据类型不能自动转换。

### 基本语法

表达式T(v) 将值v转换为类型T

- T: 就是数据类型，比如：int32/int64/float32等
- v: 就是需要转换的变量

### 示例

```go
var i int = 42
var f float64 = float64(i)
var u uint8 = uint8(f)
fmt.Println(i, f, u) //42 42 42
```

### 细节说明

1. Go中，数据类型的转换可以是从表示范围小->表示范围大，也可以 范围大->范围小

2. 被转换的是**变量存储的数据**（即值），变量本身的数据类型并没有变化

3. 在转换中，比如将int64转成int8，编译时不会报错，只是转换的结果是按溢出处理，和期望结果不一样

    ```go
    var num int64 = 999999
    var num2 int8 = int8(num)
    fmt.Println(num2) //63 转换的结果被按照溢出处理，跟预期结果不一致
    ```

## 基本数据类型和string的转换

### 基本数据类型转string

#### 方式一：`fmt.Sprintf("%参数", 表达式)`

1. 参数需要和表达式的数据类型相匹配
2. `fmt.Sprintf(...)`会返回转换后的字符串

```go
var (
    num1      int     = 99
    num2      float64 = 23.34
    isChecked bool    = true
    myChar    byte    = 'h'
    str       string  // 空字符串
)

// int -> string
str = fmt.Sprintf("%d", num1)
fmt.Printf("str type %T str=%q\n\n", str, str) //str type string str="99"

// float -> string
str = fmt.Sprintf("%f", num2)
fmt.Printf("str type %T str=%q\n\n", str, str) //str type string str="23.340000"

// bool -> string
str = fmt.Sprintf("%t", isChecked)
fmt.Printf("str type %T str=%q\n\n", str, str) //str type string str="true"

// char -> string
str = fmt.Sprintf("%c", myChar)
fmt.Printf("str type %T str=%q\n\n", str, str) //str type string str="h"
```



#### 方式二：使用`strconv`包的函数

```go
var (
    num1      int     = 99
    num2      float64 = 23.34
    isChecked bool    = true
    str       string  // 空字符串
)

// int -> string
str = strconv.FormatInt(int64(num1), 10)
fmt.Printf("str type %T str=%q\n\n", str, str) //str type string str="99"
str = strconv.Itoa(num1)
fmt.Printf("str type %T str=%q\n\n", str, str) //str type string str="99"

// float -> string
// 参数说明：'f'：格式 10：表示小数位保留10位 64：表示float64
str = strconv.FormatFloat(num2, 'f', 10, 64)
fmt.Printf("str type %T str=%q\n\n", str, str) //str type string str="23.3400000000"

// bool -> string
str = strconv.FormatBool(isChecked)
fmt.Printf("str type %T str=%q\n\n", str, str) //str type string str="true"

```

### string类型转基本数据类型

#### 使用`strconv`包的函数

```go
var str string = "true"
var b bool
b, _ = strconv.ParseBool(str)
fmt.Printf("b type %T  b=%v\n", b, b) //b type bool  b=true

var str1 string = "10"
var b1 int64
b1, _ = strconv.ParseInt(str1, 10, 64)
fmt.Printf("b1 type %T  b1=%v\n", b1, b1) //b1 type int64  b1=10

var str2 string = "10.0000001"
var b2 float64
b2, _ = strconv.ParseFloat(str2, 64)
fmt.Printf("b2 type %T  b2=%v\n", b2, b2) //b2 type float64  b2=10.0000001

var str3 string = "10"
var b3 uint64
b3, _ = strconv.ParseUint(str3, 10, 64)
fmt.Printf("b3 type %T  b3=%v\n", b3, b3)
```

### 注意事项

在将string转成基本数据类型时，要确保string类型能够转成有效的数据，否则会将其转成零值

```go
var str4 string = "hello" // 把此字符串转成整数，Go会直接将其转成0
var num int64
num, _ = strconv.ParseInt(str4, 10, 64)
fmt.Printf("num type %T  num=%v\n", num, num) // num type int64  num=0
```


