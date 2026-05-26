# 概述

`Go`最初名为`GoLang`，是由`google`发布的一种静态、强类型、编译型、并发型编程语言，尤其适合云原生，微服务，分布式系统等领域。

## 特点

- **简洁高效：**语法类似C语言，编写体验类似Python
- 并发支持
- 跨平台支持
- 标准库强大，内置Http，加密，json等功能库，方便web开发。

# `go`

`go`是Go 语言官方提供的命令行工具，用于编译、运行、构建和管理 Go 项目。

`go`的命令格式类似于`docker`，格式为

```
go <subCommand> [arguments]
```

<h4>查看帮助文档</h4>

```
go help [command/topic]
```

<h2><code>go env</code></h2>

查看`go`命令行工具相关环境变量。

```
go env 
```

```
go env -w GOPROXY=https://goproxy.cn,direct
```

```
go env -w GOPATH=path
```

- `GOPATH`是`Go`默认的工作目录，用于存放源码，依赖和工具。

<h2><code>go mod</code></h2>

 Go 的模块管理工具，用于定义模块、管理依赖，并提供 import 路径解析能力。

在当前目录初始化一个模块，生成`go.mod`文件

```
go mod init <模块名>
```

<h2><code>go get</code></h2>

在模块中添加并下载依赖。

```
go get 依赖URL
```



# 命名规范

## 访问权限

`Go`中通过变量，接口名、结构体名、方法名，结构体字段名等首字母的大小写定义其可见性。

- 首字母大写表示public（可导出），其他`package`可以直接访问
- 首字母小写 = private（包内私有），其他`package`无法访问

# 变量

## 变量声明

`Go`是一个强类型的语言，每一个变量都具有其类型。`Go`提供了三种声明变量的方式：

**声明变量，并指定类型**

```go
var 变量名 type
```

- 如果不指定类型，编译会报错

**声明变量并赋值**

此时编译器可根据字面量自动推断出变量类型，无需显示指定类型。

```go
var 变量名 type = 值

var 变量名 = 值

变量名 := 值
```

**同时声明多个变量并赋值**

```
var v1,v2... = value1,value2...
var v1,v2 ... type = value1,value2...
v1,v2 := value1,value2
```

## 变量类型

根据变量声明的位置，可将变量分为两类：

- **全局变量：**声明在方法外，可以在任何地方使用。
  - 声明全局变量并赋值时，必须使用`var`关键字，不能使用`:=`。
- **局部变量：**局部变量声明在方法内，仅能在本方法中使用

局部变量如果声明后没有使用，编译时会直接报错；而全局变量声明后没有使用，仅会警告

## 常量

常量是特殊的变量，它必须在声明时赋值，并且赋值后无法再修改常量的值

```
const v1 type =  value2
```

- `const`只能定义编译期常量，也就是常量值必须是编译期可确定的常量表达式，例如整数、浮点数、布尔值、字符串以及由常量表达式计算出的值。

# 规范

- `Go`要求：一个目录就是一个`package`，同一个目录下所有的`go`文件必须属于同一个`package`。
- `:=`貌似并不是表示声明并赋值
- `Go`强制所有流程控制语句均使用`{}`，以避免歧义

# 数据类型

- `Go`中通过预定义的标识符`nil`表示指针、`channel`、`function`、`interface`、`map`或`slice`类型的零值。

## 基本数据类型

### 整数型

`Go`中的整数类型有很多种

|                 |      |
| :-------------: | ---- |
| `uint8`/`byte`  |      |
|    `uint16`     |      |
| `uint32`/`rune` |      |
|    `uint64`     |      |
|     `uint`      |      |
|     `int8`      |      |
|     `int16`     |      |
|     `int32`     |      |
|     `int64`     |      |
|      `int`      |      |

- 默认的整数类型为`int`
- `uXXX`表示无符号，只能存非负数
- `int`和`uint`的大小取决于运行的平台

### 浮点型

`Go`支持两种浮点型数

|           |      |
| :-------: | ---- |
| `float32` |      |
| `float64` |      |

### 字符型

字符在计算机中本质是一个整数，在`Go`中，具有两种字符型，它们本质上是整数型的别名。

|        |                         |
| :----: | :---------------------: |
| `byte` | 单字节字符，对应`uint8` |
| `rune` | 多字节字符，对应`int32` |

- 在`Go`中，使用`UTF-8`编码存储字符。

### 布尔型

布尔型只有`true`和`false`两个值

- `Go`中不允许整数型与布尔型相互转换

------

