# Go指针


## 基本介绍

1. 基本数据类型，变量存的就是值，也叫值类型

2. 获取变量的地址，用`&`

    ```go
    var i int = 10
    fmt.Println("i的地址：", &i) //i的地址： 0xc0000162d8
    ```

3. 指针类型，指针变量存的是一个地址，这个地址指向的空间存的才是值

    ```go
    // ptr是一个指针变量
    // ptr的类型 *int
    // ptr本身的值是&i
    var ptr *int = &i
    fmt.Printf("ptr=%v\n", ptr)     //ptr=0xc00011c1b0
    fmt.Printf("ptr的地址：%v\n", &ptr) //ptr的地址：0xc000114028
    ```

4. 获取指针类型所指向的值，用`*`

    ```go
    fmt.Printf("ptr指向的值=%v\n", *ptr) //ptr指向的值=10
    ```

## **变量在内存中的分布：**

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206301706825.png)

## **指针在内存中的分布：**

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206301707954.png)

## 指针细节说明

1. 值类型，都有对应的指针类型，形式为`*数据类型`,比如：`int`对应的指针就是`*int`，`float32`对应的指针类型就是`*float`
2. 值类型包括：基本数据类型int系列、float系列、bool、string、数组和结构体struct

