# Go错误处理机制


## 基本说明

- 默认情况下，当发生错误（panic），程序就会退出（崩溃）
- Go语言不支持传统的`try……catch……finally`
- Go中引入的处理方式为：`defer` `panic` `recover`
    - 处理流程：抛出panic异常 -->> 在defer中通过recover捕获异常 -->> 正常处理

## 使用

```go
func test() {
	// 使用defer + recover 捕获和处理异常
	defer func() {
		err := recover() // recover 内置函数，可以捕获到异常
		if err != nil {
			fmt.Println("err=", err)
		}
	}()
	num1 := 10
	num2 := 0

	res := num1 / num2
	fmt.Println("res=", res)
}

func main() {
	test()
	fmt.Printf("main()结束")

}
```

## 自定义错误

使用`errors.New`和`panic`内置函数

1. `errorsNew("错误说明")`，会返回一个`error`类型的值，表示一个错误
2. `panic`内置函数，接收一个`interface{}`类型的值（也就是任意值）作为参数。可以接受`error`类型的变量，输出错误信息，并退出程序

```go
package main

import (
	"errors"
	"fmt"
)

func readConf(name string) (err error) {
	if name == "config.ini" {
		return nil
	} else {
		return errors.New("文件名输入有误")
	}
}

func main() {
	err := readConf("config.json")
	if err != nil {
		panic(err) // 把异常抛出，如果有异常则会抛出异常并且终止程序
	}
	fmt.Println("main()") // 有异常抛出，则不会执行
}

```


