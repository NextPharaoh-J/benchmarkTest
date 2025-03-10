## Syntax

#### 数组与切片和字符串

```go
var b = [...]int{1, 2, 3}  // var a [3]int  这是数组的声明方式 固定长度，不可修改内容，只可以整体覆盖
var b = []int {1,2,3}// 切片，长度可变，内容可变
```

>go中的中文每字3字节，而符号和英文是1字节，直接用for range进行遍历会按照 Unicode 字符来遍历字符串

```go
// 字符串定义 字符串结构由两个信息组成：第一个是字符串指向的底层字节数组，第二个是字符串的字节的长度。
type StringHeader struct {
    Data uintptr
    Len  int
}
```



>```go
>str := "世界,123"
>for i, c := range str {
>    fmt.Printf("%d %c\n", i, char)
>}
>//0 世
>//3 界
>//6 ,
>//7 1
>//8 2
>//9 3
>
>for i := 0; i < len(str); i++ {
>	fmt.Println(i, str[i])
>}
>//0 228
>//1 184
>//2 150
>//3 231
>//4 149
>//5 140
>//6 44
>//7 49
>//8 50
>//9 51
>```
>
>例1 ,数组下标的索引值无法确定。

**切片**

```go
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```

`x := []int{2,3,5,7,11}` 和 `y := x[1:3]`

![image-20240418202759139](D:\img\image-20240418202759139.png)

`var a = []int{1,2,3} `

`a = append([]int{0}, a...)`

>在开头一般都会导致内存的重新分配，而且会导致已有的元素全部复制 1 次。因此，从切片的开头添加元素的性能一般要比从尾部追加元素的性能差很多。



**整形变量打印**

使用fmt包打印一个数值时，我们可以用%d、%o或%x参数控制输出的进制格式，就像下面的例子：

```Go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x)
// Output:
// 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

请注意fmt的两个使用技巧。通常Printf格式化字符串包含多个%参数时将会包含对应相同数量的额外操作数，但是%之后的`[1]`副词告诉Printf函数再次使用第一个操作数。第二，%后的`#`副词告诉Printf在用%o、%x或%X输出时生成0、0x或0X前缀。



#### 输入：`bufio` 包

```go
func main() {
    counts := make(map[string]int)
    input := bufio.NewScanner(os.Stdin)
    for input.Scan() {
        counts[input.Text()]++
    }
    // NOTE: ignoring potential errors from input.Err()
    for line, n := range counts {
        if n > 1 {
            fmt.Printf("%d\t%s\n", n, line)
        }
    }
}
```

继续来看 `bufio` 包，它使处理输入和输出方便又高效。`Scanner` 类型是该包最有用的特性之一，它读取输入并将其拆成行或单词；通常是处理行形式的输入最简单的方法。

程序使用短变量声明创建 `bufio.Scanner` 类型的变量 `input`。

```go
input := bufio.NewScanner(os.Stdin)
```

该变量从程序的标准输入中读取内容。每次调用 `input.Scan()`，即读入下一行，并移除行末的换行符；读取的内容可以调用 `input.Text()` 得到。`Scan` 函数在读到一行时返回 `true`，不再有输入时返回 `false`。

这个例子引入了 `ReadFile` 函数（来自于`io/ioutil`包），其读取指定文件的全部内容，`strings.Split` 函数把字符串分割成子串的切片。（`Split` 的作用与前文提到的 `strings.Join` 相反。）

```go
func main() {
    counts := make(map[string]int)
    for _, filename := range os.Args[1:] {
        data, err := ioutil.ReadFile(filename)
        if err != nil {
            fmt.Fprintf(os.Stderr, "dup3: %v\n", err)
            continue
        }
        for _, line := range strings.Split(string(data), "\n") {
            counts[line]++
        }
    }
    for line, n := range counts {
        if n > 1 {
            fmt.Printf("%d\t%s\n", n, line)
        }
    }
}

```

>%d          十进制整数
>%x, %o, %b  十六进制，八进制，二进制整数。
>%f, %g, %e  浮点数： 3.141593 3.141592653589793 3.141593e+00
>%t          布尔：true或false
>%c          字符（rune） (Unicode码点)
>%s          字符串
>%q          带双引号的字符串"abc"或带单引号的字符'c'
>%v          变量的自然形式（natural format）
>%T          变量的类型
>%%          字面上的百分号标志（无操作数）

####  二叉树实现插入排序

```go
type tree struct {
	value       int
	left, right *tree
}

func add(t *tree, value int) *tree {
	if t == nil {
		t = new(tree)
		t.value = value
		return t
	}
	if value < t.value {
		t.left = add(t.left, value)
	} else {
		t.right = add(t.right, value)
	}
	return t
}

func appendValue(values []int, t *tree) []int {
	if t != nil {
		values = appendValue(values, t.left)
		values = append(values, t.value)
		values = appendValue(values, t.right)
	}
	return values
}

func Sort(values []int) {
	var root *tree
	for _, v := range values {
		root = add(root, v)
	}
	appendValue(values[:0], root)
}

func main() {
	var arr = []int{1, 9, 8, 5, 6, 1, 2, 3}
	Sort(arr)
	fmt.Println(arr)
}
```









## Web

#### HTTP状态码

HTTP/1.1协议中定义了5类状态码， 状态码由三位数字组成，第一个数字定义了响应的类别

- 1XX 提示信息 - 表示请求已被成功接收，继续处理
- 2XX 成功 - 表示请求已被成功接收，理解，接受
- 3XX 重定向 - 要完成请求必须进行更进一步的处理
- 4XX 客户端错误 - 请求有语法错误或请求无法实现
- 5XX 服务器端错误 - 服务器未能实现合法的请求

#### HttpHandler

``` go
// Http2 is an e-commerce server with /list and /price endpoints.
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	db := database{"shoes": 50, "socks": 5}
	log.Fatal(http.ListenAndServe("localhost:8000", db))
}

type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars

//!+handler
func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	switch req.URL.Path {
	case "/list":
		for item, price := range db {
			fmt.Fprintf(w, "%s: %s\n", item, price)
		}
	case "/price":
		item := req.URL.Query().Get("item")
		price, ok := db[item]
		if !ok {
			w.WriteHeader(http.StatusNotFound) // 404
			fmt.Fprintf(w, "no such item: %q\n", item)
			return
		}
		fmt.Fprintf(w, "%s\n", price)
	default:
		w.WriteHeader(http.StatusNotFound) // 404
		fmt.Fprintf(w, "no such page: %s\n", req.URL)
	}
}

//!-handler
//run main.go and http://localhost:8080/price/?item=sock 
```





#### [Go代码的执行流程](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/03.4.md)

通过对http包的分析之后，现在让我们来梳理一下整个的代码执行过程。

- 首先调用Http.HandleFunc

  按顺序做了几件事：

  1 调用了DefaultServeMux的HandleFunc

  2 调用了DefaultServeMux的Handle

  3 往DefaultServeMux的map[string]muxEntry中增加对应的handler和路由规则

- 其次调用http.ListenAndServe(":9090", nil)

  按顺序做了几件事情：

  1 实例化Server

  2 调用Server的ListenAndServe()

  3 调用net.Listen("tcp", addr)监听端口

  4 启动一个for循环，在循环体中Accept请求

  5 对每个请求实例化一个Conn，并且开启一个goroutine为这个请求进行服务go c.serve()

  6 读取每个请求的内容w, err := c.readRequest()

  7 判断handler是否为空，如果没有设置handler（这个例子就没有设置handler），handler就设置为DefaultServeMux

  8 调用handler的ServeHttp

  9 在这个例子中，下面就进入到DefaultServeMux.ServeHttp

  10 根据request选择handler，并且进入到这个handler的ServeHTTP

  ```
    mux.handler(r).ServeHTTP(w, r)
  ```

  

  11 选择handler：

  A 判断是否有路由能满足这个request（循环遍历ServeMux的muxEntry）

  B 如果有路由满足，调用这个路由handler的ServeHTTP

  C 如果没有路由满足，调用NotFoundHandler的ServeHTTP

#### 文件上传

```go
package main

import (
	"crypto/md5"
	"fmt"
	"html/template"
	"io"
	"log"
	"net/http"
	"os"
	"strconv"
	"strings"
	"time"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
	r.ParseForm() //解析url传递的参数，对于POST则解析响应包的主体（request body）
	//注意:如果没有调用ParseForm方法，下面无法获取表单的数据
	fmt.Println(r.Form) //这些信息是输出到服务器端的打印信息
	fmt.Println("path", r.URL.Path)
	fmt.Println("scheme", r.URL.Scheme)
	fmt.Println(r.Form["url_long"])
	for k, v := range r.Form {
		fmt.Println("key:", k)
		fmt.Println("val:", strings.Join(v, ""))
	}
	fmt.Fprintf(w, "Hello astaxie!") //这个写入到w的是输出到客户端的
}

func login(w http.ResponseWriter, r *http.Request) {
	fmt.Println("method:", r.Method) //获取请求的方法
	if r.Method == "GET" {
		timestamp := strconv.Itoa(time.Now().Nanosecond())
		hashWr := md5.New()
		hashWr.Write([]byte(timestamp))
		token := fmt.Sprintf("%x", hashWr.Sum(nil))

		t, _ := template.ParseFiles("c:/Code/GO/test/login.html")
		t.Execute(w, token)
	} else {
		//请求的是登陆数据，那么执行登陆的逻辑判断
		r.ParseForm()
		token := r.Form.Get("token")
		if token != "" {
			//验证token的合法性
		} else {
			//不存在token报错
		}
		fmt.Println("username length:", len(r.Form["username"][0]))
		fmt.Println("username:", template.HTMLEscapeString(r.Form.Get("username"))) //输出到服务器端
		fmt.Println("password:", template.HTMLEscapeString(r.Form.Get("password")))
		template.HTMLEscape(w, []byte(r.Form.Get("username"))) //输出到客户端
	}
}

// 处理/upload 逻辑
func upload(w http.ResponseWriter, r *http.Request) {
	fmt.Println("method:", r.Method) //获取请求的方法
	if r.Method == "GET" {
		crutime := time.Now().Unix()
		h := md5.New()
		io.WriteString(h, strconv.FormatInt(crutime, 10))
		token := fmt.Sprintf("%x", h.Sum(nil))

		t, _ := template.ParseFiles("c:/Code/GO/test/upload.html")
		t.Execute(w, token)
	} else {
		r.ParseMultipartForm(32 << 20)
		file, handler, err := r.FormFile("uploadfile")
		if err != nil {
			fmt.Println(err)
			return
		}
		defer file.Close()
		fmt.Fprintf(w, "%v", handler.Header)
		f, err := os.OpenFile("./test/"+handler.Filename, os.O_WRONLY|os.O_CREATE, 0666) // 此处假设当前目录下已存在test目录
		if err != nil {
			fmt.Println(err)
			return
		}
		defer f.Close()
		io.Copy(f, file)
	}
}

func main() {
	http.HandleFunc("/", sayhelloName) //设置访问的路由
	http.HandleFunc("/login", login)   //设置访问的路由
	http.HandleFunc("/upload", upload)
	err := http.ListenAndServe(":9090", nil) //设置监听的端口

	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}

}
```



#### 字符串分割与匹配

```go
#按指定字符串进行分割
arr := strings.Split(s,sep)
s	要分割的字符串。
sep	字符串的分割符。
返回分割后的字符串 数组。

words := strings.Split("123 456 789"," ")
# [123,456,789]
```

```
#使用空格分隔
arr := strings.Fields("123 456 789")
# [123,456,789]
```

#### 使用sort包进行排序

>sort包的底层实现是多种排序的算法，例如快排，插入等等。调用时并不公开，也无需定义用那种算法。
>sort包内部会根据实际情况，自动选择最高效的排序算法。
>
>使用的时候仅仅需要三要素：序列长度，比较方式，交换方式：Len，Less,Swap

```
#实现（Sort）
//定义序列类型
type TestStringList []string
//定义排序三要素
func (t TestStringList) Len() int { ... }
func (t TestStringList) Less(i,j int) bool { ... }
func (t TestStringList) Swap(i,j int)  { ... }
//进行排序
sort.Sort(TestStringList)
//根据定义的排序规则取反
sort.Sort(sort.Reverse(TestStringList))

#实例
//元素个数
func (t TestStringList) Len() int {
    return len(t)
}
//比较结果
func (t TestStringList) Less(i, j int) bool {
    return t[i] < t[j]
}
//交换方式
func (t TestStringList) Swap(i, j int) {
    t[i], t[j] = t[j], t[i]
}
stringList := TestStringList{
        "1-php",
        "2-java",
        "3-golang",
        "4-c",
        "5-cpp",
        "6-python",
    }
sort.Sort(stringList)
//[1-php 2-java 3-golang 4-c 5-cpp 6-python]
```

Slice同样也可以进行排序，仅仅需要一个回调函数确定排序方式

```
 sort.Slice(arr,func(i,j int){
        return arr[i]<arr[j]
    })
```

#### 字符串和int类型相互转换

* string转成int：

  ```int, err := strconv.Atoi(string)```

* string转成int64：

  `` int64, err := strconv.ParseInt(string, 10, 64)``

* int转成string：
  `string := strconv.Itoa(int)`
* int64转成string：
  `string := strconv.FormatInt(int64,10)`

#### [Socke编程]([build-web-application-with-golang/zh/08.1.md at master · astaxie/build-web-application-with-golang (github.com)](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/08.1.md))

**net包/TCP**

go的net包对IP定义如下

```
type IP []byte
```

将string转换为net.IP类型`ParseIP(s string)`

```go
addr := net.ParseIP(name)
```

服务器和客户端交互的channel定义，用于读写数据

```
func (c *TCPConn) Write(b []byte) (int, error)
func (c *TCPConn) Read(b []byte) (int, error)
```

