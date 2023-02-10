# Go Map


# Map

map是key-value数据结构，又称为字段或者关联数组。

## 基本语法

语法：`var map变量名 map[keytype]valuetype`

```go
var a map[string]string
var a map[string]int
var a map[int]string
var a map[string]map[string]string
```

**key的类型**

可以是：bool、数字、string、指针、channel，还可以值包含前面几个类型的接口、结构体、数组，通常为int、string

slice、map和function不可以当做key的类型，因为这几个没法用`==`来判断

**value的类型**

和key基本一样，通常为：数字（整数，浮点数）、string、map、struct

```go
//声明一个map
var a map[string]string
// 在使用map前，需要先make，make的作用个就是给map分配数据空间
a = make(map[string]string, 10)
a["str1"] = "第一个字符串"
a["str2"] = "第二个字符串"
a["str3"] = "第三个字符串"
fmt.Println(a) //map[str1:第一个字符串 str2:第二个字符串 str3:第三个字符串]
```

注意：声明是不会分配内存的，初始化需要`make`分配内存后才能赋值和使用。

**注意事项**

1. map在使用前一定要make
2. map的key是不能重复，如果重复了，则以最后这个key-value为准
3. map的value是可以相同的
4. map的key-value是无序的

## map的使用

方式一：

```go
var cities map[string]string
fmt.Println(cities) //map[]
// 分配一个map空间
cities = make(map[string]string, 10)
fmt.Println(cities) //map[]
```

方式二：

```go
// 声明直接make
var cities = make(map[string]string)
fmt.Println(cities) //map[]
```

方式三：

```go
// 声明直接赋值
var cities map[string]string = map[string]string{
    "city1": "成都",
}
cities["city2"] = "深圳"
fmt.Println(cities) //map[city1:成都 city2:深圳]
```

## map的操作

### map的增加和更新

`map["key"] = value` 如果key不存在就是增加，存在则修改。

```go
cities := map[string]string{
    "city1": "成都",
}
// 添加的key不存在，则新增
cities["city2"] = "深圳"
fmt.Println(cities) //map[city1:成都 city2:深圳]
// 添加的key存在，则修改
cities["city1"] = "北京"
fmt.Println(cities) //map[city1:北京 city2:深圳]
```

### map的删除

`delete(map, "key")` delete是一个内置函数，如果key存在，则删除key-value，如果key不存在，不操作，也不会报错。

```go
// 删除的key存在
delete(cities, "city1")
//删除的key不存在
delete(cities, "city3")
fmt.Println(cities) //map[city2:深圳]
```

1. 如果要删除map的所有key，没有专门的方法一次删除，可以遍历一下key，逐个删除

    ```go
    cities := map[string]string{
        "city1": "成都",
        "city2": "北京",
        "city3": "深圳",
    }
    fmt.Println(cities) //map[city1:成都 city2:北京 city3:深圳]
    for key, _ := range cities {
        delete(cities, key)
    }
    fmt.Println(cities) //map[]
    ```

    

2. 或者`map = make(...)` make一个新的，让原来的成为垃圾，被gc回收

    ```go
    cities := map[string]string{
        "city1": "成都",
        "city2": "北京",
        "city3": "深圳",
    }
    fmt.Println(cities) //map[city1:成都 city2:北京 city3:深圳]
    cities = make(map[string]string)
    fmt.Println(cities) //map[]
    ```

### map查找

`cities["city1"]` 获取map的值时，返回结果有两个，第一个为值，第二个为是否存在，存在返回true，反之false，通过判断第二个值可得到是否存在该key

```go
cities := map[string]string{
    "city1": "成都",
    "city2": "北京",
    "city3": "深圳",
}
val, ok := cities["city4"]
if ok {
    fmt.Printf("有 city1 key 值为%v\n", val)
} else {
    fmt.Printf("没有 city1 key \n")
}
```

## map的遍历

map的遍历使用for-range的结果遍历

```go
cities := map[string]string{
    "city1": "成都",
    "city2": "北京",
    "city3": "深圳",
}
for key, value := range cities {
    fmt.Println(key, value)
}
	/*
		结果：
		   city2 北京
		   city3 深圳
		   city1 成都
	*/
```

## map的长度

使用`len(map)`

```go
cities := map[string]string{
    "city1": "成都",
    "city2": "北京",
    "city3": "深圳",
}
fmt.Println(len(cities)) //3
```

## map切片

### 基本介绍

切片的数据类型如果是map，则我们称为slice of map，map切片，这样使用map个数就可以动态变化了

### 使用

```go
// 1. 声明一个map切片
var monsters []map[string]string
monsters = make([]map[string]string, 2) //准备放两个信息
// 2. 添加第一个信息
if monsters[0] == nil {
    monsters[0] = make(map[string]string, 2)
    monsters[0]["name"] = "Ronin"
    monsters[0]["age"] = "18"
}
fmt.Println(monsters) //[map[age:18 name:Ronin] map[]]
// 添加第二个信息
if monsters[1] == nil {
    monsters[1] = make(map[string]string, 2)
    monsters[1]["name"] = "Tommy"
    monsters[1]["age"] = "20"
}
fmt.Println(monsters) //[map[age:18 name:Ronin] map[age:20 name:Tommy]]
// 使用append函数动态的增加map数据
// 1.先定义信息
monster := map[string]string{
    "name": "Desire",
    "age":  "25",
}
monsters = append(monsters, monster)
fmt.Println(monsters) //[map[age:18 name:Ronin] map[age:20 name:Tommy] map[age:25 name:Desire]]

```

## map排序

- 先将map的key放入到切片中
- 对切片进行排序
-  遍历切片，然后按照key来输出map的值

```go
// map的排序
map1 := make(map[int]int, 10)
map1[10] = 100
map1[2] = 40
map1[5] = 1
map1[3] = 10
map1[9] = 20
// 打印结果，可以看到每次打印的结果顺序都不一样
for k, _ := range map1 {
    fmt.Println(k)
}

// 如果按照map的key进行排序输出
// 1. 先将map的key放入到切片中
// 2. 对切片进行排序
// 3. 遍历切片，然后按照key来输出map的值
var keys []int
for k, _ := range map1 {
    keys = append(keys, k)
}
fmt.Println(keys)
// 排序
sort.Ints(keys)
fmt.Println(keys)
for _, k := range keys {
    fmt.Printf("map1[%v]=%v \n", k, map1[k])
}
```

## map使用细节

1. map是引用类型，遵守引用类型传递的机制，在一个函数接收map，修改后，会直接修改原来的map
2. map的容量达到后，再想map增加元素，会自动扩容，并不会发生panic，也就是说map能动态的增长键值对
3. map的value也经常使用struct类型，更适合管理复杂的数据

