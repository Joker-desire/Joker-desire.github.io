# Go变量和基本数据类型


## Go变量

### 概念

变量相当于内存中一个数据存储空间的表示。通过变量名可以访问到变量（值）。

### 变量使用的基本步骤

1. 声明变量（定义变量）
2. 赋值
3. 使用

```go
var name string             //声明变量
name = "Joker"              //赋值
fmt.Println("name: ", name) //使用
```

### 变量使用注意事项

1. 变量表示内存中的一个存储区域

2. 该区域有自己的名称（变量名）和类型（数据类型）

3. Go变量使用的三种方式
    1. 指定变量类型，声明后若不赋值，使用默认值

        ```go
        var i int
        var name string
        fmt.Println("i: ", i)       //i:  0
        fmt.Println("name: ", name) //name:
        ```

    2. 根据值自行判定变量类型（类型推导）

        ```go
        i := 100
        fmt.Printf("i: %d, i的类型：%T\n", i, i) //i: 100, i的类型：int
        ```

        省略`var`，注意：`:=` 左侧的变量不应该是已经声明过的，否则会导致编译错误

4. 多变量声明

    ```go
    var a, b, c, d int
    fmt.Println(a, b, c, d) //0 0 0 0
    
    a1, b1, c1, d1 := 1, 2, 3, 4
    fmt.Println(a1, b1, c1, d1) //1 2 3 4
    
    var a2, b2, c2, d2 = "a2", 2, "c2", 23.09
    fmt.Println(a2, b2, c2, d2) //a2 2 c2 23.09
    ```

5. 该区域的数据值可以在同一类型范围内不断变化

    ```go
    var name = "Joker"
    fmt.Println(name) //Joker
    name = "Tom"
    fmt.Println(name) //Tom
    name = "Sony"
    fmt.Println(name) //Sony
    // name = 10         //cannot use 10 (type untyped int) as type string in assignment
    ```

6. 变量在同一作用域（在一个函数或者在代码块）内不能重名

    ```go
    var name = "Joker"
    fmt.Println(name) //Joker
    // 变量不能再同一作用域下重名
    var name = "Tom"  //previous declaration
    ```

7. 变量=变量名+值+数据类型

8. Go的变量如果没有赋初始值，编译器会使用默认值，比如:int默认值是0，string默认值为空，float默认值也是0

    ```go
    var (
        name  string //默认值为空
        age   int    //默认值为0
        score float64//默认值为0.0
    )
    fmt.Printf("name: %s,age:%d,score:%f\n", name, age, score) //name:,age:0,score:0.000000
    ```

### 程序中`+`号的使用

1. 当左右两边都是数值型时，则做加法运算

    ```go
    var (
        n1 = 100
        n2 = 200
    )
    fmt.Println("n1 + n2 = ", n1+n2) //n1 + n2 =  300
    ```
2. 当左右两边都是字符串，则做字符串拼接

    ```go
    var (
        n3 = "Hello"
        n4 = "World!"
    )
    fmt.Println("n3 + n4 = ", n3+n4) //n3 + n4 =  HelloWorld!
    ```

    

## 变量的数据类型

每一种数据都定义了明确的数据类型，在内存中分配了不同大小的内存空间。

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206301415947.png)

### 整数类型

#### 基本介绍

用于存放整数值，比如`0,-1,233`等

#### 示例

```go
var a int = 8900
fmt.Println("a = ", a)
var b uint = 1
fmt.Println("b = ", b)
var c byte = 255
fmt.Println("c = ", c)
var d rune = 1000
fmt.Println("d = ", d)
```

#### 整型的类型

| 类型  | 有无符号 |             占用存储空间             |            数值范围            |             备注             |
| :---: | :------: | :----------------------------------: | :----------------------------: | :--------------------------: |
| int8  |    有    |                1字节                 |            -128~127            |                              |
| int16 |    有    |                2字节                 |          -2^15~2^15-1          |                              |
| int32 |    有    |                4字节                 |          -2^31~2^31-1          |                              |
| int64 |    有    |                8字节                 |          -2^63~2^63-1          |                              |
|  int  |    有    | 32位系统4个字节<br />64位系统8个字节 | -2^31~2^31-1<br />-2^63~2^63-1 |                              |
| uint  |    无    | 32位系统4个字节<br />64位系统8个字节 |     0~2^32-1<br />0~2^64-1     |                              |
| rune  |    有    |             与int32一样              |          -2^31~2^31-1          | 等价int32，表示一个Unicode码 |
| byte  |    无    |             与uint8等价              |             0~255              |   当要存储字符时，选用byte   |