一个TCP的地址信息表示为,通过ResolveTCPAddr获取地址

```
type TCPAddr struct {
	IP IP
	Port int
	Zone string // IPv6 scoped addressing zone
}
```

```
func ResolveTCPAddr(net, addr string) (*TCPAddr, os.Error)
```

- net参数是"tcp4"、"tcp6"、"tcp"中的任意一个，分别表示TCP(IPv4-only), TCP(IPv6-only)或者TCP(IPv4, IPv6的任意一个)。
- addr表示域名或者IP地址，例如"[www.google.com:80](http://www.google.com/)" 或者"127.0.0.1:22"。

 

**客户端建立TCP连接**

```
func DialTCP(network string, laddr, raddr *TCPAddr) (*TCPConn, error)
```

- network参数是"tcp4"、"tcp6"、"tcp"中的任意一个，分别表示TCP(IPv4-only)、TCP(IPv6-only)或者TCP(IPv4,IPv6的任意一个)
- laddr表示本机地址，一般设置为nil
- raddr表示远程的服务地址

 ```go
 
 package main
 
 import (
 	"fmt"
 	"io/ioutil"
 	"net"
 	"os"
 )
 
 func main() {
 	if len(os.Args) != 2 {
 		fmt.Fprintf(os.Stderr, "Usage: %s host:port ", os.Args[0])
 		os.Exit(1)
 	}
 	service := os.Args[1]
 	tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
 	checkError(err)
 	conn, err := net.DialTCP("tcp", nil, tcpAddr)
 	checkError(err)
 	_, err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
 	checkError(err)
     // go 1.6之后弃用的读通道方法
 	// result, err := ioutil.ReadAll(conn)
 	result := make([]byte, 256)
 	_, err = conn.Read(result)
 	checkError(err)
 	fmt.Println(string(result))
 	os.Exit(0)
 }
 func checkError(err error) {
 	if err != nil {
 		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
 		os.Exit(1)
 	}
 }
 
 ```

**服务端建立TCP连接**

```go
func ListenTCP(network string, laddr *TCPAddr) (*TCPListener, error)
func (l *TCPListener) Accept() (Conn, error)
```

同样的是来自net中的交互函数声明，参数与上相同。下是一个时间同步服务，端口为7777

```go
package main

import (
    "fmt"
    "os"
    "time"
    "net"
)

func main() {
    service := ":7777"
    tcpAddr, err := net.ResolveTCPAddr("tcp", service)
    checkError(err)
    listener, err := net.ListenTCP("tcp", tcpAddr)
    checkError(err)
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        daytime := time.Now().String()
        conn.Write([]byte(daytime))
        conn.Close()
    }
}

func checkError(err error) {
    if err != nil {
        fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
        os.Exit(1)
    }
}
```

![image-20231105095856570](D:\img\image-20231105095856570.png)

当客户端对本地地址127.0.0.1:7777进行访问时，返回时间。但是这是单线程任务。

```go

package main

import (
	"fmt"
	"net"
	"os"
	"time"
)

func main() {
	service := ":1200"
	tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
	checkError(err)
	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)
	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}
		go handleClient(conn)
	}
}

func handleClient(conn net.Conn) {
	defer conn.Close()
	daytime := time.Now().String()
	conn.Write([]byte(daytime)) // don't care about return value
	// we're finished with this client
}
func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
		os.Exit(1)
	}
}

```

通过go routine 将业务处理分离到`handleClient`

`	conn.SetReadDeadline(time.Now().Add(2 * time.Minute))` // set 2 minutes timeout

当客户端超过一段时间没有发送请求时，conn自动给关闭。在使用`	read_len, err := conn.Read(request)`读取数据时，request需要设定固定长度，并且每次读取完之后需要清空。

`	request := make([]byte, 128)` // set maxium request length to 128B to prevent flood attack

**控制TCP连接**

TCP有很多连接控制函数，我们平常用到比较多的有如下几个函数：

```
func DialTimeout(net, addr string, timeout time.Duration) (Conn, error)
```

设置建立连接的超时时间，客户端和服务器端都适用，当超过设置时间时，连接自动关闭。

```
func (c *TCPConn) SetReadDeadline(t time.Time) error
func (c *TCPConn) SetWriteDeadline(t time.Time) error
```

用来设置写入/读取一个连接的超时时间。当超过设置时间时，连接自动关闭。

```
func (c *TCPConn) SetKeepAlive(keepalive bool) os.Error
```

设置keepAlive属性。操作系统层在tcp上没有数据和ACK的时候，会间隔性的发送keepalive包，操作系统可以通过该包来判断一个tcp连接是否已经断开，在windows上默认2个小时没有收到数据和keepalive包的时候认为tcp连接已经断开，这个功能和我们通常在应用层加的心跳包的功能类似。