- 如果对一个变量只声明而未赋值，则其默认为为对应类型的零值，数值型为`0`,布尔型为`false`，字符串型为`""`
- `Go`中，`"`表示单行字符串字面量，`'`表示字符字面量，<code>`</code>表示多行字符串字面量。

## 数组

数组是使用一段连续的地址空间，按照一定顺序存储**相同数据类型的多个元素**的集合，每个元素都可以通过**下标（索引）**进行访问。

<h4>特点</h4>

- **长度固定：**数组在声明后，其长度（元素个数）就已经确定，运行过程中通常不能改变。
- **类型相同：**数组中所有元素的数据类型必须一致。
- **连续存储：**所有元素在内存中是连续排列的。
- **支持随机访问：**可以通过下标（如 `arr[i]`）直接访问任意元素，时间复杂度为 O(1)。

<h4>数组相关操作</h4>

**声明**

```
var arr [len]type = [len]type{e1,e2...}
//快速声明字节数组
[]byte(字符串)
```

**索引访问元素**

```go
//获取元素
arr[index]
//修改元素
arr[index] = value
```

**获取数组长度**

```
len(arr)
```

## 切片

切片(Slice)是可变长的数组，其长度(存储的元素数量)可变，相较于数组更加灵活

- 切片同样使用索引访问其元素

**声明**

```
var slice []type
var slice []type = []type{e1,e2,...}
```

## `map`

`map`即为字典，是一个无序的键值对集合。`map`的`key`必须是基本数据类型,`value`可以是任意类型。

**声明**

```
var name map[keyType]valueType = map[keyType]valueType{
key1: value1,
key2: value2,
}
```

**获取值**

`map`支持通过`key`获得`value`

```go
//map[key]会返回两个值，第一个是value,第二个表示是否获取到元素(bool)
value,ok := map[key]
```



# 运算符

## 逻辑运算符

- `Go`中的逻辑运算符与`Java`一致，且具有逻辑短路特性。

# 条件控制语句

## 判断语句

<h3><code>if</code></h3>

```go
if 条件表达式  {

}

if 条件表达式 {
    
} else{
    
}

if 条件1 {
    
} else if 条件2 {
   
} else {
    
}
```

- `if`中也可以进行变量声明，此时变量只在`if`作用域内生效

  ```
  if a:=10;条件表达式 {
  
  }
  ```

<h4>卫语句</h4>

是一种**代码风格**，在`go`里应用非常多。核心是对异常或不满足条件的情况进行**提前判断并立即返回**，从而避免多层 if 嵌套，提高代码的可读性和可维护性。

```go
//层层嵌套
if user != nil {
    if user.Age > 18 {
        if user.Active {
            fmt.Println("可以操作")
        }
    }
}
//卫语句
if user == nil {
    return errors.New("user is nil")
}
if user.Age <= 18 {
    return errors.New("under age")
}
if !user.Active {
    return errors.New("user inactive")
}
fmt.Println("可以操作")
```

- 卫语句通常用于处理边界条件、错误情况或非法输入，通过对这些异常情况进行**提前判断并返回**，将其逐步拦截，从而使主逻辑始终运行在“正常路径”上，提升代码的可读性和可维护性。

<h3><code>switch</code></h3>

```go
//值匹配
switch (变量) {
    case value1:
    case value2:
	case v3,v4 :
    default:
}
//Go中的switch除了支持值匹配，还支持表达式，相当于 if-else 链
switch {
case 条件表达式1 :
case 条件表达式2 :
default:

}
```

**`fallthrough`**

`Go`中的`switch`默认不会出现穿透现象，但可以使用`fallthrough`手动穿透。

```
switch (变量) {
 case v1 : fallthrough
 case v2 :
}
```

穿透现象表现为当某个 `case` 成功匹配后，**从匹配位置开始顺序执行该 `case` 及其后续所有 `case` 中的代码**，而不再判断后续 `case` 的条件。

## 循环语句

`Go`中的循环语句只有`for`,但可以实现所有循环效果 

```go
//完整写法
for 初始化循环变量 ; 条件表达式 ; 更新循环变量 {
    
}
//类while写法，使用外部循环变量，在代码块中更新变量
for 条件表达式 {
    更新循环变量
}
//无限循环
for {
    
}

