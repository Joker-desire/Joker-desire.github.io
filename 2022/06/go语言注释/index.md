# Go语言注释


Go支持C语言风格的`/* */`块注释，也支持C++风格的`//`行注释。

行注释更通用，块注释要勇于针对包的详细说明或者屏蔽大块的代码。

### 1）行注释

格式：`//注释文字`

示例：

```go
fmt.Println("Hello World!")//输出：Hello World!
```

### 2）块注释（多行注释）

格式：`/* 注释文字 */`

示例：

```go
/*
	Multipart/Urlencoded 表单
	两个参数都传：
		curl -v --form message=msg --form nick=nick2 http://localhost:8080/v1/form_post
		结果：{"message":"msg","nick":"nick2","status":"posted"}
	只传一个参数，另一个使用默认值：
		curl -v --form message=msg http://localhost:8080/v1/form_post
		结果：{"message":"msg","nick":"anonymous","status":"posted"}
*/
v1.POST("/form_post", fast_started.Form_post)
```

### 使用细节

1. 对于行注释和块注释，被注释的文字，不会被Go编译器解释执行
2. 块注释里面不允许有块注释嵌套
