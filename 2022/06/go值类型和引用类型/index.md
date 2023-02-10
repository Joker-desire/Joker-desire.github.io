# Go值类型和引用类型


## 说明

- 值类型：基本数据类型 int系列、float系列、bool、string、数组和结构体struct

- 引用类型：指针、slice切片、map、管道chan、interface等都是引用类型

## 特点

- **值类型**：变量直接存储值，内存通常在栈中分配

    ![image-20220630182227241](https://raw.githubusercontent.com/XD825/picgo/main/img/202206301822281.png)

- **引用类型**：变量存储的是一个地址，这个地址对应的空间才真正存储数据（值）。内存通常在对上分配，当没有任何变量引用这个地址时，该地址对应的数据空间就成为一个垃圾，由GC来回收。

![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206301822753.png)

- 内存的栈区和堆区示意图：

    ![](https://raw.githubusercontent.com/XD825/picgo/main/img/202206301823678.png)

