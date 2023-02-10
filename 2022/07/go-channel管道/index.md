# Go channel（管道）、任务和sync


## channel介绍

1. `channel`本质就是一个数据结构-队列
2. 数据是先进先出（FIFO）
3. 线程安全，多goroutine访问时，不需要加锁
4. `channel`是有类型的，一个string的`channel`只能存放string数据类型

## channel使用

### channel定义/声明

`var 变量名 chan 数据类型`

```go
var intChan chan int // intChan用于存放int类型
var mapChan chan map[int]string // mapChan用于存放map[int]string类型
var perChan chan Person // 用于存放结构体类型
var perChan2 chan *Person // 用于存放指针类型
var allChan chan interface{} // 用于存放任意数据类型
```

1. channel 是引用类型
2. channel 必须初始化才能写入数据，即make后才能使用
3. 管道是有类型的，intChan 只能写入整数

### channel初始化

**使用make进行初始化**

```go
var intChan chan int
intChan = make(chan int, 10)

fmt.Printf("intChan值：%v \n", intChan)     //intChan值：0xc0000b4000
fmt.Printf("intChan本身地址：%p \n", &intChan) //intChan本身地址：0xc0000ac018
```

### 向channel中写入数据

当给channel写入数据时，写入的数量不能超过其容量大小，超过则报错：`all goroutines are asleep - deadlock!`

```go
vat intChan chan int
intChan = make(chan int, 3)
num:=99
intChan <- 10
intChan <- num

// 管道的长度和cap（容量）
fmt.Printf("intChan的长度：%v\n", len(intChan)) // intChan的长度：2
fmt.Printf("intChan的容量：%v\n", cap(intChan)) // intChan的容量：3

// 再次向channel中写入数据
intChan <- 44
intChan <- 11 // 报错：fatal error: all goroutines are asleep - deadlock!
```

### 从channel中读取数据

当读取channel数据时，如果管道数据已经全部取出，再取则会报错：`all goroutines are asleep - deadlock!`

```go
var intChan chan int
intChan = make(chan int, 3)
num := 99
intChan <- 10
intChan <- num
fmt.Printf("intChan的长度：%v\n", len(intChan)) // intChan的长度：2
fmt.Printf("intChan的容量：%v\n", cap(intChan)) // intChan的容量：3

// 取数据赋值给num2
num2 := <-intChan
fmt.Println("取出的数据：", num2)                 //取出的数据： 10
fmt.Printf("intChan的长度：%v\n", len(intChan)) // intChan的长度：1
fmt.Printf("intChan的容量：%v\n", cap(intChan)) // intChan的容量：3

// 继续取
<-intChan
<-intChan //fatal error: all goroutines are asleep - deadlock!
```

### channel使用注意事项

1. channel中只能存放指定的数据类型
2. channel的数据放满后，就不能再放入了
3. 如果从channel取出数据后，可以继续放入
4. 如果channel数据取完了，再取，就会报错：`deadlock!`

## channel的遍历和关闭

### channel的关闭

使用内置函数`close`可以关闭channel，当channel关闭后，就不能再向channel写数据了，但是仍然可以从该channel读取数据。

```go
intChan := make(chan int, 3)
intChan <- 10
intChan <- 20
close(intChan) //关闭管道
// 关闭管道后不可再写数据
// intChan <- 30  //panic: send on closed channel
n1 := <-intChan // 关闭管道后，可以继续读
fmt.Println(n1) //10
```



### channel的遍历

channel支持`for-range`的方式进行遍历，需要注意两点：

1. 在遍历时，如果channel没有关闭，则会出现`deadlock!`错误

    ```go
    intChan := make(chan int, 100)
    for i := 0; i < 100; i++ {
        intChan <- i
    }
    for v := range intChan {
        fmt.Println("v = ", v)
    } // 取完后，会报错：fatal error: all goroutines are asleep - deadlock!
    ```

    

