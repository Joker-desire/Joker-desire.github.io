# Go数组和切片


## 数组

数组可以存放多个同一类型数据。数据也是一种数据类型，在Go中，数组是值类型。

### 数组的定义

语法：`var 数组名 [数组大小]数据类型`

注意：定义完数组后，其实数组的各个元素都是有默认值的，默认值为数据类型的零值

```go
var nums [6]int
```

1. 数组的地址可以通过数据名来获取`&nums`
2. 数组的第一个元素的地址，就是数组的首地址
3. 数组的各个元素的地址间隔是依据数组的类型决定的，比如`int64->8` `int32->4`……

```go
var nums [6]int
fmt.Printf("nums内存地址：%p \n", &nums)       //nums内存地址：0xc000018360
fmt.Printf("nums[0]内存地址：%p \n", &nums[0]) //nums内存地址：0xc000018360
fmt.Printf("nums[1]内存地址：%p \n", &nums[1]) //nums内存地址：0xc000018368
fmt.Printf("nums[2]内存地址：%p \n", &nums[2]) //nums内存地址：0xc000018370
fmt.Printf("nums[3]内存地址：%p \n", &nums[3]) //nums内存地址：0xc000018378
```

### 数组的使用

#### 访问数组元素

语法：`数组名[下标]`

```go
var nums [6]int
fmt.Println(nums[0]) //0
```

#### 初始化数组

```go
var numsArray01 [3]int = [3]int{1, 2, 3}
var numsArray02 = [3]int{4, 5, 6}
var numsArray03 = [...]int{7, 8, 9}
// 可以指定元素值对应的下标
var names = [3]string{1: "tom", 0: "tommy", 2: "loss"}
fmt.Println("numsArray01：", numsArray01) //numsArray01： [1 2 3]
fmt.Println("numsArray02：", numsArray02) //numsArray02： [4 5 6]
fmt.Println("numsArray03：", numsArray03) //numsArray03： [7 8 9]
fmt.Println("names：", names)             //names： [tommy tom loss]
```

#### 数组遍历

##### 常规遍历

```go
names := [3]string{1: "tom", 0: "tommy", 2: "loss"}
for i := 0; i < len(names); i++ {
    fmt.Printf("第%d个元素：%s\n", i, names[i])
}
/*
	遍历结果：
		第0个元素：tommy
		第1个元素：tom
		第2个元素：loss
*/
```

##### for-range结构遍历

```go
for index, value := range names {
    fmt.Printf("第%d个元素：%s\n", index, value)
}
```

- 第一个返回值`index`是数组的下标
- 第二个`value`是在该下标位置的值
- `index`和`value` 是仅在for循环内部可见的局部变量
- 遍历数组元素的时候，如果不需要下标`index`，可以使用`_`替代
- `index`和`value` 名称不是固定的，可以自行指定

### 数组的注意事项

1. 数组是多个相同类型数据的组合，一个数组一旦声明/定义，其长度是固定的，不能动态变化。
2. 数组中的元素可以是任意数据类型，包括值类型和引用类型，但是不能混用。
3. 数组创建后，如果没有赋值，有默认值，默认值为数据类型的零值。
4. 数组的下标是从0开始的，下标必须在指定范围内使用。
5. Go的数据属于值类型，在默认情况下是值传递，因此会进行值拷贝。数据间不会相互影响。
6. 如果想在其他函数中，修改原来的数组，可以使用引用传递（指针方式）。
7. 长度是数组类型的一部分，在传递函数参数时，需要考虑数组的长度。



## 切片（slice）

### 基本介绍

1. 切片是数组的一个引用，因此切片是引用类型，在进行传递时，遵守引用传递的机制。
2. 切片的使用和数组类似，遍历切片、访问切片的元素和求切片长度`len(slice)`都一样。
3. 切片的长度是可以变化的，因此切片是一个可以动态变化的数组。

### 切片定义

语法：`var 切片名 []类型`

```go
var a []int
```

### 切片在内存中的分布