[**UDP socket**](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/08.1.md#udp-socket)

与tcp相比，UDP在对多个客户端请求数据包的方式不同，缺少了对客户端连接请求的Accept函数。

```go
func ResolveUDPAddr(net, addr string) (*UDPAddr, os.Error)
func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err os.Error)
func ListenUDP(net string, laddr *UDPAddr) (c *UDPConn, err os.Error)
func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err os.Error)
func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (n int, err os.Error)
```

客户端和服务器的代码除了函数名字几乎一致

#### [Websocket]([build-web-application-with-golang/zh/08.2.md at master · astaxie/build-web-application-with-golang (github.com)](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/08.2.md))

相比于传统的HTTP，websocket可以持续连接，而不是为了即时通信采用轮询的方式定时发送请求进行刷新。优势如下

- 一个Web客户端只建立一个TCP连接
- Websocket服务端可以推送(push)数据到web客户端.
- 有更加轻量级的头，减少数据传送量

**REST**

>REST是一种架构风格，汲取了WWW的成功经验：无状态，以资源为中心，充分利用HTTP协议和URI协议，提供统一的接口定义，使得它作为一种设计Web服务的方法而变得流行。在某种意义上，通过强调URI和HTTP等早期Internet标准，REST是对大型应用程序服务器时代之前的Web方式的回归。目前Go对于REST的支持还是很简单的，通过实现自定义的路由规则，我们就可以为不同的method实现不同的handle，这样就实现了REST的架构。**REST就是根据不同的method访问同一个资源的时候实现不同的逻辑处理。**

**RPC(Remote Procedure Call Protocol)**

- 在分布式系统中，允许一个计算机程序调用另一个计算机程序上的函数或方法，就像本地函数调用一样。这使得程序能够协同工作，执行分布在不同计算机上的任务。
- RPC 可以用于构建分布式系统的各个组件，例如，在微服务架构中，各个微服务之间可以通过 RPC 进行通信。
- 它允许客户端和服务端在网络上进行通信，执行远程的计算任务。例如，在示例中，客户端可以请求服务端执行数学运算。

#### 网络安全

**CSRF攻击：跨站请求伪造**

> 攻击者利用用户已登录的状态，通过伪造请求诱导用户执行不知情的操作，例如在受害者已登录到某个网站后，通过钓鱼链接触发 CSRF 攻击，执行一些未经授权的操作，但不能直接访问或窃取 Cookie。

要完成一次CSRF攻击，受害者必须依次完成两个步骤 ：

- 1.登录受信任网站A，并在本地生成Cookie 。
- 2.在不退出A的情况下，访问危险网站B。

CSRF攻击主要是因为Web的隐式身份验证机制，Web的身份验证机制虽然可以保证一个请求是来自于某个用户的浏览器，但却无法保证该请求是用户批准发送的。

预防操作：

- 正确使用GET,POST,COOKIE。

​		GET用于不改变资源属性，仅仅返回资源

​		POST用于发送表单，改变属性，或一些别的事情，将所有设计用户权限的操作设置为POST

- 在非GET请求中增加伪随机数

  - 为每个用户生成一个唯一的cookie token，所有表单都包含同一个伪随机值,这种方案最简单，因为攻击者不能获得第三方的Cookie(理论上)，所以表单中的数据也就构造失败，但是由于用户的Cookie很容易由于网站的XSS漏洞而被盗取，所以这个方案必须要在没有XSS的情况下才安全。

  - 每个请求使用验证码，这个方案是完美的，因为要多次输入验证码，所以用户友好性很差，所以不适合实际运用。

  - 不同的表单包含一个不同的伪随机值


生成随机数token

```
h := md5.New()
io.WriteString(h, strconv.FormatInt(crutime, 10))
io.WriteString(h, "ganraomaxxxxxxxxx")
token := fmt.Sprintf("%x", h.Sum(nil))

t, _ := template.ParseFiles("login.gtpl")
t.Execute(w, token)
```

输出token

```
<input type="hidden" name="token" value="{{.}}">
```

验证token

```
r.ParseForm()
token := r.Form.Get("token")
if token != "" {
	//验证token的合法性
} else {
	//不存在token报错
}
```

>**同源策略**
>
>1. **域名不同**: 同源策略要求网页中的 JavaScript 只能访问与其来源相同的域名，协议和端口。如果网页来自 example.com，它只能与 example.com 下的资源进行交互。
>2. **Cookie 安全性**: 在同源策略中，浏览器的 Cookie 存储是域名敏感的。这意味着 JavaScript 无法直接访问不属于当前网页域名的 Cookie。
>3. **XHR (XMLHttpRequest) 限制**: XMLHttpRequest（一种用于创建和发送 HTTP 请求的 JavaScript API）也受到同源策略的限制，因此它不能跨域发送请求。

#### Gin

![image-20240620152424880](D:\img\image-20240620152424880.png)

```go
r := gin.Default()
r.GET("/path", func(c *gin.Context) {
    id := c.Param("id") // url路径获取
    //user := c.Query("user") // params参数获取 ?user="  "
    user := c.DefaultQuery("user", "admin") // 设置未传参时的默认user
    pwd := c.Query("pwd")
    c.JSON(200, gin.H{
        "id":   id,
        "user": user,
        "pwd":  pwd,
    })
})

r.POST("/path/:id", func(c *gin.Context) {
    user := c.DefaultPostForm("user", "admin") //post中的默认参数设置
    pwd := c.PostForm("pwd")
    c.JSON(200, gin.H{
        "user": user,
        "pwd":  pwd,
    })
})
r.Run("8008")//运行服务器 
```

#### Redis

```go
redis-server #cmd 开启redis服务端 默认6379端口
netstat -an | findstr 6379 查看运行情况
```

## Algorithm

### 堆和栈

栈（Stack）和堆（Heap）是两种用于内存管理的不同区域，它们在使用方式和管理策略上有显著的区别。了解它们的区别对于理解程序的内存管理和性能优化非常重要。以下是它们之间的主要区别：

 **栈（Stack）**

1. **内存分配和释放**：
   - 栈内存的分配和释放由编译器自动管理，通常是在函数调用时进行分配，在函数返回时进行释放。这是一种非常高效的内存管理方式。
2. **存储内容**：
   - 栈主要用于存储局部变量、函数调用参数、返回地址等。栈上的内存分配方式是连续的，按照先入后出的顺序管理。
3. **速度**：
   - 栈内存的分配和释放非常快，因为它遵循严格的LIFO（Last In First Out）顺序，不需要复杂的管理和查找。
4. **内存大小**：
   - 栈的大小通常比较小，并且是固定的（在程序启动时确定）。这意味着栈上不能存储太多或太大的数据。
5. **线程安全**：
   - 每个线程有自己的栈，因此栈内存是线程安全的，不需要额外的同步机制。

**堆（Heap）**

1. **内存分配和释放**：
   - 堆内存的分配和释放由程序员或垃圾收集器管理。程序员需要显式地分配内存（如使用 `new` 或 `make`），并且在不再使用时释放内存（在 Go 中由垃圾收集器自动管理）。
2. **存储内容**：
   - 堆用于存储需要在多个函数之间共享的数据或生命周期超过函数调用的数据。堆上的内存可以在任何时候分配和释放，适用于动态和不确定大小的内存需求。
3. **速度**：
   - 堆内存的分配和释放相对较慢，因为它需要复杂的内存管理机制，如内存碎片整理和垃圾回收。
4. **内存大小**：
   - 堆的大小通常较大，并且是动态的（可以根据需要增长），这使得它适合存储大量或大型数据。
5. **线程安全**：
   - 堆内存是共享的，需要通过同步机制来保证线程安全。这增加了并发编程的复杂性。

### 位运算技巧

**最低位1的获取/- b的二进制表示**

我们可以用 b & (−b) 得到 b 二进制表示中最低位的 1，这是因为 (−b) 在计算机中以补码的形式存储，它等于 ∼b+1。b 如果和 ∼b 进行按位与运算，那么会得到 0，但是当 ∼b 增加 1 之后，最低位的连续的 1 都变为 0，而最低位的 0 变为 1，对应到 b 中即为最低位的 1，因此当 b 和 ∼b+1 进行按位与运算时，只有最低位的 1 会被保留；

- *b* & (−*b*) 可以获得 *b* 的二进制表示中的最低位的 1 的位置；
- *b* & (*b*−1) 可以将 *b* 的二进制表示中的最低位的 1 置成 0。

### 逃逸分析（Escape Analysis）

**Go编译器通过逃逸分析来决定变量是分配在栈上还是堆上**。以下是逃逸分析的一些关键点：

1. **局部变量**：如果一个变量只在函数内部使用，并且不会被外部引用，那么它会被分配在栈上。
2. **逃逸变量**：如果一个变量的引用被传递到函数外部（如通过指针或闭包），或者生命周期超出当前函数的范围，则它会逃逸到堆上。
3. **优势**：逃逸分析可以提高内存管理的效率。栈上的内存分配和释放开销较低，使用栈可以减少垃圾回收的压力和频率，从而提高程序性能。

### KMP算法 

**匹配字符串问题**:给你两个字符串 `文本串haystack` 和 `模式串needle` ，请你在 `haystack` 字符串中找出 `needle` 字符串的第一个匹配项的下标（下标从 0 开始）。如果 `needle` 不是 `haystack` 的一部分，则返回 `-1` 。

通过构建最长相等先后缀来加速字符串匹配的过程，next数组记录的就是最长公共前后缀的长度，next[i]表示到i位置为止，最长的相等前后缀，如

next[0] = 0 因为只有一个数

next[3] = 1 因为next[3]=next[0]=A且next[2]!=next[1]

<img src="D:\img\image-20240522154225515.png" alt="image-20240522154225515" style="zoom:50%;" />

next数组的获取可以用自身迭代加速

> n为文本串长度，m为模式串长度，因为在匹配的过程中，根据前缀表不断调整匹配的位置，可以看出匹配的过程是O(n)，之前还要单独生成next数组，时间复杂度是O(m)。所以整个KMP算法的时间复杂度是O(n+m)的。

**前缀表的加速原理**是，跳过next表中前缀后缀相同的部分

![image-20240522155912233](D:\img\image-20240522155912233.png)

上述红色部分的计算跳过长度的公式为**跳过的个数=匹配上字符串长度-重复字符串长度(查表)**，一切的加速都是在模式串和next表上进行的。

**next表**的迭代加速获取方式：

如果next[0], next[1], ... next[x-1]均已知，那么如何求出 next[x] ？

我们已经知道next[x-1],标记 `next[x-1]=temp` 为重复的前后缀个数 ,则可以讨论A[temp]和A[x]的值，分2种情况讨论：

> A[temp]表示前缀的相同部分最后一个数的下一个数，这是数组下标的trick,因为temp是相同前后缀长度，而下标从0开始，带入后会相差1，正好表示需要比较的位置，A[x]表示最后一个数
>
> ![image-20240522160810264](D:\img\image-20240522160810264.png)

- A[temp]等于A[x]，也就是说在前一个next结果上又多了一个字符串相同的长度，因此`next[x] = next[x-1]+1`
- **当A[temp]和A[x]不相等的时候**，我们需要缩小temp,`把temp变成next[temp-1]`，直到A[temp]=A[x]为止。*这种迭代加速的目的是跳过不匹配字符，并且从上一个匹配字符的相同前后缀*

![image-20240522160845148](D:\img\image-20240522160845148.png)

最后得到的匹配长度同样是`next[x] = next[x-1]+1`，`ABA 长度为3`

```go
func strStr(haystack, needle string) int {
	n, m := len(haystack), len(needle)
	if m == 0 {
		return 0
	}
    
    // getNext表
	next := make([]int, m)
	for i, j := 1, 0; i < m; i++ {
        // 找到A[temp]等于A[x]的时候，j就是temp
		for j > 0 && needle[i] != needle[j] {
			j = next[j-1]
		}
        // next[x] = next[x-1]+1
		if needle[i] == needle[j] {
			j++
		}
		next[i] = j  // j是temp=next[x-1]
	}
    
    // 加速匹配
    //i 是 haystack（大字符串）中的当前索引
	//j 是 needle（模式字符串）中的当前索引
	for i, j := 0, 0; i < n; i++ {
		for j > 0 && haystack[i] != needle[j] {
			j = next[j-1]	// 将模式串匹配上的数量更新
		}
		if haystack[i] == needle[j] {
			j++   		// 模式串和原串当前位置匹配
		}
		if j == m { // 模式串全部匹配
			return i - m + 1
		}
	}
	return -1
}
```

> KMP 算法通过利用 `next` 数组来避免重复匹配，在匹配过程中高效地跳过已经匹配的部分，使得时间复杂度降为 O(n + m)，其中 n 是 `haystack` 的长度，m 是 `needle` 的长度。这段代码展示了如何利用 `next` 数组在匹配过程中进行跳跃以提高效率。

### 排序算法

![image-20231209162045083](D:\img\image-20231209162045083.png)

![image-20231209163500419](D:\img\image-20231209163500419.png)

#### 冒泡排序

```go
func bubble(list []int) []int {
    //用来判断是否完成了排序
    flag:=true
    len := len(list)
    for i:=0;i<len-1;i++{
        //预设这一轮没发生数的交换,在比较中，如果发生了数的交换，则把flag置为false
        flag=true
        for j:=0;j<len-1-i;j++{
            if list[j]>list[j+1]{
                //在这一轮中有数的交换
                flag=false
                list[j],list[j+1] = list[j+1],list[j]
            }
        }
        //如果这一轮中没有数的交换，就表示已经排序完成，直接跳出循环
        if flag{
            break
        }
    }
    return list
}
```

#### 选择排序

```go
func compare(list []int) []int {
    len := len(list)
    for i:=0;i<len-1;i++{
        for j:=i+1;j<len;j++{
            if list[i]>list[j]{
                list[j],list[i] = list[i],list[j]
            }
        }
    }
    return list
}
```

#### 快速排序

1.hoare方法(常见，1.0版本)

其单趟排序的思路是：取区间中最左或最右边的元素为key，定义两个变量，这里假设是i和j，j从区间的最右边向左走，找到比key小的元素就停下。i从最左边向右走，找到比key大的元素就停下。然后交换i和j所指向的元素，重复上面的过程，直到i,j相遇，此时i,j相遇的位置就是key排序后待的位置。

>设定哨兵priovt，将数组于等于它的和大于它的放在它的左边和右边。
>
>1） 左侧数组长度 i ，若[i]<=privot，[i]和<=区的下一个数进行交换，<=区右扩，i++
>
>2） [i]>num,i++
>
>本质就是将<=区逐渐向右扩展，直到数组索引越界
>
>实现方式为将数组中第一个数设为privot，将大于和小于它的数从中间分开变为[ priovt,{小于},{大于}]，然后将priovt和小于区最右边的一个数进行交换。得到[{小于},priovt,{大于}]，然后分别对{小于}{大于}在使用该算法(哨兵分区，再将哨兵放到中间)
>
>

```
func QuickSort(arr []int, left, right int) {
	if left >= right {
		return
	}
	i, j := left, right
	privot := arr[i]
	for i < j {
		for i < j && arr[j] >= privot {
			j--
		}
		arr[i] = arr[j]
		for i < j && arr[i] <= privot {
			i++
		}
		arr[j] = arr[i]
	}
	arr[i] = privot
	QuickSort(arr, left, i-1)
	QuickSort(arr, i+1, right)
}
```

2.挖坑法

这个方法单趟排序的思路是：取最左或最右边的元素为key，假设把这个位置“挖空”，让最右边的q向左走，直到遇到比key小的数，将其放到key的位置，自己的位置变“空”。直到pq相遇，那么这个位置就是最终的坑位，再将key填入这个坑位，就完成了一趟排序。

```go
func QuickSort(arr []int, left, right int) int {
    key := left  //取最左边的元素作为key
    privot := arr[left]
    for left < right {
        for left < right && arr[right] >= privot {  //从右往左找，找到比privot小的就停下来
            right--
        }
        arr[key] = arr[right]   //将元素填入坑内
        key = right    //更新坑的位置
        for left < right && arr[left] <= privot {   //从左往右找，找到比privot大的就停下来
            left++
        }
        arr[key] = arr[left]
        key = left
    }
    arr[key] = privot
    return key  
}
```

3.快慢指针法

取最左边的数为key，定义两个快慢指针，都从key的下一位往右走，fast每走一步判断一下它指向的元素是否小于key，若小于则交换fast和slow位置的元素，并且让slow向前走，直到fast走到底，结束循环。最后让slow和key位置的值交换。再返回key的位置。

```go
func QuickSort(arr []int, left, right int) int {
    key := left   //取最左边的为key
    fast := key+1 //快指针
    slow := key   //慢指针
    for fast <= right {
        if arr[fast] < arr[key] {   //当快指针指向元素小于key就交换
            arr[slow], arr[fast] = arr[fast], arr[slow]
            slow++
        }
        fast++
    }
    arr[key], arr[slow] = arr[slow], arr[key]   //慢指针回退一位再交换
    return slow   //返回key的位置
}
```

快排2.0版本(荷兰国旗)

![image-20231206211848773](D:\img\image-20231206211848773.png)数组按哨兵分为三部分，然后将哨兵交换放到大于区的第一个位置

![image-20231206211944477](D:\img\image-20231206211944477.png)

效率略高于1.0，由于等于部分被单独拿出来，再次迭代大小区的长度就会减少

![image-20231206212115256](D:\img\image-20231206212115256.png)

快排3.0，随机选择一个数交换在最右边，从而实现统计上的O（n*logn）

#### 插入排序

时间复杂度O(n2) 空间复杂度O(1)

```go
func insertSort(nums []int) {
    if nums == nil || len(nums) <= 1 {
       return
    }
    //对第i个数，将其插入到前i-1个有序数组的某一位置
    //对于增序排列，遍历从i=j+1一直到比它小的第一个数进行插入，如果比它大就交换位置
    for i := 1; i < len(nums); i++ {
       for j := i - 1; j >= 0 && nums[j] > nums[j+1]; j-- {
          swap(nums, j, j+1)
       }
    }
}
//交换i和j位置元素，不使用新内存
func swap(nums []int, i, j int) {
    nums[i] = nums[i] ^ nums[j]
    nums[j] = nums[i] ^ nums[j]
    nums[i] = nums[i] ^ nums[j]
}
```



#### 归并排序

整体是简单递归，左边排好序，右边排好序，让整体有序。

将数组内的左右两侧有序数组合并为新的有序数组，不断迭代子数组，直到数组只有一个元素(L==R)

时间复杂度O(N*logN)，空间复杂度O(N)

```go
func mergeSort(nums []int, left, right int) {
    if left == right {
       return
    }
    // a>>1 等效于 a/2 都是向下取整的除法
    mid := left + (right-left)>>1
    mergeSort(nums, left, mid)
    mergeSort(nums, mid+1, right)
    merge(nums, left, mid, right)
}

func merge(nums []int, left, mid, right int) {
    // 将nums从left到mid 从mid+1到right的 左右两部分有序list进行归并merge
    // 具体为顺序遍历左右侧list，若左侧小于等于右侧则将左侧元素放入新list，使左侧指针位置++，否则放入右侧元素，令右侧指针++
    
    // 新建list大小用于存放挨个放进去的元素
    temp := make([]int, right-left+1)
    
    // m,n分别为左侧和右侧起始下标
    m, n := left, mid+1
    
    // pos为新list的下标 如果使用append不需要该变量
    pos := 0
    
    //遍历两个数组直到其中一个index指针越界
    for m <= mid && n <= right {
       // go不支持三元表达式和nums[i++]类似操作
       // nums[m]<=nums[n] ? temp[pos++] = nums[n++]:temp[pos++] = nums[m++]
       if nums[m] <= nums[n] {
          temp[pos] = nums[m]
          pos++
          m++
       } else {
          temp[pos] = nums[n]
          pos++
          n++
       }
    }
    for m <= mid {
       temp[pos] = nums[m]
       pos++
       m++
    }
    for n <= right {
       temp[pos] = nums[n]
       pos++
       n++
    }
    
    // 此时temp已经完成了归并，是左右侧排序后的结果，将其粘贴回原数组
    for i := 0; i < len(temp); i++ {
       nums[left+i] = temp[i]
    }
}
```

#### 堆排序

```go
// 大根堆是指每个结点的值大于等于其左右孩子的值
// 某个数处在index的位置上，将其向上更新，不断与父节点进行比较[(i-1)/2]
func heapInsert(arr []int, index int) {
    // 但它父亲存在且它的值大于它的父亲
    for arr[index] > arr[(index-1)/2] {
       swap(arr, index, (index-1)/2)
       index = (index - 1) / 2
    }
}
```

```go
// 某个数在index的位置上，能否向下移动
func heapify(arr []int, index, heapSize int) {
    left := index*2 + 1
    var largest int
    for left < heapSize {
       // 比较两个孩子中谁的值大，将下标给largest,left为左孩子,left+1为右孩子
       // 且判断有没有孩子，若左孩子不存在，则一定没有孩子
       if arr[left+1] > arr[left] {
          largest = left + 1
       } else {
          largest = left
       }

       // 比较父[index]和子[largest]之间谁的值大，下标给largest
       if arr[largest] > arr[index] {
          // 子大于父，交换父子值，将index放到子位置上，继续寻找大于其的父结点
          swap(arr, largest, index)
          index = largest
          left = index*2 + 1
       } else {
          // 父大于等于子，则不用交换，直接退出
          break
       }
    }

}
```

```go
func heapSort(arr []int) {
	if arr == nil || len(arr) < 2 {
		return
	}
	for i := 0; i < len(arr); i++ {
		// 分成大根堆 O(logN)
		heapInsert(arr, i)
	}
	// 堆中元素heapSize个
	heapSize := len(arr) - 1

	// 交换堆顶(最大)，堆尾(最小)的元素的数组值，并将heapSize--来去掉堆中最小元素的下标(最大数出堆)
	// 从而将最后一个数置为数组中最大的元素

	swap(arr, 0, heapSize)
	heapSize--
	// heapSize表示堆能访问到的位置，此时堆顶元素为最小值，需要依次与其子结点进行比较
	for heapSize > 0 {
		// 通过递归以该最小值为堆顶的大根堆，重新得到正确的大分堆
		heapify(arr, 0, heapSize)
		// 将次大元素依次放置于len(arr)-heapSize的位置上。完成排序
		swap(arr, 0, heapSize)
		heapSize--
	}

}
```

```go
func swap(nums []int, i, j int) {
    if i == j {
       return
    }
    nums[i] ^= nums[j]
    nums[j] ^= nums[i]
    nums[i] ^= nums[j]
}
func main() {
	nums := []int{2, 5, 4, 7, 8, 1, 6, 7, 9, 1, 0, 6, 11}
	heapSort(nums)
	fmt.Println(nums)
}
```

#### 桶排序

一种特殊的排序算法，不属于比较排序。工作的原理是将数组分到有限数量的桶里。每个桶再个别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序），最后依次把各个桶中的记录列出来记得到有序序列。当要被排序的数组内的数值是均匀分配的时候，桶排序使用线性时间（Θ(n)）。但桶排序并不是比较排序，不受到O(n log n)下限的影响。

**基数排序**

```go
// 桶排序之基数排序
// 根据数组中的数据的每一位大小排序
// 首先从个位数上的值进行升序排列(入桶出桶)
// 然后从十位、百位...直到最大位进行升序排列
func radixSort(arr []int, L, R int, digit int) {
    // L,R分别表示数组需要排序的左右边界
    // digit表示最大数的位数
    // radix表示十进制，即用10个桶进行分类
    const radix int = 10
    i, j := 0, 0
    // 有多少个数准备多少个辅助空间
    bucket := make([]int, R-L+1)
    // 按位数从低到高进行排序
    for d := 1; d <= digit; d++ {
       count := make([]int, radix)
       for i = L; i <= R; i++ {
          // 将所有数的第d位统计次数
          j = getdigit(arr[i], d)
          count[j]++
       }
       for i = 1; i < radix; i++ {
          // 取前缀和作为新的值
          // count[0]表示当前位(d位)是0数字有多少个
          // count[1]表示当前位(d位)是0和1的数字有多少个
          // count[i]表示当前位(d位)是(0~i)的数字有多少个
          count[i] += count[i-1]
       }
       // 得到有序数组bucket
       for i = R; i >= L; i-- {
          // 从数组最右边开始遍历，依次弹出到bucket中
          // bucket为hash类型的数据结构，将第d位上的值准确映射到有序位置
          j = getdigit(arr[i], d)
          bucket[count[j]-1] = arr[i]
          count[j]--
       }
       // 将arr的排序部分更新为bucket排序后的值
       for i, j = L, 0; i <= R; i, j = i+1, j+1 {
          arr[i] = bucket[j]
       }
    }
}
```

```go
func getdigit(x, d int) int {
    // 返回数x的第d位数
    // 如getdigit(123456,2) => 5
    return (x / (int(math.Pow(10, float64(d-1)))) % 10)
}
func maxdigit(x []int) (res int) {
    // 计算数组最大数的位数
    max := math.MinInt
    for _, k := range x {
       if max < k {
          max = k
       }
    }
    for max != 0 {
       res++
       max /= 10
    }
    return
}
func main() {
	nums := []int{2, 5, 4, 7, 8, 1, 6, 7, 9, 1, 0, 6, 11}
	radixSort(nums, 0, len(nums)-1, maxdigit(nums))
	fmt.Println(nums)
	//[0 1 1 2 4 5 6 6 7 7 8 9 11]
}
```

**计数排序**

```go
// 桶排序之计数排序
// 应用范围较窄，如统计职工年龄等范围较小的场景
// 年龄的范围属于[0,200]，因此建立hash表用作次数索引
// arr[i]表示i岁的职工个数，遍历数组建立hash表后
// 取最小前缀和就是排序后的数组下标(降序排序)
```

#### 工程上排序的优化

比如快排中左右边界差较小时(<60)，直接使用插入排序，对于小样本排序更有优势。如

```go
func QuickSort(arr []int, left, right int) {
	if left >= right {
		return
	}
	if left>right-60{
		插入排序()
	}
    //快排操作
	QuickSort(左侧)
	QuickSort(右侧)
}
```

### 链表算法

自定义链表

```go
type Node struct {
    value int
    next  *Node
}
type listNode struct {
    Head *Node
}
```

```go
// 返回链表头
func InitialListNode() *listNode {
    return &listNode{Head: &Node{}}
}
// 链表尾部新增
func AppendListNode(h *listNode, value int) {
    cur := h.Head
    for cur.next != nil {
       cur = cur.next
    }
    cur.next = &Node{value, nil}
}
// 链表插入新增
func InsertListNode(h *listNode, index, value int) {
    cur := h.Head
    for i := 1; i < index && cur.next != nil; i++ {
       cur = cur.next
    }
    next := cur.next
    cur.next = &Node{value, nil}
    cur.next.next = next
}
// 链表打印
func PrintListNode(h listNode) {
    cur := h.Head.next
    for cur != nil {
       fmt.Print(cur.value, ",")
       cur = cur.next
    }
    fmt.Println()
}
// 链表删除指定值第一个结点
func DeleteListNode(h *listNode, value int) {
    cur := h.Head
    for cur.next != nil && cur.next.value != value {
       cur = cur.next
    }
    if cur.next.value == value {
       cur.next = cur.next.next
       return
    }
}
// 链表倒序
func ReverseListNode(h *listNode) *listNode {
    pre := h.Head.next
    cur := pre.next
    next := cur.next
    pre.next = nil
    for next != nil {
       cur.next = pre
       pre = cur
       cur = next
       next = cur.next
    }
    cur.next = pre
    return &listNode{Head: &Node{next: cur}}
}
```

**倒序链表，插入法逆序**

```go
func reverse(Node head) {
        if (head == null || head.next == null) {
            return;
        }
        Node cur = null;
        Node next = null;
        // 当前从第二个开始 2
        cur = head.next.next;
        // 第一个节点变为尾
        head.next.next = null;
        while (cur != null) {
            next = cur.next;
            cur.next = head.next;
            head.next = cur;
            cur = next;
        }
    }
```

**链表判断环存在**

* 哈希表，将所有路过的结点放入哈希表并自增其值，当第二次路过同一结点时，该节点为入环结点。若全遍历后无重复结点则无环。额外空间复杂度O(N)
* 快慢指针，快指针走两步，慢指针走一步，第一次快慢指针相遇后将快指针置于链表中第一个结点，然后快慢指针都走一步，再次相遇为入环结点。

```go
// 返回第一个入环结点，若无环返回nil
func LoopEntryNode(h *listNode) *listNode {
    // cur 为环上第一个结点
    cur := h.Head.next
    if cur == nil || cur.next == nil || cur.next.next == nil {
        // 环为空或者环中仅含一个结点或环中两个结点不互链
        return nil
    }
    fast := cur.next.next
    slow := cur.next
    // 第一次相遇 fast走两步 slow走一步
    for fast != slow {
        if fast.next.next == nil || slow.next == nil {
            // 遍历链表无环
            return nil
        }
        fast = fast.next.next
        slow = slow.next
    }
    // 第二次相遇,fast每次走一步
    fast = cur
    for fast != slow {
        fast = fast.next
        slow = slow.next
    }
    return &listNode{fast}
}
```



### 二叉树

#### 先序遍历（中左右）(深度优先遍历)

非递归实现

![image-20231212131911337](D:\img\image-20231212131911337.png)

```go
func preOrder(n *Node) {
	if n != nil {
		stack := []Node{}
		stack = append(stack, *n)
        // 如上图
		for len(stack) != 0 { // 栈不为空
			// 栈顶元素出栈并操作
			pop := stack[len(stack)-1]
			stack = stack[:len(stack)-1]
			fmt.Println(pop.value)
			if pop.right != nil {
				stack = append(stack, *pop.right)
			}
			if pop.left != nil {
				stack = append(stack, *pop.left)
			}
		}
	}
}
//  方法二
func preorderTraversal(root *TreeNode) []int {
    stack := make([]*TreeNode,0)
    value := make([]int,0)
    node := root
    for len(stack)>0 || node != nil{
        for node!=nil{ // 遍历节点至最深的左孩子并入栈
            value = append(value,node.Val)
            stack = append(stack,node)
            node = node.Left
        }
        node = stack[len(stack)-1].Right
        stack = stack[:len(stack)-1] // 节点出栈遍历右孩子 
    }
    return value
}
```

递归实现

```go
// 二叉树简单结构
type node struct{
	value int 
	left node
	right node
}
```

```go
// 先序遍历
func preOrder(n node){
    if node==nil{
        // 边界条件
        return 
    }
    handle(n) // 打印或处理该结点
    preOrder(n.left) // 递归左孩子
    preOrder(n.right) // 递归右孩子
}
```

#### 中序遍历（左中右）

![image-20231212140848931](D:\img\image-20231212140848931.png)

非递归实现:和先序遍历最大区别是出栈时候才会访问值

```go
// 按照左边界遍历
func midOrder(n *Node) {
	if n != nil {
		// 将所有左子树顺序压栈，第一个左子树最先进栈
		stack := []Node{}
		for len(stack) != 0 || n != nil {
			if n != nil {
				// 压栈左子树直到没有左子树存在
				stack = append(stack, *n)
				n = n.left
			} else {
				// 栈顶元素出栈并打印
				n = &stack[len(stack)-1]
				stack = stack[:len(stack)-1]
				fmt.Println(n.value)
				// 对该元素右树重复上述操作
				n = n.right
			}
		}
	}
}

func inorderTraversal(root *TreeNode) []int {
    ans := make([]int,0)
    stack := make([]*TreeNode,0)
    node := root
    for len(stack) > 0 || node != nil{
        for node != nil{// 遍历到最左的最深孩子
            stack = append(stack,node)
            node = node.Left
        }
        //无左孩子 出栈访问
        node = stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        ans = append(ans,node.Val)
        node = node.Right
    }
    return ans
}
```

递归实现

```
// 先序遍历
func midOrder(n node){
    if node==nil{
        // 边界条件
        return 
    }
    midOrder(n.left) // 递归左孩子
    handle(n) // 打印或处理该结点
    midOrder(n.right) // 递归右孩子
}
```

#### 后序遍历（左右中）

后序遍历是将前序遍历(中左右)的顺序改变为中右左，再将结果倒叙即可

关键在于

```go
for len(stack)>0 || node != nil{
        for node!=nil{ // 遍历节点至最深的左孩子并入栈
            value = append(value,node.Val)
            stack = append(stack,node)
            node = node.Right
        }
        node = stack[len(stack)-1].Left
        stack = stack[:len(stack)-1] // 节点出栈遍历右孩子 
    }

func reverse(nums []int){
    l,r := 0,len(nums)-1
    for l<r{
        nums[l],nums[r] = nums[r],nums[l]
        l++
        r--
    }
}
```

#### 层序遍历(宽度遍历)

![image-20231212145054357](D:\img\image-20231212145054357.png)

只有非递归写法

队列：先入结点先出，对于出队列的结点(处理)，先将左入队列再将右入队列。

```go
func wideOrder(n *Node){
    // 层序遍历
    if n==nil{
        return
    }
    queue := []Node{}
    queue = append(queue,*n)
    for len(queue)!=0{
        // 队头元素cur出队列
        cur := queue[0]
        queue = queue[1:]
        fmt.Println(cur)
        // 遍历其左右孩子
        if cur.left!=nil{
            queue = append(queue,cur.left)
        }
        if cur.right!=nil{
            queue = append(queue,cur.right)
        }
    }
}
```

#### 搜索二叉树

>左孩子的值均小于该节点，右孩子均大于该节点。若中序遍历结果为升序序列则为BST(binary search tree)。

```go
// 递归写法
// 维护全局遍历preValue作为左树的最大值，每次遍历结点的值应该大于该值。
var preValue int = 0
func CheckBST(n *Node)bool{
    if n==nil{
        return true
    }
    if CheckBST(n.left)==false{
        // 递归左子树
        return false
    }
    if n.value <= preValue{
        // 判断当前结点满足升序要求
        return false
    }else{
        preValue = n.value
    }
    return CheckBST(n.right)
}
```

#### 完全二叉树

>要么所有层都是满的，要么最后一层不满且从左到右开始变为不满

**层序遍历**该树，如果1)存在结点只有右孩子无左孩子，不是CBT。2)第一个无孩子或只有左孩子的结点即为叶子，其后面遍历到的结点都是叶子(无孩子)。满足两点即为CBT