2. 在遍历时，如果channel已经关闭，则会正常遍历数据，遍历完后，就会退出遍历

    ```go
    intChan := make(chan int, 100)
    for i := 0; i < 100; i++ {
        intChan <- i
    }
    close(intChan) //关闭管道
    for v := range intChan {
        fmt.Println("v = ", v)
    }
    ```



## 多路选择和超时

- `select`：进行多路选择
- `<-time.After(time.Microsecond * 100)`：检测超时

```go
package main

import (
	"fmt"
	"time"
)

func service() string {
	return fmt.Sprintf("%d", time.Now())
}

func AsyncService() chan string {
	retCh := make(chan string, 1)
	go func() {
		ret := service()
		fmt.Println("returned result.")
		retCh <- ret
		fmt.Println("service exited.")

	}()
	return retCh
}

func main() {
	select {
	case ret := <-AsyncService():
		fmt.Println(ret)
	case <-time.After(time.Microsecond * 100):
		fmt.Println("超时")
	}
}
```



## goroutine和channel结合

### 案例一

1. 开启一个writeData协程，向管道intChan中写入50个整数
2. 开启一个readData协程，从管道intChan中读取writeData写入的数据
3. writeData和readData操作的是同一个管道
4. 主线程需要等待writeData和readData协程都完成工作才能退出
   
```go
    func main() {
    	var intChan = make(chan int, 50)
    	var exitChan = make(chan bool, 1)
    
    	go func(intChan chan int) {
    		for i := 0; i < 50; i++ {
    			intChan <- i
    			fmt.Println("write Data写入管道信息 -->> ", i)
    		}
    		close(intChan) //关闭
    	}(intChan)
    	go func(intChan chan int, exitChan chan bool) {
    		for {
    			v, ok := <-intChan
    			if !ok {
    				break
    			}
    			fmt.Println("read Data读取管道信息 -->> ", v)
    		}
    		// 读取完数据，即任务完成
    		exitChan <- true
    		close(exitChan)
    	}(intChan, exitChan)
    
    	for {
    		_, ok := <-exitChan
    		if !ok {
    			break
    		}
    	}
    	fmt.Println("主线程结束")
    
    }
```

### 案例二

1. 启动一个协程，将1~2000的数放入到一个channel（numChan）中
2. 启动8个协程，从numChan取出数（n）,并计算1+……+n的值，并存放到resChan
3. 最后8个协程协同完成工作后，再遍历resChan，显示结果

```go
package main

import (
	"fmt"
	"sync"
)

func writeNum(ch chan<- int, wg *sync.WaitGroup) {
	for i := 1; i <= 2000; i++ {
		ch <- i
	}
	close(ch)
	wg.Done()
}

func readNum(ch <-chan int, resChan chan<- map[int]int, wg *sync.WaitGroup) {

	for num := range ch {
		res := 0
		for i := 0; i <= num; i++ {
			res += i
		}
		resChan <- map[int]int{num: res}
	}
	wg.Done()

}

func main() {
	numChan := make(chan int, 2000)
	resChan := make(chan map[int]int, 2000)
	var wg sync.WaitGroup
	wg.Add(1)
	go writeNum(numChan, &wg)
	for i := 0; i < 8; i++ {
		wg.Add(1)
		go readNum(numChan, resChan, &wg)
	}
	wg.Wait()
label:
	for {
		select {
		case v := <-resChan:
			fmt.Println(v)
		default:
			fmt.Println("无数据")
			break label
		}
	}

}

```



### 案例三

1. 要求统计1~200000的数字中，哪些是素数

