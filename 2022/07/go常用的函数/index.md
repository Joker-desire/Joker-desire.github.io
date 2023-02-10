# Go常用的函数


[Go语言标准库文档中文版](https://studygolang.com/pkgdoc)

## 字符串中常用的函数

![image-20220714091103183](https://raw.githubusercontent.com/XD825/picgo/main/img/202207140911256.png)


> 统计字符串的长度，按字节：`len(str)`

```go
//统计字符串的长度，按字节
s1 := "Hello World"
t.Logf("s1的长度为：%v", len(s1)) //s1的长度为：11
s2 := "Hello 小明" // golang的编码统一为utf-8(ASCII的字符（字母和数字）占一个字节，汉字占3个字节)
t.Logf("s2的长度为：%v", len(s2)) // s2的长度为：12
```



> 字符串遍历，同时处理有中文的问题：`r := []rune(str)`

```go
//字符串遍历，同时处理有中文的问题
s3 := "Hello World！"
for _,v := range []rune(s3) {
    t.Log(string(v))
}
/* 打印结果
function_test.go:15: H
function_test.go:15: e
function_test.go:15: l
function_test.go:15: l
function_test.go:15: o
function_test.go:15:  
function_test.go:15: W
function_test.go:15: o
function_test.go:15: r
function_test.go:15: l
function_test.go:15: d
function_test.go:15: ！
*/
```




> 字符串转整数：`i := strconv.Atoi("12")`

```go
// 字符串转整数
i, _ := strconv.Atoi("12")
t.Logf("i:的值为：%v, 类型为%T", i, i) //i:的值为：12, 类型为int
```



> 整数转字符串：`str = strconv.Itoa(12345)`

```go
//整数转字符串
s4 := strconv.Itoa(12345)
t.Logf("s4的值为：%v, 类型为%T", s4, s4) //s4的值为：12345, 类型为string
```



> 字符串转[]byte：`var bytes = []byte("hello go")`

```go
// 字符串转[]byte
var bytes = []byte("Hello go")
t.Logf("bytes的值为：%v, 类型为%T", bytes, bytes) //bytes的值为：[72 101 108 108 111 32 103 111], 类型为[]uint8
```



> []byte转字符串：`str := string([]byte{97, 98, 99})`

```go
//[]byte转字符串
s5 := string([]byte{72, 101, 108, 108, 111, 32, 103, 111})
t.Logf("s5的值为：%v, 类型为%T", s5, s5) //s5的值为：Hello go, 类型为string
```



> 10进制转2,8,16进制：`str := strconv.FormatInt(123, 2)`

```go
// 10进制转2,8,16进制
s6 := strconv.FormatInt(123, 2)
t.Logf("123转2进制：%v", s6) //123转2进制：1111011
s7 := strconv.FormatInt(123, 8)
t.Logf("123转8进制：%v", s7) //123转8进制：173
s8 := strconv.FormatInt(123, 16)
t.Logf("123转16进制：%v", s8) //123转16进制：7b
```



> 查找子串是否在指定的字符串中：`strings.Contains("Hello World!", "ll")`

```go
// 查找子串是否在指定的字符串中
res := strings.Contains("Hello World!", "ll")
t.Log(res) // true
```



> 统计一个字符串有几个指定的子串：`strings.Count("ceheese", "e")`

```go
//统计一个字符串有几个指定的子串
count := strings.Count("Hello World!", "l")
t.Log(count) //3
```



> 不区分大小写的字符串比较（==是区分大小写的）：`fmt.Println(strings.EqualFold("abc", "ABC"))`

```go
// 不区分大小写的字符串比较（==是区分大小写的）
fmt.Println(strings.EqualFold("abc", "ABC")) //true
```



> 返回子串在字符串第一次出现的index值，如果没有返回-1：`strings.index("NLT_abc", "abc")`

```go
//返回子串在字符串第一次出现的index值，如果没有返回-1
fmt.Println(strings.Index("NLT_abc", "abc")) //4
```



> 返回子串在字符串最后一次出现的index，如果没有返回-1：`strings.LastIndex("go golang", "go")`

```go
//返回子串在字符串最后一次出现的index，如果没有返回-1
fmt.Println(strings.LastIndex("go golang", "go")) //3
```



> 将指定的子串替换成另外一个子串：`strings.Replace("go go hell", "go", "golang", n)`
>
> n: 可以指定希望替换几个，如果n=-1表示全部替换

```go
//将指定的子串替换成另外一个子串
fmt.Println(strings.Replace("go go hell", "go", "golang", -1)) //golang golang hell
```



> 按照指定的某个字符，为分割标识，将一个字符串拆分成字符串数组：`strings.Split("hello, work, ok", ",")`

```go
//按照指定的某个字符，为分割标识，将一个字符串拆分成字符串数组
fmt.Println(strings.Split("hello, work, ok", ",")) //[hello  work  ok]
```



> 将字符串的字母进行大小写的转换：转小写：`strings.ToLower("Go")` 转大写： `strings.ToUpper("Go")`

```go
//将字符串的字母进行大小写的转换
fmt.Println(strings.ToLower("Go")) //go
fmt.Println(strings.ToUpper("Go")) //GO
```



> 将字符串左右两边的空格去掉：`strings.TrimSpace(" tn ")`

```go
//将字符串左右两边的空格去掉
fmt.Println(strings.TrimSpace(" tn ")) //tn
```



> 将字符串左右两边指定的字符去掉：`strings.Trim("! hello !", "!")`

```go
//将字符串左右两边指定的字符去掉
fmt.Println(strings.Trim("!hello!", "!")) //hello
```



> 将字符串左边指定的字符去掉：`strings.TrimLeft("! hello !", "!")`

```go
//将字符串左边指定的字符去掉
fmt.Println(strings.TrimLeft("!hello!", "!")) //hello!
```



> 将字符串右边指定的字符去掉：`strings.TrimRight("! hello !", "!")`

```go
//将字符串右边指定的字符去掉
fmt.Println(strings.TrimRight("!hello!", "!")) //!hello
```



> 判断字符串是否以指定的字符串开头：`strings.HasPrefix("ftp://192.168.0.0.1", "ftp")`

```go
//判断字符串是否以指定的字符串开头
fmt.Println(strings.HasPrefix("ftp://192.168.0.0.1", "ftp")) //true
```



> 判断字符串是否以指定的字符串结束：`strings.HasSuffix("NLT_ABC.png", ".png")`

```go
// 判断字符串是否以指定的字符串结束
fmt.Println(strings.HasSuffix("NLT_ABC.png", ".png")) //true
```

## 时间和日期相关函数

![image-20220714091135073](https://raw.githubusercontent.com/XD825/picgo/main/img/202207140911129.png)

> `time.Time类型，用于表示时间`

```go
now := time.Now()
fmt.Printf("type: %T，val = %v\n", now, now) //type: time.Time，val = 2022-07-14 17:00:12.930889 +0800 CST m=+0.001793251
```



> 获取当前时间的方法：`time.Now()`

```go
fmt.Println(time.Now()) //2022-07-14 17:01:01.243621 +0800 CST m=+0.002719584
```



> 获取其他的日期信息

```go
now := time.Now()

fmt.Println("当前年：", now.Year())     // 当前年： 2022
fmt.Println("当前月：", now.Month())    //当前月： July
fmt.Println("当前日：", now.Day())      //当前日： 14
fmt.Println("当前星期：", now.Weekday()) //当前星期： Thursday
fmt.Println("当前小时：", now.Hour())    //当前小时： 17
fmt.Println("当前分钟：", now.Minute())  //当前分钟： 4
fmt.Println("当前秒：", now.Second())   //当前秒： 58
```



> 格式化日期时间

方式一：使用`Printf` or `SPrintf`

日期字符串各个数字是固定的，必须这样写

```go
//当前年月日 2022-7-14 17:17:16 
fmt.Printf("当前年月日 %d-%d-%d %d:%d:%d \n", now.Year(), now.Month(), now.Day(), now.Hour(), now.Minute(), now.Second())

dateStr := fmt.Sprintf("当前年月日 %d-%d-%d %d:%d:%d \n", now.Year(), now.Month(), now.Day(), now.Hour(), now.Minute(), now.Second())
fmt.Println(dateStr) //当前年月日 2022-7-14 17:17:16
```

方式二：使用`Format`格式化

日期字符串各个数字可以自由的组合

```go
fmt.Println(now.Format("2006/01/02 15:04:05")) //2022/07/14 17:21:12
fmt.Println(now.Format("2006-01-02"))          //2022-07-14
fmt.Println(now.Format("20060102"))            //20220714
fmt.Println(now.Format("15:04:05"))            //17:21:12
```



> 内置的常量

```go
// time内置的时间常量
const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)

// 星期常量
const (
    Sunday Weekday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)

// 月份常量
const (
    January Month = 1 + iota
    February
    March
    April
    May
    June
    July
    August
    September
    October
    November
    December
)

```



> 休眠：`time.Sleep`

```go
time.Sleep(time.Millisecond * 1500) //休眠1.5s
```



> 获取当前Unix时间戳和UnixNano时间戳
>
> 作用：可以获取随机数字

```go
fmt.Printf("unix时间戳=%v unixnano时间戳=%v \n", now.Unix(), now.UnixNano()) //unix时间戳=1657790593 unixnano时间戳=1657790593798028000
```



## 内置函数

![image-20220714100501358](https://raw.githubusercontent.com/XD825/picgo/main/img/202207141005411.png)

1. `len`：用来求长度，比如string、array、slice、map、channel

2. `new`：用来分配内存，主要用来分配值类型，比如：int、float32、struct……返回的是指针

3. `make`：用来分配内存，主要用来分配引用类型，比如：chan、map、slice

