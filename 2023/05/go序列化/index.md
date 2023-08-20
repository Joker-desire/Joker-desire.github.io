# Go 序列化


# 序列化

> `func Marshal(v any) ([]byte, error)`
>
> 可传入任意参数进行序列化 【[官方文档](https://pkg.go.dev/encoding/json#Marshal)】

## 结构体序列化

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name  string   `json:"name"`
	Age   int      `json:"age"`
	Hobby []string `json:"hobby"`
}

user := User{
    Name:  "Joker",
    Age:   20,
    Hobby: []string{"跑步", "爬山", "游泳"},
}
// 调用json.Marshal进行序列化
marshal, err := json.Marshal(&user)
if err != nil {
    fmt.Println("Serialization err ", err)
}
fmt.Println("Struct to Serialization result: ", string(marshal))
```

## Map序列化

```go
package main

import (
	"encoding/json"
	"fmt"
)

var user map[string]any
// map使用要先make
user = make(map[string]any)

user["name"] = "Desire"
user["age"] = 19
user["hobby"] = []string{"篮球", "羽毛球"}

marshal, err := json.Marshal(user)
if err != nil {
    fmt.Println("Serialization err ", err)
}
fmt.Println("Map to Serialization result: ", string(marshal))
```

## Slice序列化

```go
package main

import (
	"encoding/json"
	"fmt"
)

var users []map[string]any
user1 := map[string]any{
    "name":  "Joker",
    "age":   18,
    "hobby": []string{"足球", "徒步"},
}
users = append(users, user1)
user2 := map[string]any{
    "name":  "Joker2",
    "age":   18,
    "hobby": []string{"足球", "徒步"},
}
users = append(users, user2)

marshal, err := json.Marshal(users)
if err != nil {
    fmt.Println("Serialization err ", err)
}
fmt.Println("Map to Serialization result: ", string(marshal))
```

# 反序列化

> `func Unmarshal(data []byte, v any) error`
>
> 参数1：反序列化数据（`[]byte`）,参数2：反序列化的对象 【[官方文档](https://pkg.go.dev/encoding/json#Unmarshal)】



## 反序列化成结构体

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name  string   `json:"name"`
	Age   int      `json:"age"`
	Hobby []string `json:"hobby"`
}

str := `{"name":"Joker","age":20,"hobby":["跑步","爬山","游泳"]}`
var user User
err := json.Unmarshal([]byte(str), &user)
if err != nil {
    fmt.Println("Deserialization err ", err)
}
fmt.Println(user)
```

## 反序列化成Map

```go
package main

import (
	"encoding/json"
	"fmt"
)

str := `{"age":19,"hobby":["篮球","羽毛球"],"name":"Desire"}`
var user map[string]any
err := json.Unmarshal([]byte(str), &user)
if err != nil {
    fmt.Println("Deserialization err ", err)
}
fmt.Println(user)
```

## 反序列化成Slice

```go
package main

import (
	"encoding/json"
	"fmt"
)

str := `[{"age":18,"hobby":["足球","徒步"],"name":"Joker"},{"age":18,"hobby":["足球","徒步"],"name":"Joker2"}]`
var users []map[string]any
err := json.Unmarshal([]byte(str), &users)
if err != nil {
    fmt.Println("Deserialization err ", err)
}
fmt.Println(users)
```