```go
package main

import (
	"fmt"
	"sync"
)

func IsPrime(n int) bool {
	if n == 1 {
		return false
	}

	//从2遍历到n-1，看看是否有因子
	for i := 2; i < n; i++ {
		if n%i == 0 {
			//发现一个因子
			return false
		}
	}
	return true
}

func writeNum(ch chan<- int, wg *sync.WaitGroup) {
	for i := 1; i <= 200000; i++ {
		ch <- i
	}
	close(ch)
	wg.Done()
}

func primeNum(ch <-chan int, resChan chan<- int, wg *sync.WaitGroup) {
	for num := range ch {
		res := IsPrime(num)
		if res {
			resChan <- num
		}
	}

	wg.Done()

}

func main() {
	numChan := make(chan int, 200000)
	primeChan := make(chan int, 200000)
	var wg sync.WaitGroup
	wg.Add(1)
	go writeNum(numChan, &wg)
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go primeNum(numChan, primeChan, &wg)
	}
	wg.Wait()
label:
	for {
		select {
		case num := <-primeChan:
			fmt.Println(num)
		default:
			break label
		}

	}
}

```

## channel使用细节和注意事项

1. channel可以声明为只读，或者只写性质

    ```go
    // 1. 默认情况下，管道是双向的
    var chan1 chan int // 可读可写
    
    // 2. 声明只写
    var chan2 chan<- int //只写
    chan2 = make(chan int, 3)
    chan2 <- 20 //写
    
    // 3. 声明只读
    var chan3 <-chan int
    num2 := <-chan3 // 读
    ```

2. channel只读和只写的最佳时间案例

    ```go
    // 只写
    func send(ch chan<- int, exitChan chan struct{}) {
    	for i := 0; i < 10; i++ {
    		ch <- i
    	}
    	close(ch)
    	var a struct{}
    	exitChan <- a
    }
    
    //只读
    func recv(ch <-chan int, exitChan chan struct{}) {
    	for {
    		v, ok := <-ch
    		if !ok {
    			break
    		}
    		fmt.Println(v)
    	}
    	var a struct{}
    	exitChan <- a
    }
    
    func main() {
    	var ch chan int
    	ch = make(chan int, 10)
    	exitChan := make(chan struct{}, 2)
    	go send(ch, exitChan)
    	go recv(ch, exitChan)
        
    	var total = 0
    	for _ = range exitChan {
    		total++
    		if total == 2 {
    			break
    		}
    	}
    	fmt.Println("主线程结束。。。")
    
    }
    ```

3. 使用select可以解决从管道取数据的阻塞问题

    ```go
    func main() {
    
    	intChan := make(chan int, 10)
    	for i := 0; i < 10; i++ {
    		intChan <- i
    	}
    
    	stringChan := make(chan string, 5)
    	for i := 0; i < 5; i++ {
    		stringChan <- "Hello" + fmt.Sprintf("%d", i)
    	}
    	// 传统的方法再遍历管道时，如果不关闭会阻塞而导致 deadlock
    	// 使用select解决从管道取数据的阻塞问题
    label:
    	for {
    		select {
    		case v := <-intChan:
    			fmt.Println("从intChan读取的数据：", v)
    		case v := <-stringChan:
    			fmt.Println("从stringChan读取的数据：", v)
    		default:
    			fmt.Println("都取不到数据")
    			break label
    		}
    	}
    
    }
    ```

    

4. goroutine中使用recover，解决协程中出现panic，导致程序崩溃问题

    ```go
    func sayHello() {
    	for i := 0; i < 10; i++ {
    		time.Sleep(time.Second)
    		fmt.Println("Hello World")
    	}
    }
    
    func test() {
    	//可是使用defer + recover捕获异常
    	defer func() {
    		//捕获test抛出的panic
    		if err := recover(); err != nil {
    			fmt.Println("test() 发生错误: ", err)
    		}
    	}()
    	var myMap map[int]string
    	myMap[0] = "golang" //panic: assignment to entry in nil map
    }
    
    func main() {
    	//goroutine中使用recover，解决协程中出现panic，导致程序崩溃问题
    	go sayHello()
    	go test()
    
    	for i := 0; i < 10; i++ {
    		fmt.Println("main() ok=", i)
    		time.Sleep(time.Second)
    	}
    }
    ```




# 任务