```go
func isCBT(n *Node)bool{
    if n==nil{
        return true
    }
    leaf := false // 是否遇到过孩子不全的结点
    queue := []Node{}
    queue = append(queue,*n)
    for len(queue)!=0{
        // 出队列
        cur := queue[0]
        queue = queue[1:]
        
        // 左右孩子
        l := cur.left
        r := cur.right
        
        if r!=nil&&l==nil || leaf&&(l!=nil||r!=nil){
            // 1.有右无左孩子
            // 2.遇到过叶结点后遇到 非叶结点(存在孩子)
            return false
        }
        
        if l!=nil{
            queue = append(queue,cur.left)
        }
        if r!=nil{
            queue = append(queue,cur.right)
        }
        if l==nil&&r==nil{
            leaf = true
        }
    }
    return true
}
```

#### 平衡二叉树

>左右子树高度差不超过1

<img src="D:\img\image-20231212160124398.png" alt="image-20231212160124398" style="zoom:50%;" />

对于平衡二叉树的任意子树，其左子树和右子树都是平衡二叉树，且左右字数高度差小于1，对于子树的信息要求为{是否为BBT和高度}。

```go
// 递归结构如下：
type returnType struct{ // 子树返回信息
    isBBT bool // 是否为平衡二叉树
    height int // 树的高度(深度)
}

func process(n *Node) returnType{
    if (n==nil){
        // 边界条件：空树
        return returnType{true,0}
    }
    leftData := process(n.left)
    rightData := process(n.right)
    
    isBalanced := leftData.isBBt&&rightData.isBBt&&abs(leftData.height-rightData.height)<=1
    height := max(leftData.height,rightData.height)+1
    
    return returnType{isBalanced,height}
}

func isBalanced(n Node)bool{
    return process(n).isBalanced
}
```

**对于递归问题普遍解法：**

需要什么左右子树的信息，取两者信息的并集作为递归返回数据结构