```go
intArr := [...]int{1, 2, 3, 4, 5, 6}
// 声明一个切片
// 引用intArr数组的起始下标为1，最后下标为3（不包含3）的值
slice := intArr[1:3]
fmt.Println("intArr=", intArr)          //intArr= [1 2 3 4 5 6]
fmt.Println("slice 的元素是：", slice)       //slice 的元素是： [2 3]
fmt.Println("slice 的元素个数：", len(slice)) //slice 的元素个数： 2
fmt.Println("slice 的容量：", cap(slice))   //slice 的容量： 5

fmt.Printf("intArr[0] 的内存地址: %p\n", &intArr[0]) //intArr[0] 的内存地址: 0xc000018360
fmt.Printf("intArr[1] 的内存地址: %p\n", &intArr[1]) //intArr[1] 的内存地址: 0xc000018368
fmt.Printf("intArr[2] 的内存地址: %p\n", &intArr[2]) //intArr[2] 的内存地址: 0xc000018370
fmt.Printf("intArr[3] 的内存地址: %p\n", &intArr[3]) //intArr[3] 的内存地址: 0xc000018378
fmt.Printf("intArr[4] 的内存地址: %p\n", &intArr[4]) //intArr[4] 的内存地址: 0xc000018380
fmt.Printf("intArr[5] 的内存地址: %p\n", &intArr[5]) //intArr[5] 的内存地址: 0xc000018388

fmt.Printf("slice[0] 的内存地址: %p\n", &slice[0]) //slice[0] 的内存地址: 0xc0000aa068
fmt.Printf("slice[1] 的内存地址: %p\n", &slice[1]) //slice[1] 的内存地址:  0xc000018370
```