## 任务的取消

```go
package main

import (
	"fmt"
	"time"
)

func isCancelled(cancelCh chan int) bool {

	select {
	case <-cancelCh:
		return true
	default:
		return false

	}
}

// 往管道里添加一个int数据，由于只会被一个任务获取到值，故只会取消一个任务
func cancel_1(cancelCh chan int) {
	cancelCh <- 0
}

// 关闭管道，以达到取消全部任务作用
func cancel_2(cancelCh chan int) {
	close(cancelCh)
}

func main() {
	cancelChan := make(chan int, 0)
	for i := 0; i < 5; i++ {
		go func(i int, cancelCh chan int) {
			for {
				if isCancelled(cancelCh) {
					break
				}
				time.Sleep(time.Second * 5)
			}
			fmt.Println(i, "Done")
		}(i, cancelChan)
	}
	//cancel_1(cancelChan) // 只会取消一个任务
	cancel_2(cancelChan) //全部取消
	time.Sleep(time.Second)
}

```

## Context上下文

- 根Context：通过`context.Background()`创建
- 子Context：通过`context.WithChancel(parentContext)`创建
    - `ctx, cancel := context.WithChancel(context.Baceground())`
- 当前Context被取消时，基于他的子context都会被取消
- 接收取消通知`<-ctx.Done()`

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func isCancelled(ctx context.Context) bool {
	select {
	case <-ctx.Done():
		return true
	default:
		return false
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	for i := 0; i < 5; i++ {
		go func(i int, ctx context.Context) {
			for {
				if isCancelled(ctx) {
					break
				}
				time.Sleep(time.Second * 5)
			}
			fmt.Println(i, "Cancelled")
		}(i, ctx)
	}
	cancel()
	time.Sleep(time.Second)

}
```

## 只运行一次（单例）

确保在多线程的情况下，只运行一次

```go
package main

import (
	"fmt"
	"sync"
	"unsafe"
)

type Singleton struct {
}

var singleInstance *Singleton
var once sync.Once

func GetSingletonObj() *Singleton {
	once.Do(func() {
		fmt.Println("Create Obj")
		singleInstance = new(Singleton)
	})
	return singleInstance
}

func main() {
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			obj := GetSingletonObj()
			fmt.Printf("%x\n", unsafe.Pointer(obj))
			wg.Done()
		}()
	}
	wg.Wait()
}

```

## 仅需任意任务完成

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func runTask(id int) string {
	time.Sleep(10 * time.Millisecond)
	return fmt.Sprintf("The result is from %d", id)
}

func FirstResponse() string {
	numOfRunner := 10
	ch := make(chan string, numOfRunner)// buffChannel防止协程泄漏
	for i := 0; i < numOfRunner; i++ {
		go func(i int) {
			ret := runTask(i)
			ch <- ret
		}(i)
	}
	return <-ch
}

func main() {
	fmt.Println("Before: ", runtime.NumGoroutine())
	response := FirstResponse()
	fmt.Println(response)
	time.Sleep(time.Second)
	fmt.Println("After: ", runtime.NumGoroutine())
}
```



## 所有任务完成

- 方式一：`sync.WaitGroup`
- 方式二：利用channel的CSP的机制

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func runTask(id int) string {
	time.Sleep(10 * time.Millisecond)
	return fmt.Sprintf("The result is from %d", id)
}

func AllResponse() string {
	numOfRunner := 10
	ch := make(chan string, numOfRunner) // buffChannel防止协程泄漏
	for i := 0; i < numOfRunner; i++ {
		go func(i int) {
			ret := runTask(i)
			ch <- ret
		}(i)
	}
	finalRet := ""
	for j := 0; j < numOfRunner; j++ {
		finalRet += <-ch + "\n"
	}
	return finalRet
}

func main() {
	fmt.Println("Before: ", runtime.NumGoroutine())
	response := AllResponse()
	fmt.Println(response)
	time.Sleep(time.Second)
	fmt.Println("After: ", runtime.NumGoroutine())
}