遍历到某一子树后，先递归其左右子树拿信息。再将信息处理判断，返回它的数据结构。如BST算法思想可如下

![image-20231212163240520](D:\img\image-20231212163240520.png)

判断左右子树是否为BST,返回左子树最大值和右子树最小值。因此返回数据结构为{min,max,isBST}

对于某个结点，判断其是否为BST(左右都为BST，该点值大于左树max，小于右树min)，计算其最大最小值，返回

边界条件设置为空树{0,0,true}

### 图

#### 通用数据结构

```go
// 点类型Node
type Node struct{
    value int 
    in int // 入度 初始0
    out int // 出度 初始0
    nexts []Node // 以node为起点有向边的下一个结点
	edges []Edge // 以node为起点的有向边
}
```

```go
// 边类型Edge
type Edge struct{
    weight int
    from Node
    to Node
}
```

```go
// Graph
type Graph struct{
    nodes map[int]Node // 点集 (序号=>Node)
    edges []Edge // 边集
}
```

#### 接口函数：边矩阵转通用

```go
func ConvertFromMatrix(arr [][]int){ // 二维数组表示边集
    // [weight, from节点 ,to节点]
	graph := Graph{}
    for i:0;i<len(arr);i++ {
        weight := arr[i][0]
        from := arr[i][1]
        to := arr[i][2]
        // 当from和to未在点集中时，添加
        if graph.nodes[from]==nil{
            graph.nodes = append(graph.nodes, from)
        }
        if graph.nodes[to]==nil{
            graph.nodes = append(graph.nodes, to)
        }
        fromNode := graph.nodes[from]
        toNode := graph.nodes[to]
        // 创建边
        newedge := Edge{weight,fromNode,toNode}
        // 修改入度、出度、起始点、边集
        fromNode.nexts = append(fromNode.nexts,toNode)
        fromNode.out++
        toNode.in++
        fromNode.edges = append(fromNode.edges,newedge)
        graph.edges = append(graph.edges,newedge)
    }
}
```

#### 宽度优先遍历BFS

优先访问一个节点的所有领点

```go
func bfs(node Node){
    if node==nil{
        return
    }
    // 队列用于顺序遍历
    // hash表用于记录已经遍历过的点
    queue := []int{}
    hashSet := map[node]bool{}
    queue.append(node)
    hashSet[node] = true
    for len(queue)>0{
        // 出队列
        cur := queue[0]
        queue = queue[1:]
        fmt.Println(cur.value)
        // 将相邻的未遍历的节点放入队列
        for next := range(cur.nexts){
            if hashSet[next]==false{
                hashSet[next] = true
                queue.append(next)
            }
        }
    }    
}
```

#### 深度优先遍历DFS

```go
func dfs(node Node){
    if node==nil{
        return
    }
    // 栈用于存储访问顺序
    stack = []int{}
    hashSet = map[Node]bool{}
    // 放入顶点并执行操作
    stack = append(stack,node)
    hashSet[node] = true
    fmt.Println(node.value)
    for len(stack)>0{
        // 出栈刚访问的顶点
        cur = stack[len(stack)-1]
        stack = stack[:len(stack)-2]
        // 依次访问该点所有的邻点
        for next:=range(cur.nexts){
            if !hashSet[cur]{
                // 将其邻点放入栈
                stack = append(stack,cur)
                stack = append(stack,next)
                // 读取邻点
                hashSet = append(hashSet,next)
                fmt.Println(next.value)
                break
                // 下一次弹出的点是最深的邻点，直到最深处访问完了，返回上一层查看有无邻点
            }
        }    
    }  
}
```

#### 拓扑逻辑排序

> 用于依赖文件的编译顺序，如A->B,C->A,此时需要按照C->A->B的顺序进行遍历，即从图最深的地方进行遍历。

```go
func sortedToplogy(graph Graph){
    // 记录每个节点的入度，对于入度为0的点，认为他是最底层的编译文件，也是整个项目的起始
    inMap := map[Node]int{}
    // 入度为0的点进入的队列，用于记录下一次需要编译的文件(节点)
    zeroQueue := []Node{}
    // 将图中的点遍历度数，0入度放入队列
    for node := range(graph.nodes){
        inMap[node] = node.in
        if node.in == 0{
            zeroQueue = append(zeroQueue,node)
        }
    }
    result := []Node{}
    for len(zeroQueue)>0{
        // 出队列最底层文件，加入到结果中
        cur := zeroQueue[0]
        zeroQueue = zeroQueue[1:]
        result = append(result,cur)
        // 将原图中所有次底层(最底层指向的节点)的入度减一
        // 将最新的入度为0的点进队列作为下一次访问节点
        for next := range(cur.nexts){
            inMap[next] = inMap[nexts]-1
       		if inMap[next] == 0{
            zeroQueue = append(zeroQueue,next)
        	}
        }
    }
    return result
}
```

#### 图的最小生成树算法

- Kruskal算法

>将边按权重排序，依次寻找最小权重的边加入，若成环则删除该边，继续寻找下一条，直到所有边遍历完。
>
>*注：存在并查集的方式可以减少查询的复杂度！在union和判断环中实现O(1)级别的时间复杂度*

***在 Go 中，使用 `map` 进行键值对存储时，对于结构体类型作为键的话，它需要满足可哈希的要求。在 Go 语言中，数组、切片、结构体等类型默认是不可哈希的，因此不能作为 `map` 的键直接使用。***因此，将setMap的映射设计为(节点value值=>节点集合)，这是在没有权重相等边的前提下。**需要指出的是，可以使用结构体指针作为map的键**

```go
// 由于涉及到环的查询，新加入的边的from和to若都在生成树的节点中，则该边会导致成环。加入边的操作会合并点集
// 思路：将原图中的每个顶点看作单独的集合，若有边被选中，将两个集合合成一个
type setMap map[int][]Node
func initial(nodes []Node) setMap {
	// 将所有的节点生成一个单独点的集合
	setmap := setMap{}
	for _, cur := range nodes {
		set := []Node{cur}
		setmap[cur.value] = set
	}
	return setmap
}
func isSameSet(from, to Node, setMap setMap) bool {
	// 判断成环
	fromSet := setMap[from.value]
	toSet := setMap[to.value]
	fmt.Println(fromSet, toSet)
    // 比较需要改写
	return fromSet == toSet
	//return false
}
func union(from, to Node, setMap setMap) {
	// 将to和from的集合结合成新的from集，再将to对应集合赋值为from的集合
	fromSet := setMap[from.value]
	toSet := setMap[to.value]
	for _, toNode := range toSet {
		fromSet = append(fromSet, toNode)
		setMap[toNode.value] = fromSet
	}
}
```

```go
import "container/heap"
type Heap []Edge
func (h Heap) Len() int           { return len(h) }
func (h Heap) Less(i, j int) bool { return h[i].weight < h[j].weight }
func (h Heap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *Heap) Pop() interface{} {
	// 弹出最后一个元素
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}
func (h *Heap) Push(x interface{}) {
	*h = append(*h, x.(Edge))
}
// 优先级队列初始化
func kruskalMST(graph Graph) []Edge {
	priortyQueue := &Heap{}
	nodelist := []Node{}
	for _, node := range graph.nodes {
		nodelist = append(nodelist, node)
	}
	setMap := initial(nodelist)
	heap.Init(priortyQueue)
	for edge := range graph.edges {
		heap.Push(priortyQueue, edge)
	}
	result := []Edge{}
	for priortyQueue.Len() > 0 {
		edge := heap.Pop(priortyQueue)
		if !isSameSet(edge.(Edge).from, edge.(Edge).to, setMap) {
			result = append(result, edge.(Edge))
			union(edge.(Edge).from, edge.(Edge).to, setMap)
		}
	}
	return result
}
```

* Prim算法

>1. 从源点出发，将所有与源点连接的点加入一个待处理的集合中
>2. 从集合中找出与源点的边中权重最小的点，从待处理的集合中移除标记为确定的点
>3. 将找到的点按照步骤1的方式处理
>4. 重复2，3步直到所有的点都被标记

```go
// 仅表达思想 仍然使用优先级队列进行排序 在这里仅用数组表示
func primMST(graph Graph){
    // 权值递减的边集合
    priorityQueue := []Edge{}
    // 标记已经选择过的点集合和边集合
    nodeSet := map[Node]bool{}
    result := map[Edge]bool{}
    for node := range(graph.nodes){
        // 随便选一个点开始，防止图非联通
        if !nodeSet[node]{
            nodeSet[node] = true
            for edge:= range(node.edges){
                // 解锁该点相邻的边
                priorityQueue.append(edge)
            }
            for len(priorityQueue)>0{
                // 弹出权值最小的边
                edge := priorityQueue[0]
                priorityQueue = priorityQueue[1:]
                toNode := edge.to // 找到该边对应的节点
                if !nodeSet[toNode]{ // 若没有访问过
                    nodeSet[toNode] = true
                	result[edge] = true
                    // 将其所有邻边加入队列
                    for nextEdge:=range(toNode.edges){
                    	priorityQueue.append(nextEdge)
                	}
                } 
            }
        }
    }
}
```

#### dijkstra算法

>计算图中某顶点到各点的最短路径

```go
func dijkstr(head Node) map[*Node]int {
	// 从头节点head到各点key的最小距离value
	distanceMap := map[*Node]int{}
	distanceMap[&head] = 0
	selectedNode := map[*Node]bool{} // 记录选择过的点
	minNode := getMinDistAndUnselected(distanceMap, selectedNode)
	for &minNode != nil {
		distance := distanceMap[&minNode]
		for _, edge := range minNode.edges {
			toNode := &edge.to
			if distanceMap[toNode] == 0 {
				distanceMap[toNode] = distance + edge.weight
			}
			distanceMap[toNode] = min(distanceMap[toNode], distance+edge.weight)
		}
		selectedNode[&minNode] = true
		minNode = getMinDistAndUnselected(distanceMap, selectedNode)
	}
	return distanceMap
}
```

``` go
// 接收节点集和已经访问过的节点
// 返回距离最近的节点
const INT_MIN = ^int(^uint(0) >> 1)
func getMinDistAndUnselected(distanceMap map[*Node]int, touchedNodes map[*Node]bool) Node {
	minNode := Node{}
	minDistance := INT_MIN
	for node, value := range distanceMap {
		if !touchedNodes[node] && value < minDistance {
			minNode = *node
			minDistance = value
		}
	}
	return minNode
}
```

#### LRU算法

***LRU(Least Recently Used)***最近最少使用，使用场景是`缓存全部存储在内存中，内存是有限的，因此不可能无限制地添加数据。假定我们设置缓存能够使用的内存大小为 N，那么在某一个时间点，添加了某一条缓存记录之后，占用内存超过了 N，这个时候就需要从缓存中移除一条或多条数据了。那移除谁呢？我们肯定希望尽可能移除“没用”的数据`

相对于仅考虑时间因素的 FIFO 和仅考虑访问频率的 LFU，LRU 算法可以认为是相对平衡的一种淘汰算法。LRU 认为，如果数据最近被访问过，那么将来被访问的概率也会更高。LRU 算法的实现非常简单，维护一个队列，如果某条记录被访问了，则移动到队尾，那么队首则是最近最少访问的数据，淘汰该条记录即可。

![image-20240613165214124](D:\img\image-20240613165214124.png)

这张图很好地表示了 LRU 算法最核心的 2 个数据结构

- 绿色的是字典(map)，存储键和值的映射关系。这样根据某个键(key)查找对应的值(value)的复杂是`O(1)`，在字典中插入一条记录的复杂度也是`O(1)`。
- 红色的是双向链表(double linked list)实现的队列。将所有的值放到双向链表中，这样，当访问到某个值时，将其移动到队尾的复杂度是`O(1)`，在队尾新增一条记录以及删除一条记录的复杂度均为`O(1)`。



### Golang包

#### container/heap实现优先级队列

```go
package main

import (
    "container/heap"
    "fmt"
)
// heap实现了五个接口 分别是
// sort.Interface中的
//  ->Len() int
//  ->Less(i, j int) bool
//  ->Swap(i, j int)
// Push(x any) // add x as element Len()
// Pop() any
// 实现了这五个方法才能使用go提供的heap
type Heap []int
func (h Heap) Len() int           { return len(h) }
func (h Heap) Less(i, j int) bool { return h[i] < h[j] }
func (h Heap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *Heap) Pop() interface{} {
    // 弹出最后一个元素
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}
func (h *Heap) Push(x interface{}) {
    *h = append(*h, x.(int))
}

func main() {
    h := &Heap{}
    heap.Init(h)
    fmt.Println(*h)
    heap.Push(h, 8)
    heap.Push(h, 4)
    heap.Push(h, 4)
    heap.Push(h, 9)
    heap.Push(h, 10)
    heap.Push(h, 3)
    fmt.Println(*h)
    for len(*h) > 0 {
       fmt.Printf("%d ", heap.Pop(h))
    }
    // 3 4 4 8 9 10
}
```

heap提供的方法不多，具体如下：

```go
h := &Heap{3, 8, 6}  // 创建IntHeap类型的原始数据
func Init(h Interface)  // 对heap进行初始化，生成小根堆（或大根堆）
func Push(h Interface, x interface{})  // 往堆里面插入内容
func Pop(h Interface) interface{}  // 从堆顶pop出内容
func Remove(h Interface, i int) interface{}  // 从指定位置删除数据，并返回删除的数据
func Fix(h Interface, i int)  // 从i位置数据发生改变后，对堆再平衡，优先级队列使用到了该方法
```

### 布隆过滤器

用于多并发中的数据处理，比如黑名单查询

特点：内存较哈希表更少，存在失误的几率(将白误报成黑，不会放出黑名单)，只能添加元素不能删除