![image-20220727141605105](https://raw.githubusercontent.com/XD825/picgo/main/img/202207271416141.png)

### 切片使用

#### 方式一

定义一个切片，然后让切片去引用一个已经创建好的数组

```go
intArr := [...]int{1, 2, 3, 4, 5, 6}
// 声明一个切片
// 引用intArr数组的起始下标为1，最后下标为3（不包含3）的值
slice := intArr[1:3]
```

#### 方式二

通过`make`来创建切片

基本语法：`var 切片名 []type = make([]type, len, [cap])`

- type: 是数据类型
- len: 大小
- cap: 指定切片容量，可选

通过make方式创建的切片对应的数组是由make底层维护，对外不可见，即只能通过slice去访问各个元素

```go
var slice2 = make([]int, 4, 10)
fmt.Println("slice2=  ", slice2)        //slice2=   [0 0 0 0]
fmt.Println("slice2 的大小：", len(slice2)) //slice2 的大小： 4
fmt.Println("slice2 的容量：", cap(slice2)) //slice2 的容量： 10
```

![image-20220727112421907](https://raw.githubusercontent.com/XD825/picgo/main/img/202207271124942.png)

#### 方式三

定义一个切片，直接就指定具体数组，使用原理类似make的方式

```go
var slice3 = []int{1, 3, 5}
fmt.Println("slice3=  ", slice3)        //slice3=   [1 3 5]
fmt.Println("slice3 的大小：", len(slice3)) //slice3 的大小： 3
fmt.Println("slice3 的容量：", cap(slice3)) //slice3 的容量： 3
```

#### 方式一、二的区别

- 方式一是直接引用数组，这个数组是事先存在的，是可见的
- 方式二通过make来创建切片，make也会创建一个数组，是由切片在底层进行维护，是不可见的

### 切片遍历

#### 常规方式遍历

```go
slice4 := []int{1, 2, 3}
for i := 0; i < len(slice4); i++ {
    fmt.Printf("第%d个元素：%d\n", i, slice4[i])
}
```

#### for-range遍历

```go
for i, v := range slice4 {
    fmt.Printf("第%d个元素：%d\n", i, v)
}
```

### 切片动态追加元素

用append内置函数，可以对切片进行动态追加

```go
slice5 := []int{1, 2, 3}
fmt.Println("slice5=  ", slice5)        //slice5=   [1 2 3]
fmt.Println("slice5 的大小：", len(slice5)) //slice5 的大小： 3
fmt.Println("slice5 的容量：", cap(slice5)) //slice5 的容量： 3

slice5 = append(slice5, 10, 20, 30)
fmt.Println("slice5=  ", slice5)        //slice5=   [1 2 3 10 20 30]
fmt.Println("slice5 的大小：", len(slice5)) //slice5 的大小： 6
fmt.Println("slice5 的容量：", cap(slice5)) //slice5 的容量： 6

// 在切片上追加切片
slice6 := []int{1000, 2000}
slice5 = append(slice5, slice6...)
fmt.Println("slice5=  ", slice5)        //slice5=   [1 2 3 10 20 30 1000 2000]
fmt.Println("slice5 的大小：", len(slice5)) //slice5 的大小： 8
fmt.Println("slice5 的容量：", cap(slice5)) //slice5 的容量： 12
```

切片append操作的底层原理分析：

- 切片append操作的本质就是对数组扩容
- Go底层会创建一个新的数组newArr（安装扩容后的大小）
- 将slice原来包含的元素拷贝到新的数组newArr
- slice重新引用到newArr（newArr是由底层来维护的，不可见）

### 切片的拷贝操作

切片使用copy内置函数完成拷贝

```go
slice7 := []int{1, 2, 3, 4, 5}
slice8 := make([]int, 10)
fmt.Println("slice7 = ", slice7) //slice7 =  [1 2 3 4 5]
fmt.Println("slice8 = ", slice8) //slice8 =  [0 0 0 0 0 0 0 0 0 0]
copy(slice8, slice7)
fmt.Println("拷贝后的slice8 = ", slice8) //拷贝后的slice8 =  [1 2 3 4 5 0 0 0 0 0]
```

### 切片的注意事项

- 切片初始化时：`var slice = arr[startIndex:endIndex]`
    - 说明：从arr数组下标为startIndex，取到下标为endIndex的元素（不含arr[endIndex]）
- 切片初始化时，不能越界，范围在[0~len(arr)]之间，但是可以动态增长
- cap是一个内置函数，用户统计切片的容量，即最大可以存放多少个元素
- 切片定义完后，还不能使用，因为本身是一个空的，需要让其引用到一个数组，或者make一个空间供切片来使用
- 切片可以继续切片
- 用append内置函数，可以对切片进行动态追加
- 切片使用copy内置函数完成拷贝
- 切片是引用类型，所在在传递时，遵守引用传递机制

## string和slice

1. string底层是一个byte数组，因此string也可以进行切片处理

    ```go
    s := "Hello World"
    fmt.Println("s = ", s)         //s =  Hello World
    fmt.Println("s[4:] = ", s[4:]) //s[4:] =  o World
    ```

2. string和切片在内存的形式

    ![image-20220727120158171](https://raw.githubusercontent.com/XD825/picgo/main/img/202207271201241.png)

3. string是不可变的，不能通过`str[0]='z'`方式来修改字符串

4. 如果要修改字符串，可以现将`string`转成`[]byte`或者`[]runne`，然后再进行修改，最后再转成`string`

    ```go
    s := "Hello World"
    
    fmt.Println("原始s = ", s) //原始s =  Hello World
    arr := []byte(s)
    arr[0] = 'z'
    s = string(arr)
    fmt.Println("更改后的s = ", s) //更改后的s =  zello World
    
    // 细节：转成[]byte后，可以处理英文和数字，但是不能处理中文
    // 原因：[]byte是按照字节来处理的，而一个中文，是3个字节，因此会出现乱码
    
    //arr2 := []byte(s)
    //arr2[0] = '中' // 编译报错：constant 20013 overflows byte
    //s = string(arr2)
    //fmt.Println("更改后的s = ", s)
    
    // 解决：将string转成[]rune即可，因为[]rune是按字符处理，兼容汉字
    arr3 := []rune(s)
    arr3[0] = '中' // 编译报错：constant 20013 overflows byte
    s = string(arr3)
    fmt.Println("更改后的s = ", s) //更改后的s =  中ello World
    ```

    