//for-range，快速遍历数组，切片,map等
for index,value := range arr {
    
}
//忽略变量
for _, v := range arr {
}
```

- `Go`中同样具有`continue`和`break`控制循环语句。

# 结构体

结构体是一种复合数据类型，可以组合多种数据类型形成一种新的数据类型

- `Go`中结构体变量中直接保存数据，而不是地址。
- `Go`中可以使用结构体的指针直接操作结构体，因为编译器会自动添加解引用，即将`p.name`转换为`(*p).name`
- 结构体的零值为每个字段的零值组成的结构体

```
//声明
type Name struct {
    field1 type1,
    field2 type2,
}
//为结构体绑定方法
func (varName StructName) FuncName() {

}
//默认为结构体绑定的方法为值传递，无法直接修改结构体中的字段
//想要修改结构体字段，需要使用引用传递
func(varName *StructName) FuncName(){

}
//使用
varName := StructName{field1: value1}
//调用方法
varName.FuncName()
```

- 如果字段类型为结构体，可以省略为`StructName`声明字段，此时字段名即为结构体名。

- Go中不存在继承，而是通过组合实现继承的功能；如果被组合的结构体的字段名与其他结构体不冲突，则可以直接使用`son.field`访问被组合结构体的字段。

  ```
  type Father struct {
  
  }
  type Son struct {
    Father
  }
  ```

## `tag`

可以通过为结构体的字段设置`tag`从而控制其序列化时的行为。

```go
type SturctName struct {
	field1 type1 `json:"tag"`,
	//-代表序列化时忽略此字段
    field2 type2 `json:"-"`
    //omitempty代表如果此字段为空，则忽略
    field3 type3 `json:"omitempty"`
}
```



# 函数

函数是一段封装了特定功能的可重用代码块，函数可以接收输入(参数) 并返回输出(返回值)。

**函数声明**

```go
//无返回值
func function_name (arg1 type1, ...) {

}
//有返回值,Go支持多返回值函数，如果只有一个返回值，括号可以省略
func func_name(arg1 type1, ...) (return_type1,return_type2...) {
    return v1,v2...
}

//Go允许为返回值命名，也就是命名返回值，此时返回值就是函数内部自带的变量，可以直接赋值
//且可以直接用裸 return 返回,Go会自动返回变量值。
func add(a, b int) (result int) {
    result = a + b
    return   // 👈 不写 result
}
```

- 如果连续多个参数类型一致，可以省略只写最后一个参数的类型
- 即使没有返回值，也可以使用`return`退出函数。

**可变参数**

`Go`支持为函数声明可变参数，函数内部会使用切片接收参数

```
func function_name (args ...type) {

}
```

## 匿名函数

`Go`不允许在函数中直接声明内部函数。但可允许匿名函数，即声明一个函数并使用一个变量接收它，之后可以使用变量名使用函数。

```go
//声明匿名函数
var function = func() (return_type) {
	
}
//使用匿名函数
function()
```

- 匿名函数不允许使用`const`变量接收
## 高阶函数
`Go`除了使用变量接受并调用函数，函数也可以作为函数参数，返回值，集合中的元素，此时参数类型为函数的签名。
```

```
### 闭包

## 值传递和引用传递

默认情况下，变量在函数调用中采用值传递，也就是在函数调用时会复制变量的值到另一块内存并进行操作，而不会影响原变量。
`Go`中也支持传递变量的引用，从而在函数中直接操作原变量。
通过`*type`声明引用类型，通过`&name`获取变量的引用，同时`*namePtr`用于解引用，即通过引用获取当前引用对应的变量。
```
func test(name *string)
var name = "123"
test(&name)
```
- 不只是函数调用时的参数，函数的返回值也是值传递

## `init`函数/`defer`函数

`init`函数是`Go`中一个特殊的函数，它会在`main`函数执行前被自动调用
**特点**
- 不能被其他函数主动调用
- 不能作为函数参数被传入
- 不能有参数和返回值
- 一个`Go`文件可以有多个`init`函数，会按照顺序依次从上往下执行。

`defer`函数又称为延迟调用函数，使用`defer`关键字在函数体中声明并注册，它会在函数`return`之前执行，常用于资源清理
多个`defer`函数会按照先声明后调用的顺序执行，也就是谁距离`return`近谁先执行。
```
defer func() {

}
defer 单条语句
```
- `defer`函数虽然在`return`前才执行，但函数体中只能访问在`defer`函数定义前声明的变量

# 类型别名

可以为类型起别名，此时无法为该别名绑定方法。

```
type Ailas = type
```

# 自定义类型

可以通过`type`自定义类型

```
type NewType type
```

同时可以为该新类型绑定方法，与结构体绑定方法一致

# 接口

接口用于规范结构体的行为，`Go`中的接口无需结构体显示的实现，而是基于发现的机制，当接口体中存在与接口中签名一致的函数，则被发现为此接口的实现类型。

- 接口也是一种数据类型

```
type interfaceName interface {
	funcName()
}
```

## 类型断言

通过断言来将接口转换为具体类型

```
switch obj.(type) {
	case type1 :
	case type2 :
}
//第一个参数接受断言的类型变量，第二个参数为是否为对应类型
c,ok := ojb.(Realtype)
//如果不接受第一个参数，如果断言失败，则会直接抛出异常
c := obj.(Realtype)
```

## 空接口

空接口就是不声明任何方法接口，这表明任何类型都可以被发现为空接口，也就是任何类型都实现了空接口。

```go
type emptyInterface interface{

}
//简洁写法
func Print(val any)
func Print(val interface{})
```

- `Go`中为空接口定义了别名`any`，可以直接使用`any`接收任意类型的值

## 接口嵌入

`Go`中的接口除了声明方法，也可以组合其他接口，类似于继承效果。

```go
//语法
type InterfaceName interface {
    OtherInterface1
    OtherInterface2
}
//示例
type Reader interface {
	Read([]byte) (int, error)
}