>某公司需要查询用户跳转的URL是否合法，于是有100亿个URL黑名单，假设每个URL的长度限制为64Byte使用hash set存储数据(key,bool)，需要640G服务器内存用于查询

>位图bit map的概念：
>
>int[100]为整型数组，需要内存为4字节 * 8bit/字节 * 100 = 3200bit内存
>
>long[100]同理需要8字节*8bit/字节 * 100 = 6400bit
>
>而bit[100]仅需要100bit内存大小，约12.5byte。可以减少内存消耗w

**位图/比特数组的实现**：用基础类型拼凑

```java
int [] arr  = new int [10] // 32bit * 10 => 320bit
// arr[0]  int 0~31
// arr[1]  int 31~63
// arr[2]  int 64~95

int i = 178; 	//相取得第178个bit的数据
int numIndex = i/32; 	//定位数组下标位置
int bitIndex = i%32; 	//定位该下标数据中的bit位置

// 获取s为178位上的数据
// 将arr上的numIndex下标的数右移bitIndex位，即将178位移到最右边
// 然后将其与1相与(1 == 0000001)
int s = (  (arr[numIndex] >> bitIndex)  &1  ) 

//将178位的信息改为1
//将arr[numIndex]的数取出，将1左移bitIndex位取或，即置为1。
arr[numIndex] = arr[numIndex] |(1<< (bitIndex) )

//同理改为0，将1左移后取反，再取位与，即置为0
arr[numIndex] = arr[numIndex] & (  ~(1<< (bitIndex)  ) )    
```

**布隆过滤器原理：**创建一个长度为m的位图。将 url 分别取出，用 k 个 hash 映射得到散列值，再分别取模 m ，得到 k 个 [0,m-1] 的数，将对应位图中的位置涂黑(置为1或0)。

通过将所有信息标记在位图中后，检验黑名单是否包含可以：k 个hash函数处理url，仅当得到的所有位置都是被涂黑时表明该url为黑名单，否则白

参数 `位图长度m` , `hash函数数量k` , `黑名单数据大小n `   ，当m太小时，出现所有位图都被标黑，任何白名单都会被错报。当m太大时，浪费存储空间。

<img src="D:\img\image-20240312203646107.png" alt="image-20240312203646107" style="zoom: 33%;" />

固定数据大小，位图越长失误率越低

<img src="D:\img\image-20240312203835695.png" alt="image-20240312203835695" style="zoom: 50%;" />

固定位图长度和数据大小，可以选择适当的hash函数个数使得失误率最低。太多导致位图被占满误报。适当的个数可以使得失误率下降

三者关系：

先确定位图长度m，根据黑名单数据量和误码率要求，将m向上取整数

![image-20240312204140100](D:\img\image-20240312204140100.png)

选择最优的hash个数k

![image-20240312204153055](D:\img\image-20240312204153055.png)

固定k，若内存可扩大，即m可以再大一点，则真实失误率如下

![image-20240312204214736](D:\img\image-20240312204214736.png)

当m在28G大小的时候，取13个hash函数处理可以达到十万分之六的错误率

### 一致性哈希

用于分布式存储服务器，将数据存在哪台服务器上的算法。有负载均衡的作用。目的是将所有数据均匀的存储到各个服务器上。

取MD5算法作为hash算法，将[ 1 , 2^64 -1]的散列结果看成一个环，将机器的标识符看作hash输入，结果放到环上用作存储位置。数据取hash，放到结果更近的服务器。

传统哈希映射的问题：将数据hash处理后得到的结果，距离哪台机器的hash值顺时针方向更近存在那里，由于数据查询存在热度，容易导致某些服务器过载。此外，删除和添加服务器的复杂度相当于所有数据重新哈希映射再编排。

![image-20240312224107659](D:\img\image-20240312224107659.png)

解决方法：**虚拟节点**

m3管m3右侧部分，m1管m1右侧，m2同理，这样插入新服务器时可以将旁边的服务器数据直接分配

给m1\m2\m3服务器分别分配1000个字符串用于抢环。数据选择的服务器取决于距离字符串hash的距离大小顺时针最近的服务器。

### 并查集 

<img src="C:\Users\18262\AppData\Roaming\Typora\typora-user-images\image-20240313201429606.png" alt="image-20240313201429606" style="zoom:50%;" /><img src="D:\img\image-20240313201503210.png" alt="image-20240313201503210" style="zoom:50%;" />

正常解决思路：遍历矩阵中所有的点，如果遇见1，计数加一，将1和它连通的所有1感染为2，遇见0和2直接跳过。

<img src="D:\img\image-20240313201650555.png" alt="image-20240313201650555" style="zoom: 67%;" />

感染过程：如果当前位置[ i, j ]没有越界[N,M]的矩阵且矩阵中该点为1。感染该位置(置为2)，并依次感染上下左右符合感染条件的。

对于原矩阵，依次遍历所有元素，判断是否为1，若为1，进行感染操作且岛数加一，否则跳过。

**改进：如何利用多个CPU并行计算？ 仅理论**

并查集：同时满足两个操作时间复杂度O（1）。 *hash表/链表只能做到其中一者O(1)，另一者为O(n)*

- 判断两个集合是否同一个大集合
- 将两个集合进行合并

并查集是一种精巧实用的数据结构，它主要用于处理一些**不相交集合的合并问题，一些常用的用途有求连通子图，求最小生成树的Kruskal算法和求最近公共祖先（LCA）等**

```go
// 建立映射a=>A,用户给的数据a，逻辑上操作节点A，两者几乎一致
type unionSet struct {
	//parent[i] 表示i的父结点，根结点的父结点是本身
	parent map[int]int
	//count[i]可以用来保存本连通区域的结点个数
	count  map[int]int
}


//这里是使用了整数切片中的值构建并查集，有时也会使用下标构建
func NewUnionSet(nums []int) *unionSet {
	us := unionSet{}
	us.parent = make(map[int]int)
	us.count = make(map[int]int)
	for _, i := range nums {
		us.parent[i] = i
		us.count[i] = 1
	}
	return &us
}


//find 用于查找x所在连通区域的根结点
func (u *unionSet) Find(x int) int {
	if _, ok := u.parent[x]; !ok {
		return -1000000001 //这里定义的是一个非法数，视情况定义
	}
	//这里使用了路径压缩
	if u.parent[x] != x {
		u.parent[x] = u.Find(u.parent[x])
	}
	return u.parent[x]
}

//union 用于合并两个连通区域
func (u *unionSet) Union(p, q int) int {
	rootP := u.Find(p)
	rootQ := u.Find(q)
	if rootP == rootQ {
		return u.count[rootP]
	}
	u.parent[rootP] = rootQ
	u.count[rootQ] = u.count[rootP] + u.count[rootQ]
	return u.count[rootQ]
}

//判断两个集合是否属于同一个集合
func (u *unionSet) isSameSet(p,q int)bool{
    if _, ok := u.parent[p], _, ok2 := u.parent[q] ; !(ok&ok2) {
		return -1000000001 //这里定义的是一个非法数，视情况定义
    }
    return u.find(p)==u.find(q)
}

//除了基本方法外，还可以视情况增加其他属性和方法，例如用于计算一个连通区域的count属性或者增加一个connected方法判断两个结点是否连通等。
```

<img src="D:\img\image-20240313211636342.png" alt="image-20240313211636342" style="zoom: 25%;" /><img src="C:\Users\18262\AppData\Roaming\Typora\typora-user-images\image-20240313211701878.png" alt="image-20240313211701878" style="zoom:25%;" />

对于左侧原始矩阵，理论上只存在一个岛。但是将矩阵划分为两部分分给两个CPU计算后发现存在四个岛。这时需要并查集计算有哪些边界可以合并。

<img src="D:\img\image-20240313211824161.png" alt="image-20240313211824161" style="zoom:50%;" />

将原始感染点以及它感染的记为一个单独集合A/B/C/D，比较边界处非同一个集合是否可以合并，若可以，岛的数量减一，将其合并。如第一行A,C不同根，则将C挂在A下，同理将第二行B挂在{A,C}下,第三行将D挂在{A,B,C}下，第四行发现AC同集合岛数量为4-1-1-1 = 1.

对于多个CPU处理，需要对四个边界分别计算并查集合并

<img src="D:\img\image-20240313212226366.png" alt="image-20240313212226366" style="zoom:50%;" />





## MySQL

### DML语言（数据操作）

启动MySQL服务端

>base:C:\Software\MYSQL\bin>
>
>先注册为windows服务
>
>"C:\Software\MYSQL\bin\mysqld"  --install
>
>net start mysql 启动
>
>net stop mysql 关闭

启动MySQL客户端

> mysql -h主机名 -u用户名 -p密码 -P端口号
>
> mysql -hlocalhost -uroot -p123456 -P3307

#### **创建表**

```mysql
use mysql;
show databases;
show tables;

CREATE TABLE person_info(
 id INT NOT NULL auto_increment,
 name VARCHAR(100) NOT NULL,
 birthday DATE NOT NULL,
 phone_number CHAR(11) NOT NULL,
 country varchar(100) NOT NULL,
 PRIMARY KEY (id),
 KEY idx_name_birthday_phone_number (name, birthday, phone_number)
);

show columns from person_info   #show create table <table_name> \G
select * from <table_name>;
```

>**varchar 和 char的区别**
>
>char 是固定长度类型，无论存储的字符串多长，都分配固定的空间，如char(10)。访问速度会更快，适用于存储长度固定的字段，如手机号，国家代码等长度不变的字段。最长255字符。
>
>varchar是可变长度字符类型，根据存储的字符串长度动态分配内存，varchar(10)在存储'abc'时只会分配三个字符空间和一个用于存储长度的信息，访问速度稍慢。最大长度65535字符。

#### **插入操作**

```mysql
# INSERT INTO 表名[(字段1,字段2,字段3,...)] VALUES('值1','值2','值3')
# 在不包含保留字或特殊字符时，使用反引号``和不使用反引号没有区别。
INSERT INTO `users`(`username`, `password`, `avatar`) VALUES ('a', 'b', 'c')
# 等效于
INSERT INTO users(username, password, avatar) VALUES ('a', 'b', 'c');
```

#### **更新操作**

```mysql
# UPDATE 表名 SET column=value [,column=value2,...] WHERE [条件];
# ● column:要更改的数据列
# ● value 为修改后的数据 , 可以为变量 , 具体指 , 表达式或者嵌套的SELECT结果
# ● condition 为筛选条件 , 如不指定则修改该表的所有列数据
UPDATE user SET password = 'qwe',avatar='asd' WHERE username = 'a';
```

> **WHERE常用关键词： **
>
> `<> 或 !=`  不等于
>
> `BETWEEN`   在某个范围之内 `BETWEEN 5 AND 5`
>
> `AND`  和  `OR`  与或条件

#### **删除操作**

```mysql
例如删除 user 表： drop 是直接删除表信息，速度最快，但是无法找回数据
drop table user;

truncate 是删除表数据，不删除表的结构，速度排第二，但不能与where一起使用
truncate table user;  例如删除user表的所有数据

delete 是删除表中的数据，不删除表结构，速度最慢，但可以与where连用，可以删除指定的行
delete from user where user_id = 1; 删除user表的指定记录

# 如果有外键约束
show create table <table_name>; #查看外键约束引用的表 具体约束属性  
alter table <table_name> drop foreign <key/constraint> <columns_name>; #删除外键
drop table <table_name>;
```

三种方式的区别

*相同点*

truncate和不带where子句的delete，drop都会**删除表内的数据**；

drop，truncate都是DDL语句（数据定义语言），执行后会自动提交；

*不同点*

语句类型：delete语句是数据库操作语言（DML），truncate，drop是数据库定义语言（DDL）；
效率：一般来说   **drop > truncate> delete；**
是否删除表结构：**truncate和delete 只删除数据不删除表结构**，truncate 删除后将重建索引（新插入数据后id从0开始记起），而 delete不会删除索引 （新插入的数据将在删除数据的索引后继续增加），drop语句将删除表的结构包括依赖的约束，触发器，索引等；
安全性：drop和truncate删除时不记录MySQL日志，不能回滚，delete删除会记录MySQL日志，可以回滚；
返回值：delete 操作后返回删除的记录数，而 truncate 返回的是0或者-1（成功则返回0，失败返回-1）；

*delete 与 delete from 区别*

如果只针对一张表进行删除，则效果一样；如果需要联合其他表，则需要使用from

delete tb1 from tb1 m where id in (select id from tb2);

用法总结

- 希望删除表结构时，用 drop;

- 希望保留表结构，但要删除所有记录时， 用 truncate;

- 希望保留表结构，但要删除部分记录时， 用 delete。
  

### DQL语言（数据查询）

 #### **SELECT语法** 

- **[] 括号代表可选的 , {}括号代表必选得**

```mysql
SELECT [ALL | DISTINCT]
	{* | table.* | [table.field1[as alias1][,table.field2[as alias2]][,...]]} 
FROM table_name [as table_alias]
    [left | right | inner join table_name2] -- 联合查询 [WHERE ...] -- 指定结果需满足的条件
    [GROUP BY ...] -- 指定结果按照哪几个字段来分组 [HAVING] -- 过滤分组的记录必须满足的次要条件
    [ORDER BY ...] -- 指定查询记录按一个或多个条件排序
    [LIMIT {[offset,]row_count | row_countOFFSET offset}]; 
    -- 指定查询的记录从哪条至哪条
```

#### **指定查询字段**

```mysql
-- 查询表的全部数据
select * from <table_name>;

-- 查询指定字段 as取别名
select `username` as name from user;s
```

#### **条件查询**

