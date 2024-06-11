# Go 命令行参数


# Args解析命令行参数

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305241837301.png" alt="image-20230524183704157" style="zoom:50%;" />



# `flag`解析命令行参数

```go
package main

import (
	"flag"
	"fmt"
	"time"
)

//flag库是Go语言标准库之一，提供了命令行参数解析的能力。
func main() {
	b := flag.Bool("testBool", false, "testBool is bool byte.") // go run main.go -testBool=false
	port := flag.Int("port", 8081, "服务端口号")                     // go run main.go --port 8080

	var (
		boolValue     bool
		intValue      int
		int64Value    int64
		uintValue     uint
		uint64Value   uint64
		stringValue   string
		float64Value  float64
		durationValue time.Duration
	)

	flag.BoolVar(&boolValue, "bool", false, "This is bool value.")
	flag.IntVar(&intValue, "int", 0, "This is int value.")
	flag.Int64Var(&int64Value, "int64", 0, "This is int64 value.")
	flag.UintVar(&uintValue, "uint", 0, "This is uint value.")
	flag.Uint64Var(&uint64Value, "uint64", 0, "This is uint64 value.")
	flag.StringVar(&stringValue, "string", "", "This is string value.")
	flag.Float64Var(&float64Value, "float64", 0, "This is float64 value")
	flag.DurationVar(&durationValue, "duration", time.Second*0, "This is duration value.")

	flag.Parse()
	fmt.Println(*b, *port)
}

```

![image-20230524184303268](https://raw.githubusercontent.com/XD825/picgo/main/img/202305241843355.png)

