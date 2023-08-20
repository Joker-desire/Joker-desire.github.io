# Go文件操作


# 基本介绍

文件再程序中是以**流**的形式来操作的。

![image-20230524110947948](https://raw.githubusercontent.com/XD825/picgo/main/img/202305241109115.png)

- **流**：数据再数据源（文件）和程序（内存）之间经历的路径

- **输入流**：数据从数据源（文件）到程序（内存）的路径

- **输出流**：数据从程序（内存）到数据源（文件）的路径

`os.File`封装了所有文件相关的操作，`File`是一个结构体。

[文档](https://pkg.go.dev/os#File)

![image-20230524111642371](https://raw.githubusercontent.com/XD825/picgo/main/img/202305241116462.png)

# 文件操作

## 文件打开和关闭

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305241224087.png" alt="image-20230524122425940" style="zoom:50%;" />

## 读文件操作应用实例

**1、读取文件的内容使用带缓冲区的方式，使用`os.Open`, 	`file.Close`, `bufio.NewReader()`, `reader.ReadString`函数和方法**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305241237793.png" alt="image-20230524123743715" style="zoom:50%;" />

**2、读取文件的内容使用`ioutil`一次将整个文件读入到内存中，这种方式适用于文件不大的情况。相关方法和函数([`ioutil.ReadFile`](https://pkg.go.dev/io/ioutil#ReadFile))**

<img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305241249288.png" alt="image-20230524124903190" style="zoom:50%;" />

## 写文件操作应用实例

### `os.OpenFile`写文件

**`func OpenFile(name string , flag int , perm FileMode ) (* File , error )`**

**说明：**

`os.OpenFile`是一个更一般性的文件打开函数，它会使用指定的选项、指定的模式打开指定名称的文件。如果操作成功，返回文件对象可用于I/O。如果出错，错误底层类型是`*PathError`。

**参数说明：**

- 第一个参数：文件路径

- 第二个参数：打开文件的[选项](https://pkg.go.dev/os#pkg-constants)：

    <img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305241543191.png" alt="image-20230524154319025" style="zoom:50%;" />

- 第三个参数：[权限控制](https://pkg.go.dev/io/fs#FileMode)

    <img src="https://raw.githubusercontent.com/XD825/picgo/main/img/202305241549512.png" alt="image-20230524154946418" style="zoom:50%;" />

#### **实例：**

##### 1、创建新文件并写入内容

**`os.O_CREATE|os.O_WRONLY`(创建|只写)**

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {

	// 创建新文件，写入内容
	fileName := "go-basic/go-file/writer_file/test.txt"
	file, err := os.OpenFile(fileName, os.O_CREATE|os.O_WRONLY, 0666)
	if err != nil {
		fmt.Println("open file err=", err)
		return
	}
	defer file.Close()

	// 写入内容
	str := "Hello Golang\n"
	// 使用带缓存的写入
	writer := bufio.NewWriter(file)
	for i := 0; i < 5; i++ {
		writer.WriteString(str)
	}
	//因为写入是带缓存的，所以需要调用Flush方法，将缓存的数据真正写入到文件中
	writer.Flush()
}
```

##### 2、打开存在的文件，将原来的内容进行覆盖

**`os.O_TRUNC|os.O_WRONLY`(覆盖|只写)**

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {

	// 打开存在的文件，将原来的内容进行覆盖
	fileName := "go-basic/go-file/writer_file/test.txt"
	file, err := os.OpenFile(fileName, os.O_TRUNC|os.O_WRONLY, 0666)
	if err != nil {
		fmt.Println("open file err=", err)
		return
	}
	defer file.Close()

	// 写入内容
	str := "Hello New Golang\n"
	// 使用带缓存的写入
	writer := bufio.NewWriter(file)
	for i := 0; i < 10; i++ {
		writer.WriteString(str)
	}
	//因为写入是带缓存的，所以需要调用Flush方法，将缓存的数据真正写入到文件中
	writer.Flush()
}
```

##### 3、打开存在的文件，在原来的内容中追加内容

 **`os.O_APPEND|os.O_WRONLY`(追加|只写)**

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {

	// 打开存在的文件，在原来的内容中追加内容
	fileName := "go-basic/go-file/writer_file/test.txt"
	file, err := os.OpenFile(fileName, os.O_APPEND|os.O_WRONLY, 0666)
	if err != nil {
		fmt.Println("open file err=", err)
		return
	}
	defer file.Close()

	// 写入内容
	str := "APPEND!\n"
	// 使用带缓存的写入
	writer := bufio.NewWriter(file)
	writer.WriteString(str)
	//因为写入是带缓存的，所以需要调用Flush方法，将缓存的数据真正写入到文件中
	writer.Flush()
}
```

##### 4、打开存在的文件，将原来内容输出在终端，并且追加内容

 **`os.O_RDWR|os.O_APPEND`（只读|追加）**

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

func main() {

	// 打开存在的文件，将原来内容输出在终端，并且追加内容
	fileName := "go-basic/go-file/writer_file/test.txt"
	file, err := os.OpenFile(fileName, os.O_RDWR|os.O_APPEND, 0666)
	if err != nil {
		fmt.Println("open file err=", err)
		return
	}
	defer file.Close()

	// 读内容原内容
	reader := bufio.NewReader(file)
	for {
		readString, err := reader.ReadString('\n')
		if err == io.EOF {
			break
		}
		fmt.Print(readString)
	}
	// 写入内容
	str := "APPEND!\n"
	// 使用带缓存的写入
	writer := bufio.NewWriter(file)
	writer.WriteString(str)
	//因为写入是带缓存的，所以需要调用Flush方法，将缓存的数据真正写入到文件中
	writer.Flush()
}
```

### `ioutil.WriterFile`写文件

`func WriteFile(filename string, data []byte, perm fs.FileMode) error`

#### **实例：**

```go
package main

import (
	"fmt"
	"io/ioutil"
)

func main() {
	fileName := "go-basic/go-file/writer_file2/test.txt"
	fileName2 := "go-basic/go-file/writer_file2/test2.txt"

	data, err := ioutil.ReadFile(fileName)
	if err != nil {
		fmt.Println("read file err=", err)
		return
	}

	// 将test.txt文件内容写入到test2.txt文件中（如果文件不存在则新建）
	err = ioutil.WriteFile(fileName2, data, 0666)
	if err != nil {
		fmt.Println("write file err=", err)
		return
	}
}

```

## 判断文件或文件夹是否存在

`func (f *File) Stat() (FileInfo, error)`

1. 如果返回的错误为`nil`，说明文件或文件夹存在
2. 如果返回的错误类型使用`os.IsNotExist()`判断为`true`，说明文件或文件夹不存在
3. 如果返回的错误为其他类型，则不确定是否存在

```go
func PathIsExist(path string) bool {
	_, err := os.Stat(path)
	if err == nil {
		return true
	}
	if os.IsExist(err) {
		return true
	}
	return false
}
```

## 拷贝文件

`func Copy(dst Writer, src Reader) (written int64, err error)`

**实例：**

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

func FileCopy(dstFileName string, srcFileName string) (written int64, err error) {
	// 打开需要拷贝的文件
	srcFile, err := os.Open(srcFileName)
	if err != nil {
		return 0, err
	}
	// 创建一个reader
	reader := bufio.NewReader(srcFile)

	// 打开需要写入的文件
	dstFile, err := os.OpenFile(dstFileName, os.O_WRONLY|os.O_CREATE, 0666)
	if err != nil {
		return 0, err
	}
	// 创建一个writer
	writer := bufio.NewWriter(dstFile)

	// 关闭文件
	defer func() {
		srcFile.Close()
		dstFile.Close()
	}()
	// 使用io.Copy开始拷贝
	return io.Copy(writer, reader)
}

func main() {
	// 将6.JPG文件拷贝到6_copy.JPG
	srcFile := "6.JPG"
	dstFile := "6_copy.JPG"
	fileCopy, err := FileCopy(dstFile, srcFile)
	if err != nil {
		return
	}
	fmt.Println("拷贝成功，共拷贝了", fileCopy, "字节")
}
```

## 统计不同类型的字符数量

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

type CharCount struct {
	ChCount    int // 记录英文个数
	NumCount   int // 记录数字个数
	SpaceCount int // 记录空格个数
	OtherCount int // 记录其他字符个数
}

func main() {
	// 实例化记录结构体
	count := CharCount{}
	// 打开文件
	fileName := "go-basic/go-file/file_count/test.txt"
	file, err := os.Open(fileName)
	if err != nil {
		fmt.Println("open file err=", err)
		return
	}
	defer file.Close()
	//创建一个reader
	reader := bufio.NewReader(file)
	// 每读取一行，就去统计该行有多少个英文、数字、空格以及其他字符
	for {
		readString, err := reader.ReadString('\n')
		if err == io.EOF {
			break
		}
		// 遍历readString，进行统计
		for _, v := range readString {
			switch {
			case v >= 'a' && v <= 'z':
				fallthrough // 穿透
			case v >= 'A' && v <= 'Z':
				count.ChCount++
			case v == ' ' || v == '\t':
				count.SpaceCount++
			case v >= '0' && v <= '9':
				count.NumCount++
			default:
				count.OtherCount++
			}
		}

	}
	// 然后将结果保存在一个结构体中
	fmt.Println("字符的个数为：", count.ChCount)
	fmt.Println("数字的个数为：", count.NumCount)
	fmt.Println("空格的个数为：", count.SpaceCount)
	fmt.Println("其他字符的个数为：", count.OtherCount)

}
```