```mysql
-- 示例SQL语句 WHERE
-- 查询result表的学号与学生成绩
SELECT `StudentNo`, `StudentResult` FROM result;
-- 查询成绩在95-100之间的
SELECT `StudentNo`, `StudentResult` FROM result WHERE StudentResult >=95 AND StudentResult <= 100;
-- AND也可以写成&&
SELECT `StudentNo`, `StudentResult` FROM result WHERE StudentResult >=95 && StudentResult <= 100;
-- 模糊查询(区间)
SELECT `StudentNo`, `StudentResult` FROM result WHERE StudentResult BETWEEN 95 AND 100;
-- 查询除了1000号同学的成绩
SELECT `StudentNo`, `StudentResult` FROM result WHERE NOT StudentNo = 1000;
-- NOT也可以写成!
SELECT `StudentNo`, `StudentResult` FROM result WHERE StudentNo != 1000;
```

#### **模糊匹配**

```mysql
-- ======= like =======
-- like结合 %(代表0到任意个字符) , _(一个字符)
-- 查询姓刘的同学
select `StudentNo`, `StudentName` from student where `StudentName` like '刘%';
-- 查询姓刘的同学，名字后面只有一个字的
select `StudentNo`, `StudentName` from student where `StudentName` like '刘_';
-- 查询姓刘的同学，名字后面只有两个字的
select `StudentNo`, `StudentName` from student where `StudentName` like '刘__';
-- 查询名字中间有嘉字的同学
select `StudentNo`, `StudentName` from student where `StudentName` like '%嘉%';

-- ======= in =======
-- 查询 1001，1002，1003号学员
select `StudentNo`, `StudentName` from student where StudentNo in (1001,1002,1003);
-- 查询 地址在北京，南京，河南洛阳的学生
select `StudentNo`, `StudentName`, `Address` from student where Address in ('北京', '南京', '河南洛阳');

-- ======= null/not null =======
-- 查询出生日期没有填写的同学
select `StudentName` from student where BornDate is null;
-- 查询出生日期填写了的同学
select `StudentName` from student where BornDate is not null;
-- 查询没有写家庭地址的同学(空字符串不等于null)
select `StudentName` from student where Address='' or Address is null;
```

#### 连接查询

>`join` 连接多个表返回符合条件的结果， `on`指定连接条件 。常见包括 `inner join` `left join` `right join`

```mysql
-- INNER JOIN 内连接 ： 返回两个表都有匹配关系的记录
SELECT * 
FROM table1 
INNER JOIN table2 ON table1.column=table2.column; -- 返回表一和表二中相等的行

-- LEFT JOIN 左连接 ： 返回左表匹配右表无匹配的记录
SELECT *
FROM table1
LEFT JOIN table2 ON table1.column = table2.column;-- 返回表一中所有记录，如果其column与表二匹配会显示对应的表二数据，不匹配显示NULL

-- RIGHT JOIN 右连接同理
```

>**示例**
>
>**`students` 表**
>
>| id   | name  |
>| ---- | ----- |
>| 1    | Alice |
>| 2    | Bob   |
>| 3    | Carol |
>
>**`grades` 表**
>
>| student_id | grade |
>| ---------- | ----- |
>| 1          | A     |
>| 2          | B     |
>
>执行以下 `JOIN` 查询：
>
>```mysql
>SELECT students.name, grades.grade
>FROM students
>LEFT JOIN grades ON students.id = grades.student_id;
>```
>
>结果：
>
>| name  | grade |
>| ----- | ----- |
>| Alice | A     |
>| Bob   | B     |
>| Carol | NULL  |
>
>这里使用了 `LEFT JOIN`，即使 `Carol` 在 `grades` 表中没有匹配的记录，`students` 表中的记录仍然被返回，并且 `grade` 字段显示为 `NULL`。

#### 分页

```mysql
SELECT <column> FROM <table_name> LIMIT [offset,]row_count;
-- row_count ：返回的记录数量
-- offset ：从哪条记录开始(默认是0)
```

>**分页查询**
>
>```
>offset = (page_number - 1) * page_size
>page_size = 每页显示的记录数量
>```
>
>在 Web 应用中可以根据用户的需求，分页加载数据，例如显示每页 10 条记录：
>
>```mysql
>SELECT * FROM users
>LIMIT 10 OFFSET 0;  -- 查询第1页
>SELECT * FROM users
>LIMIT 10 OFFSET 10; -- 查询第2页
>SELECT * FROM users
>LIMIT 10 OFFSET 20; -- 查询第3页
>```

#### 排序

```mysql
-- order by [字段] [desc/asc] , desc降序 ，asc升序s
select * from students order by name desc; -- 将students表中按姓名降序排序
```

#### MYSQL函数

```mysql
-- ======= 数学运算 =======
select abs(-8);-- 绝对值
select ceiling(9.4);-- 向上取整
select floor(9.4);-- 向下取整
select rand();-- 返回一个0~1之间的整数
select sign(10);-- 判断一个数的符号，负数返回1，正数返回1，0返回0

-- ======= 字符串函数 =======
select char_length('你猜我猜不猜你猜不猜'); -- 字符串长度
select concat('a', 'b', 'c'); -- 拼接字符串
select insert('我爱Golang',1,1,'SliverHorn'); -- 查询，从某个位置开始替换某个长度
select lower('SliverHorn'); -- 小写字母
select UPPER('SliverHorn'); -- 大写字母
select instr('SliverHorn', 'r'); -- 返回第一次出现的索引
select replace('SliverHorn说SliverHorn', '说', '-->'); -- 替换出现的指定字符串
select substr('SliverHorn', 4, 6); -- 返回制定本的子字符串(源字符串，截取的位置，截取的长度)
select reverse('SliverHorn');-- 字符串反转
select replace(StudentName, '周', '邹') as 名字 from student where StudentName LIKE '周%';-- 查询姓周的同学,改成邹

-- ======= 系统信息函数 =======
select version();-- 版本
select user();-- 用户

-- ======= 日期和时间函数 =======
select current_date();-- 获取当前日期
select now();-- 获取当前的时间

-- ======= 聚合函数 =======
select count(StudentName) from student; -- 对全部数据行的查询并计数。
select sum(r.StudentResult) from result as r; -- 求和所有分数
```

### 事务 Transaction

>- 事务就是将一组SQL语句放在同一批次内去执行
>- 如果一个SQL出错,则改批次内的所有SQL都将被取消执行
>- MySQL事务处理只支持InnoDB和BDB数据表类型

**ACID原则**

**原子性(Atomic)** 整个事务中的所有操作,要么全部完成,要么全部失败,不可能停滞在中间的某个环节.事务在执行过后才能中发生错误,会被回滚(Rollback) 到事务开始前的状态,就像这个事务从来没有执行过一样.

**一致性(Consist)**  一个事务可以封装状态改变(除非它是一个只读的).事务必须始终保持系统处于一致的状态,不管在任何给定的时间并发事务有多少.如果事务是并发多个,系统也必须如同串行事务一样的操作,其主要特征是保护性和不变性(Presering an lnvarriant).

> 以转账案例为例,假如有五个账户,每个账户余额是100元,那么五个账户总额是500元,如果在这个5个账户之间同事发生多个转账,无论并发多少个,比如在A与B账户之间转账5元,在C与D账户之间转账10元,在B与E之间转账15元.五个账户的总额也应该还是500,这就是保护性和不变性.

**隔离性(isolated)**  隔离状态执行事务,是它们好像是系统在给定时间内执行的唯一操作.如果有两个事务,运行在相同的时间内,执行相同的功能,事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统.这种属性有时称为串行化,为了防止事务操作的混淆,必须串行化或者序列化请求,使得在同一时间仅有一个请求用于同一数据.

**持久性(Durable)**  在事务完成后,该事务对数据库所作的更改,持久的保存在数据库之中,并不会被回滚

```mysql
-- mysql 是默认开启事务，自动提交
set autocommit = 0;-- 关闭自动提交
set autocommit = 1;-- 开启自动提交

-- 流程
set autocommit = 0;-- 手动处理事务
start transaction;-- 事务开启：标记一个事务的开始，从这个之后的sql都在同一个事务
commit;-- 提交：持久化到磁盘(成功)
rollback;-- 回滚：回到原来的样子(失败！)
set autocommit = 1;-- 事务结束开启自动提交
savepoint 保存点名称;-- 设置一个事务的保存点
rollback to savepoint 保存点名称;-- 回滚到保存点
release savepoint 保存点名称;-- 删除保存点
```

>**示例**
>
>*A在线买一款价格为500元商品,网上银行转账. A的银行卡余额为2000,然后给商家B支付500. 商家B一开始的银行卡余额为10000*
>
>```mysql
>-- 创建数据库shop和创建表account并插入2条数据 
>CREATE DATABASE `shop`CHARACTER SET utf8 COLLATE utf8_general_ci; USE `shop`;
>CREATE TABLE `account` (
>	`id` INT(11) NOT NULL AUTO_INCREMENT, `name` VARCHAR(32) NOT NULL,
>  `cash` DECIMAL(9,2) NOT NULL, PRIMARY KEY (`id`)
>) ENGINE=INNODB DEFAULT CHARSET=utf8 INSERT INTO account (`name`,`cash`)
>VALUES('A',2000.00),('B',10000.00)
>```
>
>```mysql
>-- 转账实现
>SET autocommit = 0; -- 关闭自动提交
>START TRANSACTION; -- 开始一个事务,标记事务的起始点 
>UPDATE account SET cash=cash-500 WHERE `name`='A'; 
>UPDATE account SET cash=cash+500 WHERE `name`='B'; 
>COMMIT; -- 提交事务
># rollback;
>SET autocommit = 1; -- 恢复自动提交
>```

### 索引

`主键索引(Primary Key)`  `唯一索引(Unique)` `常规索引(Index)` `全文索引(FullText)` 

- 索引不是越多越好
- 不要对经常变动的数据加索引
- 小数据量的表建议不要加索引
- 索引一般应加载查找条件的字段

>-- 我们可以在创建上述索引的时候，为其指定索引类型，分两类 hash类型的索引:查询单条快，范围查询慢 btree类型的索引:b+树，层数越多，数据量指数级增长(我们就用它，因为innodb默认支持它)
>-- 不同的存储引擎支持的索引类型也不一样
>
>- InnoDB 支持事务，支持行级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引; 
>- MyISAM 不支持事务，支持表级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引; 
>- Memory 不支持事务，支持表级别锁定，支持 B-tree、Hash 等索引，不支持 Full-text 索引; 
>- NDB 支持事务，支持行级别锁定，支持 Hash 索引，不支持 B-tree、Full-text 等索引; 
>- Archive 不支持事务，支持表级别锁定，不支持 B-tree、Hash、Full-text 等索引;

### 设计原则

**三大范式**

>- 第一范式的目标是确保每列的原子性,如果**每列都是不可再分的最小数据单元**,则满足第一范式
>
>- 第二范式要求**每个表只描述一件事情**。需要满足第一范式。
>- 如果一个关系满足第二范式,并且**除了主键以外的其他列都不传递依赖于主键列**,则满足第三范式.第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。



  ![image-20231212165353714](D:\img\image-20231212165353714.png)

- 连接管理

> 每当有一个客户端进程连接到服务器进程时，服务器进程都会创建一个线程来专门处理与这个 客户端的交互，当该客户端退出时会与服务器断开连接，服务器并不会立即把与该客户端交互的线程销毁掉，而 是把它缓存起来，在另一个新的客户端再进行连接时，把这个缓存的线程分配给该新客户端。这样就起到了不频 繁创建和销毁线程的效果，从而节省开销。

- 解析与优化

>1. 查询缓存
>
>   MySQL 服务器会把刚刚处理过的查询请求和结果缓存起来，如果下一次有一模一样的请求过来，直接从缓存中查找结果。
>
>   如果两个查询请求在任何字符上的不同（例如：空格、注释、大小写），都 会导致缓存不会命中。另外，如果查询请求中包含某些系统函数、用户自定义变量和函数、一些系统表，如 mysql 、information_schema、 performance_schema 数据库中的表，那这个请求就不会被缓存。
>
>   MySQL的缓存系统会监测涉及到的每张表，只要该表的结构或者数 据被修改，如对该表使用了 INSERT 、 UPDATE 、 DELETE 、 TRUNCATE TABLE 、 ALTER TABLE 、 DROP TABLE 或 DROP DATABASE 语句，那使用该表的所有高速缓存查询都将变为无效并从高速缓存中删除！
>
>2. 语法解析
>
>   因为客户端程序发送过来的请求只是一段文本而 已，所以 MySQL 服务器程序首先要对这段文本做分析，判断请求的语法是否正确，然后从文本中将要查询的表、 各种查询条件都提取出来放到 MySQL 服务器内部使用的一些数据结构上来。
>
>3. 查询优化
>
>   服务器程序获得到了需要的信息，比如要查询的列是哪些，表是哪个，搜索条件是什么等等，但 光有这些是不够的，因为MySQL 语句执行起来效率可能并不是很高， MySQL 的优化程序会对我们的语句 做一些优化，如外连接转换为内连接、表达式简化、子查询转为连接吧啦吧啦的一堆东西。优化的结果就是生成 一个执行计划，这个执行计划表明了应该使用哪些索引进行查询，表之间的连接顺序是啥样的。

- 存储引擎

>MySQL 服务器把数据的存储和提取 操作都封装到了一个叫 存储引擎 的模块里。
>
>我们知道 表 是由一行一行的记录组成的，但这只是一个逻辑上的概 念，物理上如何表示记录，怎么从表中读取数据，怎么把数据写入具体的物理存储器上，这都是 存储引擎 负责 的事情。为了实现不同的功能， MySQL 提供了各式各样的 存储引擎 ，不同 存储引擎 管理的表具体的存储结构 可能不同，采用的存取算法也可能不同。



### **快速查询**：B+树

在InnoDB数据页面中，可以由七个部分组成，包括