type Writer interface {
	Write([]byte) (int, error)
}
//等价于此接口组合了内部其他接口的方法声明。
type ReadWriter interface {
	Reader
	Writer
}
```



# 协程

协程是Go的一大特色

```
go 函数调用
```

如果主线程结束，协程的任务即使没有执行完，也会结束，`go`提供了`wait`工具以进行同步

```
var wait sync.WaitGroup

wait.add(任务数)
每一个任务结束
wait.Done();
主线程执行
wait.Wait()
```

## `channel`

`channel`用于在协程中传递数据

```go
//第一个type表示这是一个channel，第二个表示channel中放哪种类型的数据
//创建一个长度为0的信道，如果信道满了，其他数据会等待
var chan1 chan int = make(chan int)
//向信道中传递数据
chanName <- value
//从信道中取数据
 value,ok <- chanName
//ok表示信道是否关闭，此时可以停止取数据
//信道需要手动关闭
close(chanName)
//信道也可以使用for循环获取数据
for value := range chanName {
    
}
```

## `select`

用于同时使用多个信道的场景，防止代码阻塞在某个信道的接收上

```go
select {
case value1 := <- chan1:{
	}
case value2 := <- chan2 :{

}
}
```

## 超时处理

```go
select {
	case value1 <- chan1 : {

	}
    case <- time.After(超时时间) :{
        
    }
}
```

# 线程安全

### `Mutex`

互斥锁

```
var lock sync.Mutex

//加锁
lock.Lock()
//解锁
lock.Unlock()
```

### `Map`

一个线程安全的`Map`

```
var map = sync.Map{}
```

# 异常处理

# 

# 常用包

## `fmt`

```
fmt.Println()
```

```
fmt.Print()
```

```
fmt.Printf()
```

```
func SPrintf() string
```

## `json`

序列化为json格式字符串

```
json.Marshal()
```

## `net/http`

Go内置的 Http 服务开发包。

**监听端口**

```go
http.ListenAndServe("0.0.0.0:8080", nil)
//0.0.0.0表示监听所有 IPv4 地址，可以简写为
http.ListenAndServe(":8080", nil)
```

**注册路径对应的处理器函数**

```
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
```



# `Gin`

`Gin`是一个基于 Go 的轻量级 Web 框架，它基于 Go 内置的 `net/http`包开发，提供了高效、快速、易用的 Web 服务开发体验。

**`net/http`包的缺陷**

**安装依赖**

```
go get github.com/gin-gonic/gin
```

<h3>使用</h3>

1. 获得`Gin`引擎，通过引擎进行绑定路由等操作

   ```go
   //方法签名
   func Default(opts ...OptionFunc) *Engine
   
   r := gin.Default()
   ```

2. 挂载路由

   ```go
   func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes
   
   r.GET("/index",func (c *gin.Context)  {
   		
   })
   ```

3. 绑定监听端口

   ```
   r.Run(":8080")
   ```

## 运行模式

`Gin`有两种运行模式，即`debug`与`release`，`debug`模式会打印日志，而`release`不会，默认`gin`运行在`debug`模式下。

```
gin.SetMode("release")
```

## `Context`

`Context` 封装了一次 HTTP 请求处理过程中的上下文环境，包含请求数据、响应写入能力、中间件控制以及请求生命周期中的共享数据。

Gin 在每次收到请求时，都会创建一个 `*gin.Context`，并把它传给处理函数。

### 响应

<h4>响应<code>Json</code></h4>

<h4>响应<code>Html</code></h4>

1. 在`Go`中，如果想要响应`Html`需要先将`Html`文件进行加载。

   ```go
   //LoadHTMLXXX系列方法
   func (engine *Engine) LoadHTMLGlob(pattern string)
   func (engine *Engine) LoadHTMLFiles(files ...string)
   func (engine *Engine) LoadHTMLFS(fs http.FileSystem, patterns ...string) 
   
   //使用示例
   r.LoadHTMLGlob("./templates/*") //表示加载该路径下的所有HTML文件
   ```

2. 然后通过`Context`的`HTML`方法返回响应。

   ```go
   //第三个参数表示向HTML传递数据，可以使用Go自己的模板语言在HTML中获取数据并渲染
   func (c *Context) HTML(code int, name string, obj any)
   ```

   - 使用`{{.fieldName}}`访问具体数据，其中.表示模板本身

