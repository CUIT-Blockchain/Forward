## 嵌入

Go编译的二进制文件一般都是单个，非常适合复制与部署，所以我们可以将一些配置文件、静态文件等嵌入到其中，就可以只执行单个文件来启动应用程序

Go 1.16 版本发布了嵌入功能，即`go embed`



## 嵌入为string

我们在当前文件目录下新建`hello.txt` : "hello world~"

编写main.go，使用`//go:embed hello.txt`来嵌入文件

```go
package main

import( 
	"fmt"
	_ "embed"
)

//go:embed hello.txt
var s string

func main () {	
	fmt.Println(s) // hello world~
}
```





## 嵌入为byte slice

同理

```go
package main

import( 
	"fmt"
	_ "embed"
)

//go:embed hello.txt
var b []byte

func main () {	
	fmt.Println(string(b)) // hello world~
}
```







## 嵌入为fs.FS



我们还可以嵌入为一个文件系统，通过这种方式我们可以嵌入多个文件

```go
package main

import (
	"embed"
	"fmt"
)

//go:embed hello.txt
//go:embed hello2.txt
var f embed.FS

func main () {	
	data, _ := f.ReadFile("hello.txt")
	fmt.Println(string(data))
	data, _ = f.ReadFile("hello2.txt")
	fmt.Println(string(data))
}
```



## 注意

embed不支持嵌入到函数内部的变量，如：

```go
package main
import (
	_ "embed"
	"fmt"
)
func main() {
	//go:embed hello.txt
	var s string
	fmt.Println(s)
}
```

报错：

```go
# command-line-arguments
./main.go:7:4: go:embed cannot apply to var inside func
```