>1.InnoDB为了不同的目的而设计了不同类型的页，我们把用于存放记录的页叫做 数据页 。
>
>2.一个数据页可以被大致划分为7个部分，分别是
>
> - File Header ，表示页的一些通用信息，占固定的38字节。 
> - Page Header ，表示数据页专有的一些信息，占固定的56个字节。
> - Infimum + Supremum ，两个虚拟的伪记录，分别表示页中的最小和最大记录，占固定的 26 个字节。
> - User Records ：真实存储我们插入的记录的部分，大小不固定。 
> - Free Space ：页中尚未使用的部分，大小不确定。 
> - Page Directory ：页中的某些记录相对位置，也就是各个槽在页面中的地址偏移量，大小不固定，插 入的记录越多，这个部分占用的空间越多。 
> - File Trailer ：用于检验页是否完整的部分，占用固定的8个字节。
>
>3.每个记录的头信息中都有一个 next_record 属性，从而使页中的所有记录串联成一个 单链表 。 
>
>4.InnoDB 会为把页中的记录划分为若干个组，每个组的最后一个记录的地址偏移量作为一个 槽 ，存放在 Page Directory 中，所以在一个页中根据主键查找记录是非常快的，分为两步： 通过二分法确定该记录所在的槽。 通过记录的next_record属性遍历该槽所在的组中的各个记录。 
>
>5.每个数据页的 File Header 部分都有上一个和下一个页的编号，所以所有的数据页会组成一个 双链表 。
>
>6.为保证从内存中同步到磁盘的页的完整性，在页的首部和尾部都会存储页中数据的校验和和页面最后修改时 对应的 LSN 值，如果首部和尾部的校验和和 LSN 值校验不成功的话，就说明同步过程出现了问题。

知道了各个数据页可以组成一个 双向链表 ，而每个数据页 中的记录会按照主键值从小到大的顺序组成一个 单向链表 ，每个数据页都会为存储在它里边儿的记录生成一个 页目录 ，在通过主键查找某条记录的时候可以在 页目录 中使用二分法快速定位到对应的槽，然后再遍历该槽对 应分组中的记录即可快速找到指定的记录。页和记录的关系示意图如下：

![image-20240422200754307](D:\img\image-20240422200754307.png)

具体某一页中会包含一下信息：以数据主键为索引范围的最小和最大记录、各条数据的具体数值。需要指出的是，**页所指向的下条数据页的最小数值必须大于当前页的最大数值(主值)**

![image-20240422200953186](D:\img\image-20240422200953186.png)

而目录即索引，包括某一页的最小和页码信息，由很多条信息摘要组成一个目录页。同样目录页很多之后会被继续被上层的目录页所索引

![image-20240422201309255](D:\img\image-20240422201309255.png)

在 InnoDB 存储引擎中， **聚簇索引 **就是数 据的存储方式（所有的用户记录都存储在了 叶子节点 ），也就是所谓的索引即数据，数据即索引。

但上述的方法只能在搜索条件是主键值时才能发挥作用。如果想搜索其他非主键数据怎么办呢(非遍历)，我们可以多建几棵 B+ 树，不同的 B+ 树中的数据采用不同的排序规则。并且以该非主键数据列作为主键，将原主键作为略缩信息。**目录项记录中不再是 主键+页号 的搭配，而变成了 c2列+页号 的搭配。B+ 树的叶子节点存储的并不是完整的用户记录，而只是 c2列+主键 这两个列的值。**

![image-20240422202143068](D:\img\image-20240422202143068.png)

这种方式称为**二级索引**，它与原索引的不同指出在于，查到到数据位于某一页中之后也只能得到略缩的信息(索引的次主键和数据原主键)，所以我们必须再根据主键 值去聚簇索引中再查找一遍完整的用户记录(`回表`)。

### B+树使用



## Docker

### 基础操作

```bash
$ docker version # 安装完成查看
$ docker info
# 启动docker服务
$ sudo service docker start # service 命令的用法
$ sudo systemctl start docker # systemctl 命令的用法

#查看镜像文件image
$ docker image ls 
$ docker image rm [imageName]  # 删除 image 文件
```

>**Hello-world**
>
>```bash
>$ docker image pull hello-world
>$ docker image ls
>$ docker container run hello-world
># Hello from Docker!
># This message shows that your installation appears to be working    correctly.... ...
># 对于那些不会自动终止的容器，必须使用docker container kill 命令手动终止。
>$ docker container kill [containID]
>
># 列出本机正在运行的容器
>$ docker container ls
># 列出本机所有容器，包括终止运行的容器
>$ docker container ls --all
>```

### 其他有用的操作

**（1）docker container start**

前面的`docker container run`命令是新建容器，每运行一次，就会新建一个容器。同样的命令运行两次，就会生成两个一模一样的容器文件。如果希望重复使用容器，就要使用`docker container start`命令，它用来启动已经生成、已经停止运行的容器文件。

> ```bash
> $ docker container start [containerID]
> ```

**（2）docker container stop**

前面的`docker container kill`命令终止容器运行，相当于向容器里面的主进程发出 SIGKILL 信号。而`docker container stop`命令也是用来终止容器运行，相当于向容器里面的主进程发出 SIGTERM 信号，然后过一段时间再发出 SIGKILL 信号。

> ```bash
> $ docker container stop [containerID]
> ```

这两个信号的差别是，应用程序收到 SIGTERM 信号以后，可以自行进行收尾清理工作，但也可以不理会这个信号。如果收到 SIGKILL 信号，就会强行立即终止，那些正在进行中的操作会全部丢失。

**（3）docker container logs**

`docker container logs`命令用来查看 docker 容器的输出，即容器里面 Shell 的标准输出。如果`docker run`命令运行容器的时候，没有使用`-it`参数，就要用这个命令查看输出。

> ```bash
> $ docker container logs [containerID]
> ```

**（4）docker container exec**

`docker container exec`命令用于进入一个正在运行的 docker 容器。如果`docker run`命令运行容器的时候，没有使用`-it`参数，就要用这个命令进入容器。一旦进入了容器，就可以在容器的 Shell 执行命令了。

> ```bash
> $ docker container exec -it [containerID] /bin/bash
> ```

**（5）docker container cp**

`docker container cp`命令用于从正在运行的 Docker 容器里面，将文件拷贝到本机。下面是拷贝到当前目录的写法。

> ```bash
> $ docker container cp [containID]:[/path/to/file] .
> ```

**（6）docker container cp**

如果你想清理未使用的镜像，可以运行以下命令：

>```bash 
>$ docker image prune # 这样会删除未使用的镜像和未引用的中间层，释放空间。
>```



## Ubuntu

### conda

创建虚拟环境
conda create -n name python==3.9

激活环境
conda activate name

退出环境
conda deactivate

查看虚拟环境
conda info --envs

删除虚拟环境
conda remove -n name --all


删除所有的安装包及cache(索引缓存、锁定文件、未使用过的包和tar包)
conda clean -y --all

删除pip的缓存
rm -rf ~/.cache/pip

### linux 常见指令

`pwd` 是当前路径
`sudo apt remove example` 卸载
`sudo dpkg -i example` 安装deb后缀 
`dpkg -l | grep example` 查看已经安装的
`chmod 777 example` 修改文件权限


### 系统backup

`tar -cvpzf backup.tar.gz --exclude=/proc --exclude=/lost+found --exclude=/backup.tar.gz --exclude=/mnt --exclude=/sys --exclude=/media /`  系统的压缩文件快照制作



`-c`：新建备份文档
`-v`：显示详细信息
`-p`：保存权限
`-z`：gzip压缩
`-f`：指定压缩包名称路径(最后一个参数)
`-exclude`：不备份的目录

`tar -xvpzf backup.tar.gz -C /` 重写解压系统 

> 最好先rm -rf /tmp/root*  删除根目录文件





### 浏览和创建文件

**cat：一次性显示文件所有内容，更适合查看小的文件。**

cat file.name

【常用参数】

    -n显示行号。

**less：分页显示文件内容，更适合查看大的文件**。

less file.name

【快捷操作】

    空格键：前进一页（一个屏幕）；
    
    b 键：后退一页；
    
    回车键：前进一行；
    
    y 键：后退一行；
    
    上下键：回退或前进一行；
    
    d 键：前进半页；
    
    u 键：后退半页；
    
    q 键：停止读取文件，中止less命令；
    
    = 键：显示当前页面的内容是文件中的第几行到第几行以及一些其它关于本页内容的详细信息；
    
    h 键：显示帮助文档；
    
    / 键：进入搜索模式后，按 n 键跳到一个符合项目，按 N 键跳到上一个符合项目，同时也可以输入正则表达式匹配。

**mkdir 创建目录**

【**常用参数**】

- `-p`递归的创建目录结构`mkdir -p one/two/three`

tourch 创建文件

**cp 拷贝文件**

``` 
cp file file_copy --> file 是目标文件，file_copy 是拷贝出来的文件  
cp file one --> 把 file 文件拷贝到 one 目录下，并且文件名依然为 file  
cp file one/file_copy --> 把 file 文件拷贝到 one 目录下，文件名为file_copy  
cp *.txt folder --> 把当前目录下所有 txt 文件拷贝到 folder 目录下  
```

**ln  创建链接**

```
Linux 文件的存储方式分为 3 个部分，文件名、文件内容以及权限，其中文件名的列表是存储在硬盘的其它地方和文件内容是分开存放的，每个文件名通过 inode 标识绑定到文件内容。
Linux 下有两种链接类型：硬链接和软链接。
```

硬链接

    使链接的两个文件共享同样文件内容，就是同样的 inode ，一旦文件 1 和文件 2 之间有了硬链接，那么修改任何一个文件，修改的都是同一块内容，它的缺点是，只能创建指向文件的硬链接，不能创建指向目录的（其实也可以，但比较复杂）而软链接都可以，因此软链接使用更加广泛。
    ln file1 file2  --> 创建 file2 为 file1 的硬链接  

软链接就类似 windows 下快捷方式。

```
ln -s file1 file2  
```

**chmod：修改访问权限。**

```
chmod 740 file.txt  
```

【**常用参数**】

- `-R`可以递归地修改文件访问权限，例如`chmod -R 777 /home/lion`

其中d rwx r-x r-x表示文件或目录的权限。

- 它是一个文件夹；所有者具有：读、写、执行权限；群组用户具有：读、执行的权限，没有写的权限；其它用户具有：读、执行的权限，没有写的权限。

  d ：表示目录，就是说这是一个目录，普通文件是 - ，链接是 l 。
  r ：read表示文件可读。
  w ：write表示文件可写，一般有写的权限，就有删除的权限。
  x ：execute表示文件可执行。

  - ：表示没有相应权限。

| 权限 | 数字 |
| ---- | ---- |
| r    | 4    |
| w    | 2    |
| x    | 1    |

用字母来分配权限

    u：user 的缩写，用户的意思，表示所有者。
    
    g ：group 的缩写，群组的意思，表示群组用户。
    
    o ：other 的缩写，其它的意思，表示其它用户。
    
    a ：all 的缩写，所有的意思，表示所有用户。
    
    + ：加号，表示添加权限。
    
    - ：减号，表示去除权限。
    
    = ：等于号，表示分配权限。

> chmod u+rx file --> 文件file的所有者增加读和运行的权限  
> chmod g+r file --> 文件file的群组用户增加读的权限  
> chmod o-r file --> 文件file的其它用户移除读的权限  
> chmod g+r o-r file --> 文件file的群组用户增加读的权限，其它用户移除读的权限  
> chmod go-r file --> 文件file的群组和其他用户移除读的权限  
> chmod +x file --> 文件file的所有用户增加运行的权限  
> chmod u=rwx,g=r,o=- file --> 文件file的所有者分配读写和执行的权限，群组其它用户分配读的权限，其他用户没有任何权限  



**tar 创建一个tar归档。**

    tar -cvf sort.tar sort/ # 将sort文件夹归档为sort.tar  
    tar -cvf archive.tar file1 file2 file3 # 将 file1 file2 file3 归档为archive.tar  

常用参数

    -cvf 表示 create（创建）+ verbose（细节）+ file（文件），创建归档文件并显示操作细节；
     
    -tf  显示归档里的内容，并不解开归档；
    
    -rvf 追加文件到归档，tar -rvf archive.tar file.txt；
    
    -xvf 解开归档，tar -xvf archive.tar。

**gzip / gunzip “压缩 / 解压” 归档**，默认用gzip命令，压缩后的文件后缀名为.tar.gz。

可以用 tar 命令同时完成归档和压缩的操作，就是给 tar 命令多加一个选项参数，使之完成归档操作后，还是调用gzip 或 bzip2命令来完成压缩操作。

**zcat、zless、zmore**

之前讲过使用cat less more可以查看文件内容，但是压缩文件的内容是不能使用这些命令进行查看的，而要使用zcat、zless、zmore进行查看。

**zip/unzip**

“压缩 / 解压” zip 文件（ zip 压缩文件一般来自 windows 操作系统）。

>unzip archive.zip # 解压 .zip 文件  
>unzip -l archive.zip # 不解开 .zip 文件，只看其中内容  
>
>zip -r sort.zip sort/ # 将sort文件夹压缩为 sort.zip，其中-r表示递归  

### redis

**Starting and stopping Redis in the background**

You can start the Redis server as a background process using the `systemctl` command. This only applies to Ubuntu/Debian when installed using `apt`, and Red Hat/Rocky when installed using `yum`.

```bash
sudo systemctl start redis
```

To stop the server, use:

```bash
sudo systemctl stop redis
```

**Connect to Redis**

Once Redis is running, you can test it by running `redis-cli`:

```bash
redis-cli -h 127.0.0.1 -p 6379 -a 123456
# -a 表示密码 可以后续使用auth 123456登陆
```

Test the connection with the `ping` command:

```bash
127.0.0.1:6379> ping
PONG
```