#### 整型的使用细节

1. Go各整数类型分类：有符号和无符号，int uint的大小和系统有关
2. Go的整型默认声明为int型

    ```go
    num := 100
    fmt.Printf("num=%d, num的类型：%T\n", num, num) //num=100, num的类型：int
    ```
3. Go程序中整型变量在使用时，遵守保小不保大的原则，即：在保证程序正确运行下，尽量使用占用空间小的数据类型，比如：年龄
4. bit: 计算机中的最小存储单位。byte: 计算机中基本存储单元

### 浮点型

#### 基本介绍

浮点型就是用于存放小数的，比如：`1.2, 0.3, -1.293`

#### 示例

```go
var price float32 = 89.02
fmt.Println("price = ", price) //price =  89.02
var (
    num1 float32 = -0.00089
    num2 float64 = -7890976.09
)
fmt.Println("num1 = ", num1, "，num2 = ", num2) //num1 =  -0.00089 ，num2 =  -7.89097609e+06
```

#### 浮点类型的分类

|      类型      | 占用存储空间 |       数值范围       |
| :------------: | :----------: | :------------------: |
| 单精度 float32 |    4字节     |  -3.403E38~3.403E38  |
| 双精度 float64 |    8字节     | -1.798E308~1.798E308 |

**说明：**

1. 浮点型的存储分为三部分：符号位+指数位+尾数位，在存储过程中，精度会有丢失

    ```go
    var (
        n1 float32 = -123.0000901
        n2 float64 = -123.0000901
    )
    fmt.Println("n1 = ", n1, " n2 = ", n2) //n1 =  -123.00009  n2 =  -123.0000901
    ```

2. float64的精度比float32的要准确

3. 如果需要保存一个精度高的数，则应该选用float64

#### 浮点型的使用细节

1. Go浮点类型有固定的范围和字段长度，不受具体OS的影响

2. Go的浮点型默认声明为float64类型

    ```go
    num3 := 12.09
    fmt.Printf("num3的类型：%T\n", num3) //num3的类型：float64
    ```

3. 浮点型常量有两种表示形式

    1. 十进制形式：如：

        ```go
        num4 := 5.12
        num5 := .123                                   //0.123
        fmt.Println("num4 = ", num4, " num5 = ", num5) //num4 =  5.12  num5 =  0.123
        ```

    2. 科学计数法形式：如：

        ```go
        num6 := 5.12e2
        num7 := 5.12e-2
        fmt.Println("num6 = ", num6, " num7 = ", num7) //num6 =  512  num7 =  0.0512
        ```

4. 通常情况下，应该使用float64，因为它比float32更精准



### 字符类型（char）

#### 基本介绍

Go中没有专门的字符类型，如果要存储单个字符（字母），一般使用`byte`来保存。

字符串就是一串固定长度的字符连接起来的字符序列。Go的字符串是由单个字节连接起来的，也就是说对于传统的字符串是由字符组成的，而Go的字符串不同，它是由字节组成的（官方将string归属到基本数据类型中）。

#### 示例

```go
var (
    c1 byte = 'a'
    c2 byte = '0' //字符的0
)

// 当我们直接输出byte值，就是输出了对应的字符的码值
fmt.Println("c1 = ", c1) //c1 =  97
fmt.Println("c2 = ", c2) //c2 =  48
// 如果希望输出对应的字符，则需要使用格式化输出，%c
fmt.Printf("c1 = %c, c2 = %c\n", c1, c2) //c1 = a, c2 = 0

//var c3 byte = '北' //constant 21271 overflows byte
var c3 int = '北' // 由于上面一行定义的字符类型，对应码大于255，建议大于255的考虑使用int类型保存
fmt.Printf("c3 = %c, c3对应码值=%d\n", c3, c3) //c3 = 北, c3对应码值=21271
```

**说明：**

1. 如果我们保存的字符在ASCII表中，那么可以直接保存到byte
2. 如果我们保存的字符对应码大于255，这时我们可以考虑使用int类型保存
3. 如果我们需要按照字符的方式输出，这时我们需要格式化输出`fmt.Printf("%c 对应的码值：%d", c, c)`

#### 字符类型的使用细节

