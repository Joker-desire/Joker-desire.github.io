# Go语言转义字符

## Go常用的转义字符（escape char）

1. `\t` : 一个制表位，实现对齐的功能
2. `\n` : 换行符
3. `\\` : 一个`\`
4. `\"` : 一个`"`
5. `\r` : 一个回车 ，从当前行的最前面开始输出，覆盖掉以前内容

### 示例

```go
package escaptechar

import (
	"fmt"
	"testing"
)

func TestEscapteChar(t *testing.T) {
	// \t制表符
	fmt.Println("tom\tjack") //tom	jack
	// \n换行
	fmt.Println("hello\nword")
	// \\
	fmt.Println("c:\\Users\\Administrator\\Desktop\\Go语言") //c:\Users\Administrator\Desktop\Go语言
	// \"
	fmt.Println("Joker say \"Welcome to China!\"") // Joker say "Welcome to China!"
	// \r回车，从当前行的最前面开始输出，覆盖掉以前的内容
	fmt.Println("天龙八部雪山飞狐\r张飞") //张飞
}
```

```
=== RUN   TestEscapteChar
tom	jack
hello
word
c:\Users\Administrator\Desktop\Go语言
Joker say "Welcome to China!"
张飞
--- PASS: TestEscapteChar (0.00s)
```


