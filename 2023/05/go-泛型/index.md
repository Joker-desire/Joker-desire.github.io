# Go 泛型（1.18+）


# 泛型基本含义

在定义函数（结构等）时，可能会有多种类型传入，真正使用方法的时候才可以确定用的是什么类型，此时就可以用一个更加宽泛的类型（存在一定约束，只能在哪些类型的范围内使用）暂时占位。这个类型就叫**泛型**。

# 泛型定义

> 写法：`[泛型标识 泛型约束] ` 例如：`[T any] 、 [S int|string]`

## 泛型方法

```go
func Say[T any](msg T) T {
	return msg
}

func TestFunc(t *testing.T) {
	// 可以传递任意类型的参数
	fmt.Println(Say("hello"))
	fmt.Println(Say(1))
	// 指定类型
	fmt.Println(Say[int](1))
	fmt.Println(Say[string]("hello"))
	// 如果指定了类型，参数必须是指定的类型，否则报错：cannot use 1 (untyped int constant) as string value in argument
	//fmt.Println(Say[string](1))
}
```

## 泛型结构体

```go
// 泛型结构体
type User[T any] struct {
	Name T
}

// 泛型结构体的方法
func (t User[T]) GetName() T {
	return t.Name
}

func TestStruct(t *testing.T) {
	// 在创建结构体变量时，指定类型（必须）
	fmt.Println(User[string]{Name: "Joker"})
	fmt.Println(User[int]{Name: 111})
	// 如果没有指定类型，会报错：cannot use generic type User[T any] without instantiation
	//fmt.Println(User{Name: 111})
	// 如果指定了类型，参数必须是指定的类型，否则报错：cannot use "Joker" (untyped string constant) as int value in struct literal
	//fmt.Println(User[int]{Name: "Joker"})
	// 调用泛型结构体的方法
	fmt.Println(User[string]{Name: "Joker"}.GetName())
}
```

## 泛型Map

```go
// map的key必须是可比较的类型，所以这里使用了comparable
type MyMap[K comparable, V any] map[K]V

func TestMap(t *testing.T) {
	// 定义map时，指定类型
	var myMap MyMap[string, int]
	myMap = make(MyMap[string, int])
	myMap["Joker"] = 18
	fmt.Println(myMap)
}
```

## 泛型切片

```go

type mySlice[T any] []T

func TestSlice(t *testing.T) {
	// 定义slice时，指定类型
	var slice mySlice[int]
	slice = append(slice, 1, 3, 4, 5)
	// 如果写入的类型不是指定的类型，会报错：cannot use "Joker" (untyped string constant) as int value in argument to append
	//slice = append(slice, "Joker")
	fmt.Println(slice)

	// 定义slice时，如果指定的类型是any，可以写入任意类型的值
	var slice2 mySlice[any]
	slice2 = append(slice2, 1, 3, 4, 5, "Joker")
	fmt.Println(slice2)
}
```

## 自定义泛型约束

```go
// 定义一个接口
type z interface {
	GetValue() string
}

// 定义一个结构体，实现z接口
type MyStruct struct {
	Name string
}

func (s MyStruct) GetValue() string {
	return s.Name
}

// 定义一个结构体，没有实现z接口
type MyStruct2 struct {
}

// 定义一个函数，参数必须是实现了z接口的类型
func test[T z](s T) T {
	fmt.Println(s.GetValue())
	return s
}

func TestCustomize(t *testing.T) {
	myStruct := MyStruct{
		Name: "Joker",
	}
	// 只能MyStruct类型，MyStruct实现了z接口
	m := test[MyStruct](myStruct)
	fmt.Println(m.Name)

	user := MyStruct2{}
	// 如果传入的类型没有实现z接口，会报错：MyStruct2 does not implement z (missing GetValue method)
	u := test[MyStruct2](user)
	fmt.Println(u)
}
```

## `~`用法

> 将多个约束条件通过`~`前缀增加到自定义接口，以实现多类型约束
>
> `testType[T MyType]`等价于`testType2[T int | string | bool]`

```go

type MyType interface {
	~int | ~string | ~bool
}

func testType[T MyType](t T) {
	fmt.Println(t)
}

// 使用接口实现泛型约束~int | ~string | ~bool，等价于下面的int | string | bool
func testType2[T int | string | bool](t T) {
	fmt.Println(t)
}
func TestMyType(t *testing.T) {
	// 只能传入int、string、bool类型的参数
	testType[int](123)
	testType[string]("Joker")
	testType[bool](false)
	// 可以不传递类型，但是参数必须是int、string、bool类型
	testType(123)
	testType("Joker")
	testType(false)
	// 如果传入的类型不是指定的类型，会报错：float64 does not implement MyType
	//testType(12.8)

	// 只能传入int、string、bool类型的参数
	testType2[int](123)
	testType2[string]("Joker")
	testType2[bool](false)
	// 可以不传递类型，但是参数必须是int、string、bool类型
	testType2(123)
	testType2("Joker")
	testType2(false)
	// 如果传入的类型不是指定的类型，会报错：float64 does not implement int|string|bool
	//testType2(12.8)
}
```