1. 字符常量是用单引号（''）括起来的单个字符。例如：

    ```go
    var c1 byte = 'a'
    var c2 byte = '中'
    var c3 byte = '9'
    ```

2. Go中允许使用转义字符`\`来表示将其后的字符转变为特殊字符型常量，例如:

    ```go
    var c3 char = '\n' //'\n'表示换行符
    ```

3. Go语言的字符使用UTF-8编码；

4. 在Go中，字符的本质是一个整数，直接输出时，是该字符对应的UTF-8编码的码值；

    ```go
    var (
        c1 byte = 'a'
        c2 byte = '0' //字符的0
    )
    
    // 当我们直接输出byte值，就是输出了对应的字符的码值
    fmt.Println("c1 = ", c1) //c1 =  97
    fmt.Println("c2 = ", c2) //c2 =  48
    ```

5. 可以直接给某个变量赋一个数字，然后按格式化输出时使用`%c`会输出该数字对应的unicode字符；

    ```go
    //可以直接给某个变量赋一个数字，然后按照格式化输出使用%c，会输出该数字对应的Unicode字符
    var c4 int = 22269
    fmt.Printf("c4=%c\n", c4) //c4=国
    ```

6. 字符类型是可以进行运算的，相当于一个整数，因为它都对应的有Unicode码。

    ```go
    //字符类型是可以进行运算的，相当于一个整数，运算时按照码值运行
    var n1 = 10 + 'a'        // 10 + 97
    fmt.Println("n1 = ", n1) // n1 =  107
    ```

    

#### 字符类型的本质

1. 字符型 存储到计算机中，需要将字符对应的码值（整数）找出来
    1. 存储：字符 -> 对应码值 -> 二进制 -> 存储
    2. 读取：二进制 -> 码值 -> 字符 -> 读取
2. 字符和码值的对应关系是通过字符编码表来决定的（是已经规定好的）
3. Go语言的编码都统一成了`utf-8`，再也没有编码的困扰了

### 布尔类型（bool）

#### 基本介绍

1. 布尔类型也叫bool类型，bool类型数据只允许取值true和false
2. bool类型占1个字节
3. bool类型适用于逻辑运算，一般用于程序流程控制

#### 示例

```go
var b = false
fmt.Println("b=", b)                       //b= false
fmt.Println("b的占用空间 = ", unsafe.Sizeof(b)) //b的占用空间 =  1
```

#### 布尔类型的使用细节

1. 不可以0或非0的整数替代false和true

### 字符串类型（string）

#### 基本介绍

字符串就是一串固定长度的字符连接起来的字符序列。Go的字符串是由单个字节连接起来的。Go语言的字符串的字节使用UTF-8编码标识Unicode文本

#### 示例

```go
var name string // 显示声明string变量
name = "Joker"
fmt.Println(name) //Joker

address := "深圳"//隐式声明string变量
fmt.Printf("address=%s type=%T", address, address) //address=深圳 type=string
```

#### 字符串类型的使用细节

1. Go语言的字符串的字节使用UTF-8编码标识Unicode文本；

2. 字符串一旦赋值了，字符串就不能修改了：在Go中字符串是不可变的

3. 字符串的两种表示形式

    1. 双引号，会识别转义字符
    2. 反引号，以字符串的原生形式输出，包括换行和特殊字符，可以实现防止攻击、输出源代码等效果

4. 字符串拼接方式

    ```go
    var str string
    str = "hello " + "world!"
    fmt.Println(str) //hello world!
    str += "ok"      //+=等价于 str = str + "ok"
    fmt.Println(str) //hello world!ok
    ```

5. 当一行字符串太长时，需要使用到多行字符串

    ```go
    str2 := "hello " + // 使用多行字符串，+号必须在结尾
    	"world!"
    fmt.Println(str2) //hello world! 
    ```

### 基本数据类型默认值

在Go中，数据类型都有一个默认值，当没有赋值时，就会保留默认值，在Go中默认值又叫零值。

| 数据类型 | 默认值 |
| -------- | ------ |
| 整型     | 0      |
| 浮点型   | 0      |
| 字符串   | ""     |
| 布尔类型 | false  |

```go
var (
    a         int
    b         float32
    c         float64
    isMarried bool
    name      string
)
// %v：表示按照变量的值输出
fmt.Printf("a=%d,b=%v,c=%v,isMarried=%v,name=%v\n", a, b, c, isMarried, name) 
//输出结果：a=0,b=0,c=0,isMarried=false,name=
```


