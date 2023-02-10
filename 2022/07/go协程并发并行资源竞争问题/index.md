# Go协程并发(并行)资源竞争问题


## 并行（并发）问题

### 使用协程实现求n的阶乘

```go
// 定义一个map集合用来存放数据
var MyMap = make(map[int]int, 10)

// 求n的阶乘，并把数据存放到MyMap集合
func test2(n int) {
	res := 1
	for i := 1; i <= n; i++ {
		res *= i
	}
	MyMap[n] = res

}

func main() {
	
    // 开启200个协程执行
	for i := 1; i <= 200; i++ {
		go test2(i)
	}
	
    // 遍历map集合
	for i, v := range MyMap {
		fmt.Println(i, v)

	}
}
```

### 执行结果

❌报错：并发映射写入

```
fatal error: concurrent map writes
fatal error: concurrent map writes
```

### 执行图

![image-20220722115450636](https://raw.githubusercontent.com/XD825/picgo/main/img/202207221154737.png)

## 如何解决

### 不同协程之间如何通讯

- 全局变量加锁同步
- 使用channel（管道）

### 使用全局变量加锁同步解决

- 因为没有对全局变量 `MyMap` 加锁，因此会出现资源争夺问题，代码会出现错误：concurrent map writes
- 解决方案：加入互斥锁

```go

var (
	MyMap = make(map[int]uint64, 10)
	// lock 是一个全局互斥锁
	lock sync.Mutex
)

func test2(n int) {
	var res uint64 = 1
	for i := 1; i <= n; i++ {
		res *= uint64(i)
	}
	//加锁
	lock.Lock()
	MyMap[n] = res
	//解锁
	lock.Unlock()

}

func main() {
	for i := 1; i <= 200; i++ {
		go test2(i)
	}
	// 添加等待时间，等协程执行完毕
	time.Sleep(time.Second * 5)
	lock.Lock()
	for i, v := range MyMap {
		fmt.Println(i, v)
	}
	lock.Unlock()
}

```

#### 执行流程示意图

![image-20220722161032761](https://raw.githubusercontent.com/XD825/picgo/main/img/202207221610867.png)

### 使用等待组（sync.WaitGroup）主线程等待协程执行完毕

```go
var (
	MyMap = make(map[int]uint64, 10)
	// lock 是一个全局互斥锁
	lock sync.Mutex
	wg   sync.WaitGroup
)

func test2(n int) {
	var res uint64 = 1
	for i := 1; i <= n; i++ {
		res *= uint64(i)
	}
	//加锁
	lock.Lock()
	MyMap[n] = res
	//解锁
	lock.Unlock()
	wg.Done()

}

func main() {
    
	for i := 1; i <= 200; i++ {
		wg.Add(1)
		go test2(i)
	}
	wg.Wait()
	lock.Lock()
	for i, v := range MyMap {
		fmt.Println(i, v)
	}
	lock.Unlock()
}
```





### 使用channel

#### 为什么需要channel

1. 上面使用全局变量等待所有协程全部完成的时间很难确定

2. 如果主线程休眠的时间长了，会加长等待时间，如果短了，可能还有协程处于工作状态，这时也会随主线程的退出而销毁
3. 通过全局变量加锁同步来实现通讯，也并不利于多个协程对全局变量的读写操作

#### channel实现

1. 定义一个结果channel
2. 计算后的阶乘结果，放入到channel中
3. 最后在从channel取出结果打印出来

```go
func main() {
	type MyMap map[int]uint64
	resChan := make(chan MyMap, 200)
	var wg sync.WaitGroup
	for i := 1; i <= 200; i++ {
		wg.Add(1)
		go func(n int, resChan chan<- MyMap, wg *sync.WaitGroup) {
			var res uint64 = 1
			for i := 1; i <= n; i++ {
				res *= uint64(i)
			}
			resChan <- MyMap{n: res}
			wg.Done()
		}(i, resChan, &wg)
	}
	wg.Wait()
label:
	for {
		select {
		case res := <-resChan:
			fmt.Println(res)
		default:
			fmt.Println("无数据")
			break label
		}
	}
}
```