```



# 使用buffered channel 实现对象池

```go
package main

import (
	"errors"
	"fmt"
	"time"
)

type ReusableObj struct {
}

type ObjPool struct {
	bufChan chan *ReusableObj //用于缓冲可重用对象
}

func NewObjPool(numOfObj int) *ObjPool {
	objPool := ObjPool{}
	objPool.bufChan = make(chan *ReusableObj, numOfObj)
	for i := 0; i < numOfObj; i++ {
		objPool.bufChan <- &ReusableObj{}
	}
	return &objPool

}

// GetObj 从对象池中获取对象
func (p *ObjPool) GetObj(timeout time.Duration) (*ReusableObj, error) {
	select {
	case ret := <-p.bufChan:
		return ret, nil
	case <-time.After(timeout): //超时控制
		return nil, errors.New("time out")
	}
}

// ReleaseObj 放回对象到对象池
func (p *ObjPool) ReleaseObj(obj *ReusableObj) error {
	select {
	case p.bufChan <- obj:
		return nil
	default:
		return errors.New("overflow")

	}
}

func main() {
	pool := NewObjPool(10)

	for i := 0; i < 11; i++ {
		if v, err := pool.GetObj(time.Second); err != nil {
			panic(err)
		} else {
			fmt.Printf("%T\n", v)
			// 如果不放回，当循环达到10次后，再次获取对象则会超时
			if err := pool.ReleaseObj(v); err != nil {
				panic(err)
			}
		}
	}
	fmt.Println("Done")
}

```

# sync.Pool对象缓存

## sync.Pool对象获取

- 尝试从私有对象获取
- 私有对象不存在，尝试从当前Processor的共享池获取
- 如果当前Processor共享池也是空的，那么就尝试去其他Processor的共享池获取
- 如果所有子池都是空的，最后就用用户指定的New函数产生一个新的对象返回

## sync.Pool对象的放回
- 如果私有对象不存在则保存为私有对象
- 如果私有对象存在，放入到当前Processor子池的共享池中

## sync.Pool使用
```go
pool := &sync.Pool{
    New: func() interface{} {
        return 100
    },
}
v := pool.Get().(int) //取数据并做断言
...
pool.Put(3)  // 放数据
```

### 使用实例

```go
pool := &sync.Pool{
    New: func() interface{} {
        fmt.Println("Create a new object.")
        return 100
    },
}
v := pool.Get().(int) //取数据并做断言
fmt.Println(v)
pool.Put(3)  // 放数据
runtime.GC() // GC 会清除sync.pool中缓存的对象
v1, _ := pool.Get().(int)
fmt.Println(v1)
v2, _ := pool.Get().(int)
fmt.Println(v2)
/*
		不使用GC清除结果：
			Create a new object.
			100
			3
			Create a new object.
			100
		使用GC清除sync.pool中缓存的对象结果：
			Create a new object.
			100
			Create a new object.
			100
			Create a new object.
			100

*/
```
在协程中使用
```go
pool := &sync.Pool{
    New: func() interface{} {
        fmt.Println("Create a new object.")
        return 100
    },
}
pool.Put(100)
pool.Put(100)
pool.Put(100)

var wg sync.WaitGroup
for i := 0; i < 10; i++ {
    wg.Add(1)
    go func(id int) {
        fmt.Println(pool.Get())
        wg.Done()
    }(i)
}
wg.Wait()
/*
		结果：
			100
			100
			100
			Create a new object.
			100
			Create a new object.
			100
			Create a new object.
			100
			Create a new object.
			100
			Create a new object.
			100
			Create a new object.
			100
			Create a new object.
			100
*/
```

## sync.Pool总结：
- 适用于通过复用，降低复杂对象的创建和GC代价
- 协程安全，会有锁的开销
- 生命周期受GC影响，不适合于做连接池等，需自己管理生命周期的资源的池化
