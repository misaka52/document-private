## 环境搭建

### 安装

https://golang.google.cn/ go官网

https://studygolang.com/dl go中文学习社区，可下载最新版本go

配置环境变量

```go
// 查看go版本
go version
// 查看go环境变量
go env
// 更换go代理地址为国内，更快的下载依赖。代理含义，首选goproxy，否则再直接拉取
go env -w GOPROXY=https://goproxy.cn,direct
// go依赖管理，运用新model机制，下载包更快
go env -w GO111MODULE=on
// 安装goimports工具，自动管理依赖。在idea中和filewatcher配合使用
go get -v golang.org/x/tools/cmd/goimports
```

### IDE

##### idea+go插件

插件安装

- go
- file watcher。作用：自动导入包、自动格式化代码、代码提示
  - Perference，搜索inlay Hints勾选go，去掉函数参数提示

#### GoLand

## 基础语法

### 1. 变量定义

- 变量定义后必须使用，否则编译错误
- 变量用var修饰，可以通过:=省略var关键字
- 变量存在初始化值，int为0
- 打印：%q，在打印的字符串外打印双引号
- 全局变量仅在包内有效，且不能使用:=初始化

```go
func variablesZero() {
	var a int
	var b string
	fmt.Println(a, b)
	// %q 携带引号打印字符串
	fmt.Printf("%d %q\n", a, b)
}

func variablesInit() {
	// 变量初始化
	var a int = 1
	var b string = "ddd"
	fmt.Println(a, b)
}

func variablesInit2() {
	// 多变量初始化
	var a, b, c = 1, true, "str"
	fmt.Println(a, b, c)
}

func variablesInit3() {
	// :=多变量初始化
	a, b, c := 1, true, "str"
	fmt.Println(a, b, c)
}

var a = 1

// var b := 1，报错，:=不能用于全局变量
var (
	aa = 3
	bb = true
	cc = "fa"
)

func globalVariables() {
	fmt.Println(a, aa, bb, cc)
}

func main() {
	fmt.Println("hello world")
	variablesZero()
	variablesInit()
	variablesInit2()
	variablesInit3()
	globalVariables()
}
```

### 2. 变量类型

bool, string

(u)int, (u)int8, (u)int16, (u)int32, (u)int64, uintptr

Byte, rune(类似于char类型，大小4字节，用于应对各种编码保存字符)

float32, float64, complex64, complex128

Complex：复数，分为实部和虚部，比如3+4i，其中complex64实部和虚部分别均为float32。i*i=-1

```go
func euler() {
	a := 3 + 4i
	fmt.Println(cmplx.Abs(a))
	// 欧拉公式e^iπ + 1 = 0
	fmt.Println(cmplx.Exp(1i*math.Pi) + 1)
	fmt.Printf("%.3f\n", cmplx.Exp(1i*math.Pi)+1)
}
```

 **强制类型转换**

需要显示转换类型

```go
func triangle() {
	var a, b int = 3, 4
	var c int
	c = int(math.Sqrt(float64(a*a + b*b)))
	fmt.Println(c)

	var d float64 = 4.99
	// 直接抹除小数位，靠近0取整
	c = int(d)
	fmt.Println(int(c))
}
```

### 3. 常量和枚举

常量用const修饰。当不限定常量类型时，类似于文本替换动作，常量类型可以随意切换

```go
func consts() {
	const filename = "a.txt"
	const a, b = 3, 4
	var c int
	c = int(math.Sqrt(a*a + b*b))
	fmt.Println(c)
}
```

**枚举**

go中没有特定枚举关键字，通过一组const表示

iota表示后续变量自增，其值从0开始，也可以自定义值

```go
func enum() {
	// iota自增值，从0开始
	const (
		c = iota
		java
		_
		python
		golang
	)
  fmt.Println(c, java, python, golang)
	const (
		b = 1 << (10 * iota)
		kb
		mb
		gb
		tb
	)
	fmt.Println(b, kb, mb, gb, tb)
}
```

### 4. 条件语句

**if**

- if条件中没有括号
- 可以定义if作用域变量，变量仅包含在if作用域内

```go
func ifCondition() {
	const filename = "a.txt"
	if contents, err := ioutil.ReadFile(filename); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%s\n", contents)
	}
}
```

**switch**

- switch后可以没有条件表达书
- 每个case不需要break，自动携带。除非条件中显示使用fallthrough，相当于去掉break

```go
func switchCondition(score int) string {
	grade := ""
	switch {
	case score < 0 || score > 100:
		// 抛出异常
		panic(fmt.Sprintf("invalid score, %d", score))
	case score < 40:
		grade = "D"
		// 忽略等级D
		fallthrough
	case score < 60:
		grade = "C"
	case score < 80:
		grade = "B"
	case score < 100:
		grade = "A"
    default:
    gread = "E"
	}
	return grade
}
```

### 5. 循环

for可用来当做循环使用

一般for分为三部分组成：初始化、循环结束条件、循环递归

当for中不包含任何条件判断时，表示死循环

```go
func intToBin(num int) string {
	if num == 0 {
		return "0"
	}
	result := ""
	for ; num > 0; num /= 2 {
		tmp := num % 2
		result = strconv.Itoa(tmp) + result
	}
	return result
}

fun circulation() {
  for {
    fmt.Println("circul")
  }
}
```

### 6. 函数

- 函数可以返回多个结果
- 可以使用_接一些不需要的变量，后续也不可使用
- 支持可变参数
- 没有默认参数，方法重载重写

```go
func div(a, b int) (int, int) {
   return a / b, a % b
}

// 定义返回变量名称，但不推荐使用，变量一多容易部分变量未赋值
func div2(a, b int) (q, r int) {
	q = a / b
	r = a % b
	return
}

func operator(a, b int, op string) (int, error) {
	switch op {
	case "+":
		return a + b, nil
	case "-":
		return a - b, nil
	case "*":
		return a * b, nil
	case "/":
		return a / b, nil
	default:
		return 0, fmt.Errorf("unsupportted operation %s", op)
	}
}

// 函数式编程
func apply(op func(int, int) int, a, b int) int {
	// 反射获取函数名
	p := reflect.ValueOf(op).Pointer()
	name := runtime.FuncForPC(p).Name()
	fmt.Printf("call fun %s with args(%d,%d)\n", name, a, b)
	return op(a, b)
}

func pow(a, b int) int {
	return int(math.Pow(float64(a), float64(b)))
}

func main() {
	// _ 忽略结果
	q, _ := div(13, 4)
	fmt.Println(q)

	fmt.Println(apply(pow, 2, 5))

	// 匿名函数
	fmt.Println(apply(func(a, b int) int {
		return int(math.Pow(float64(a), float64(b)))
	}, 2, 5))
}

// 可变参数
func sum(values ...int) int {
	res := 0
	// num为下标，从0开始
	for num := range values {
		res += values[num]
	}
	return res
}
```

### 7. 指针

go中支持值传递和指针传递

```go
// 值传递
func swap(a, b int) {
	a, b = b, a
}

// 引用传递。swap2(&a, &b)
func swap2(a, b *int) {
	*a, *b = *b, *a
}
// 返回新值
func swap3(a, b int)(int, int) {
	return a, b
}
```

## 内建容器

### 1. 数组

- []int{}
- 使用range遍历

```go
func arr() {
	var arr1 [5]int
	arr2 := [3]int{1, 3, 5}
	arr3 := [...]int{2, 4, 6, 8, 10}
	var grid [3][2]int
	fmt.Println(arr1, arr2, arr3, grid)

 	// 不能使用++i
	for i := 0; i < len(arr3); i++ {
		fmt.Println(arr3[i])
	}

	for i, v := range arr3 {
		fmt.Println(i, v)
	}
}
```

**方法传递数组**

- 方法中传递数组使用的是值传递，即在传递时进行了数组拷贝，生成了一个新数组
- 数组传递时类型、数组大小必须完全相同
- 使用*[]int和&arr，来进行数组引用传递
- 数组床底太过于局限，一般不使用，而是使用切片

```go
func printArray(arr [5]int) {
	fmt.Println("printArray")
	arr[0] = 100
	for i, v := range arr {
		fmt.Println(i, v)
	}
}

func printArrayRef(arr *[5]int) {
	fmt.Println("printArrayRef")
	arr[0] = 100
	for i, v := range arr {
		fmt.Println(i, v)
	}
}

func main() {
  var arr1 [5]int
  printArray(arr1)
	fmt.Println(arr1)

	printArrayRef(&arr1)
	fmt.Println(arr1)
}
```

### 2. 切片（slice）

- slice就是数组的一个视图view
- arr[a:b]表示数组的切片，左开右闭（ab表示下标，从0开始）。ab均可以省略
- slice中存在两部分，0-len表示实际占用位置，len-cap表示最大容量（不可通过下标直接访问，但可重新切片获取）

```go
func updateSlice(arr []int) {
	arr[0] = 100
}

func slice() {
	arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7}
	fmt.Println(arr[2:6])
	// 引用更新
	updateSlice(arr[:])
	fmt.Println(arr)
	arr[0] = 0
	s1 := arr[2:6]
	fmt.Printf("s1=%v, len(s1)=%d, cap(s1)=%d\n", s1, len(s1), cap(s1))
	s2 := s1[3:5]
	fmt.Printf("s2=%v, len(s2)=%d, cap(s2)=%d\n", s2, len(s2), cap(s2))
}
// 输出
[2 3 4 5]
[100 1 2 3 4 5 6 7]
s1=[2 3 4 5], len(s1)=4, cap(s1)=6
s2=[5 6], len(s2)=2, cap(s2)=3
```

### 3. 切片操作

append：往切片中添加元素，目标切片len增大。原数组容量不变，新元素替换老元素

```go
func slice() {
  arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7}
  s2 := arr[1:3]
	fmt.Printf("s2=%v, len(s2)=%d, cap(s2)=%d\n", s2, len(s2), cap(s2))
	s3 := append(s2, 11)
	s4 := append(s3, 12)
	fmt.Printf("s3= %v, %d, %d\n", s3, len(s3), cap(s3))
	fmt.Println(s4)
	fmt.Println(arr)
}
```

切片扩容，每次添加一个元素，len加一，len若大于cap则cap翻倍扩容

```go
func printSlice(arr []int) {
	fmt.Printf("len=%d, cap=%d\n", len(arr), cap(arr))
}

func sliceOp() {
	var arr []int
	for i := 0; i < 100; i++ {
		printSlice(arr)
		arr = append(arr, i*2+1)
	}
	fmt.Println(arr)
}
```

创建slice，arr := make([]int, len, cap) 限制长度和容量

copy(source, target)：将target切片中元素拷贝到source中

移除元素：采用append追加第二段内容实现

```go
func remove(arr []int, index int) {
	target := arr[index]
	arr = append(arr[:index], arr[index+1:]...)
	fmt.Printf("remove element %d, after array is %v\n", target, arr)
	printSlice(arr)
}

func sliceOp2() {
	arr := make([]int, 10, 16)
	fmt.Println(arr)
	arr2 := []int{0, 1, 2, 3}
	copy(arr, arr2)
	fmt.Println(arr)
	remove(arr, 2)
}
```

### 4. map

- 创建，make(map[string]int)
- 获取元素：map[key]。当key不存在时，返回目标类型一个空值
- 获取元素：value,ok=map[key]，当key不存在时，ok为false
- 删除元素：delete(map, key)
- map遍历，range遍历。map是一个哈希表，遍历的结果是无序的

**map的key**

> java中hashmap自定义类型需要实现hashcode和equals方法

- map使用哈希表，必须可以比较是否相等
- 除了slice、map、function类型，其他类型都可以作为map的key（数组不属于slice，可以作为key）。struct若不包括其中三个类型，也可以作为map的key

### 5. 字符与字符串处理

- 使用range遍历pos, rune对
- 使用utf8.RuneCountInString获得字符数量，使用len(s)获取字节长度
- 使用[]byte获得字节数组，使用[]rune转换成字符数组
- ch, size := utf8.DecodeRune(bytes) 解析一个字符rune

```go
func charOp() {
	s := "go语言学习！"
	fmt.Println(len(s))
	for k, v := range s {
		fmt.Printf("(%d %X) ", k, v)
	}
	fmt.Println()
	for k, v := range []byte(s) {
		fmt.Printf("(%d %X) ", k, v)
	}
	fmt.Println()

	for k, v := range []rune(s) {
		fmt.Printf("(%d %c) ", k, v)
	}
	fmt.Println()

	fmt.Printf("(%s)字符长度=%d, 字节长度=%d\n", s, utf8.RuneCountInString(s), len(s))
	bytes := []byte(s)
	for len(bytes) > 0 {
		ch, size := utf8.DecodeRune(bytes)
		bytes = bytes[size:]
		fmt.Printf("%c ", ch)
	}
	fmt.Println()
}
```

**字符串常用方法**

strings集合

- strings.Fields(string) []string：按照空白字符串拆分，将目标字符串拆分成字符串数组。其中空白字符由unicode.IsSpace定义
- strings.Split(s, string) []string：按照给定字符串seq拆分目标字符串为字符串数组
- Strings.Join([]string, string) string：字符串凭借。将给定目标字符串数组拼接为一个字符串，用给定字符串作为拼接段
- strings.index(string, string) int：找到第一个子串的位置并返回。当目标子串不存在时返回-1
- string.trim(string, string) string：取出首尾固定字符串。常用来取出空白字符

## 面向对象

### 1. 结构体和方法

- 通过 type {name} struct 定义结构体
- go语言中不支持继承和多态
- 结构体通过.来获取内部对象
- 结构体提供了部分构造方法
  - TreeNode{}
  - TreeNode{filed:value...}
  - new(TreeNode)

**构造方法**

通过定义工厂方法实现。注意返回的是局部变量地址

**方法接收者**

结构体中不能定义方法，而是通过方法接收者来实现类似于结构体中的函数: func (node TreeNode) print()，默认值传递

方法接收者可以接收nil

**值接收者和指针接收者**

- 值接收者进行数据拷贝，指针接收者传递地址
- 当结构体过大时使用指针接收者
- 一致性：尽量使用指针接收者

```go
type TreeNode struct {
	left, right *TreeNode
	value       int
}

// 工厂方法
func createTreeNode(value int) *TreeNode {
	return &TreeNode{value: value}
}

func node1() {
	var root TreeNode
	root = TreeNode{value: 3}
	root.left = &TreeNode{}
	root.right = &TreeNode{nil, nil, 1}
	root.left.left = new(TreeNode)
	root.left.right = createTreeNode(2)

	nodes := []TreeNode{
		{value: 3},
		{nil, nil, 1},
		root,
	}
	fmt.Println(nodes)

	root2 := TreeNode{}
	root2.print()
	root2.setValue(2)
	root.print()

	var rootNil *TreeNode
	rootNil.setValue(2)
}

// 方法接收者，通过struct.print()调用，可以接收nil。值传递
func (node TreeNode) print() {
	fmt.Println(node.value)
}

// 指针传递
func (node *TreeNode) setValue(value int) {
	if node == nil {
		fmt.Println("nil TreeNode, ignore")
		return
	}
	node.value = value
}
// 中序遍历
func (node *TreeNode) traverseMiddle() {
	if node == nil {
		return
	}
	node.left.traverseMiddle()
	print(node.value)
	node.right.traverseMiddle()
}

func main() {
	node1()
}
```

### 2. 包和封装

- 使用驼峰命名法
- 首字母大写表示public，首字母小写表示private。适用于结构体、结构体方法、结构体变量
- 不允许引入重名的包名

### 3. 扩展已有类型

- 别名或组合

```go
// 扩展类
type myTreeNode struct {
	*tree.Node
}

// 扩展后续遍历方法
func (myNode *myTreeNode) postOrder() {
	if myNode == nil || myNode.node == nil {
		return
	}
	left := myTreeNode{myNode.node.Left}
	right := myTreeNode{myNode.node.Right}

	left.postOrder()
	right.postOrder()
	myNode.node.Print()
}
```

**内嵌**

- 省略.调用过程， 将子类所有的域和方法都平铺在组合类中，变量名为类名

```go
type myTreeNode struct {
	// 响应与定义了名为Node的Node节点，不可重名
	*test.Node
}

// 扩展后续遍历
func (myNode *myTreeNode) postOrder() {
	if myNode == nil || myNode.Node == nil {
		return
	}
	left := myTreeNode{myNode.Left}
	right := myTreeNode{myNode.Right}

	left.postOrder()
	right.postOrder()
	myNode.Print()
}
```

### 4. 使用内嵌扩展已有类型

扩展方法

- 定义别名：最简单
- 组合：最常用
- 内嵌：可节省很多代码

## Go语言的依赖管理

### 1. 依赖管理

依赖管理的三个阶段：GOPATH，GOVENDOR，go mod

### 2. GOPATH和GOVENDOR

包寻找路径: GOVENDOR > GOROOT > GOPATH

gopath包含整个包文件，但没有版本控制

govendor在每个项目下新增vendor目录，来实现版本控制，完成不同项目对统一包的不同版本引用。第三方管理工具（将对应版本号拷贝到vendor目录下）：glide, dep, do dep...

缺点：依赖管理不完善，GOPATH不具备版本号管理，GOVENDOR产生了依赖拷贝，项目臃肿

### 3. go mod

获取依赖（不指定版本号时默认拉取最新版本）：go get -u go.uber.org/zap[@v1.17]

下载指定依赖的两种方式

- 通过命令下载依赖，go get
- 利用idea中直接运行，自动下载对应的依赖

```
# 创建go.mod文件
go mod init {moduleName}
# 下载对应依赖，清理多余版本
Go mod dity
# 将目录下所有文件都build一遍
go build ./...
```

迁移旧项目（GOPATH或GOVENDOR）：go mod init > mod build ./...

### 4. 目录管理

go中一个目录下只能有一个main函数，项目才能整体运行（可以存在多个main函数单独运行）

go build ./... 对当前目录及子目录下的所有go文件进行编译检查，不生成结果

go install ./... 对当前目录及子目录下的所有文件执行并生成结果（目标文件可执行，结果为输出），文件输出至${GOPATH}/bin目录下

## 面向接口

### 1. 接口的概念

面向接口编程，通过接口封装实现功能

### 2. duck typing

### 3. 接口实现

- 实现方只需实现对应接口，无需关注接口名

**mock实现**

**mock包实现**

```go
package mock

type Retriever struct {
	Contents string
}

func (r Retriever) Get(url string) string {
	return r.Contents
}
```

**real实现**

```go
package real

import (
	"net/http"
	"net/http/httputil"
	"time"
)

type Retriever struct {
	UserAgent string
	TimeOut   time.Duration
}

func (r Retriever) Get(url string) string {
	resp, err := http.Get(url)
	if err != nil {
		panic(err)
	}
	result, err := httputil.DumpResponse(resp, true)
	resp.Body.Close()
	return string(result)
}
```

**接口层**

```go
package main

import (
   "fmt"
   "hellogo/retriever/mock"
   "hellogo/retriever/real"
)

type retrieverImpl interface {
   Get(string) string
}

func download(r retrieverImpl) string {
   return r.Get("http://www.baidu.com")
}

func main() {
   var r retrieverImpl
   r = mock.Retriever{Contents: "this is mock retriever"}
   r = real.Retriever{}
   fmt.Println(download(r))
}
```

### 4. 接口的类型和值

- Type Assertion/Type switch：r.(type) 获取到对应类型变量

```go
func main() {
	var r retrieverIntf
	r = mock.Retriever{Contents: "this is mock retriever"}
	inspect(r)
	r = &real.Retriever{
		UserAgent: "Mozilla/5.0",
		TimeOut:   time.Minute,
	}
	inspect(r)

	if v, ok := r.(*mock.Retriever); ok {
		fmt.Println("This is mock retriever. content=", v.Contents)
	} else {
		fmt.Println("This is not mock retriever")
	}
}

func inspect(r retrieverIntf) {
	fmt.Print("inspecting", r)
  // %T：打印类型，%v：打印值
	fmt.Printf("type:%T value:%v\n", r, r)
	fmt.Print(" > Type switch: ")
	switch v := r.(type) {
	case mock.Retriever:
		fmt.Println("mock retriever, ", v.Contents)
	case *real.Retriever:
		fmt.Println("real retriever, ", v.UserAgent)
	}
}
```

**泛型表示**

- 表示任何类型：interface{}，可用于泛型参数传递。t为interface{}类型，通过t.(int)将其强制转换为int类型

```go
type Queue []interface{}

func (q *Queue) push(value interface{}) {
	*q = append(*q, value.(int))
}

func (q *Queue) pop() interface{} {
	head := (*q)[0]
	*q = (*q)[1:]
	return head
}

func (q *Queue) isEmpty() bool {
	return len(*q) == 0
}
func main() {
	q := Queue{1}

	q.push(2)
	q.push(3)
	fmt.Println(q.pop())
	fmt.Println(q.pop())
	fmt.Println(q.isEmpty())
	fmt.Println(q.pop())
	fmt.Println(q.isEmpty())

	q.push("abc")
	fmt.Println(q.pop())
}
```

### 5. 常用系统接口

**string**

实现类似于java的toString方法

```go
func (r Retriever) String() string {
	return fmt.Sprintf("MockRetriever:{content=%s}", r.Contents)
}
```

**read/write**

```go
func printFile(filename string) {
	fmt.Println("start print file, filename=", filename)
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}
	printFileContents(file)
	fmt.Println("print end!!!")
}

func printFileContents(reader io.Reader) {
	scanner := bufio.NewScanner(reader)

	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
}

func main() {
	printFile("abc.txt")
	// 可包含特殊字符，换行
	s := `fabba
		"fdaf"
		81923fdsa

	fds	`
	printFileContents(strings.NewReader(s))
}
```

### 6. 函数式编程

- 函数内部不需要修饰如何访问自由变量
- 没有lambda表达式，只有匿名函数

## 错误处理和资源管理

### 1. defer

- 在程序退出前执行，多个defer命令先进后出
- 常用语资源关闭、锁释放
- 若defer中包含计算表达式，则会先计算，将结果放入栈中，再最后执行

```go
func WriteFile(filename string) {
	file, err := os.OpenFile(filename, os.O_EXCL|os.O_CREATE|os.O_WRONLY, 0666)

	if err != nil {
		if pathError, ok := err.(*os.PathError); !ok {
			panic(err)
		} else {
			fmt.Printf("%s %s %s\n", pathError.Op, pathError.Path, pathError.Err)
		}
	}
	defer file.Close()

	writer := bufio.NewWriter(file)
	defer writer.Flush()

	f := Fibonacci()
	for i := 0; i < 20; i++ {
		fmt.Fprintln(writer, f())
	}
}
```

### 2. panic&recover

**panic**

抛出异常

**recover**

捕获异常

```go
func tryRecover() {
	// 创建匿名函数并调用
	defer func() {
		r := recover()
		if err, ok := r.(error); ok {
			fmt.Println("error occurred:", err)
		} else {
			panic(fmt.Sprintf("I don't konw what to handle it: %v", r))
		}
	}()
	// 1. 创建异常
	//panic(errors.New("this is an error"))
	// 2. 其他类型
	panic(123)
}
```

### 3. 服务异常统一处理

Filelisting.go

```go
package filelisting

import (
	"io/ioutil"
	"net/http"
	"os"
	"strings"
)
const prefix = "/list/"

func HandleFileListing(writer http.ResponseWriter,
	request *http.Request) error {
	if strings.Index(request.URL.Path, prefix) != 0 {
		return userError("url must start with " + prefix)
	}
	path := request.URL.Path[len(prefix):]
	file, err := os.Open(path)
	if err != nil {
		return err
	}
	defer file.Close()

	all, err := ioutil.ReadAll(file)
	if err != nil {
		return err
	}
	writer.Write(all)
	return nil
}

type userError string

func (e userError) Error() string {
	return e.Message()
}

func (e userError) Message() string {
	return string(e)
}

```

Web.go

```go
package main

import (
	"fmt"
	"hellogo/errorhandling/fileserver/filelisting"
	"log"
	"net/http"
	"os"
)

type appHandler func(writer http.ResponseWriter,
	request *http.Request) error

func errWrapper(handler appHandler) func(http.ResponseWriter,
	*http.Request) {
	return func(writer http.ResponseWriter,
		request *http.Request) {
		defer func() {
			r := recover()
			if r != nil {
				fmt.Printf("panic:%v", r)
				http.Error(writer,
					http.StatusText(http.StatusInternalServerError),
					http.StatusInternalServerError)
			}
		}()
		err := handler(writer, request)
		if userErr, ok := err.(userError); ok {
			log.Println("userError occurred:", userErr)
			http.Error(writer, userErr.Message(), http.StatusBadRequest)
			return
		}

		code := http.StatusOK
		if err != nil {
			log.Printf("handle error。err: %s", err)
			switch {
			case os.IsNotExist(err):
				code = http.StatusNotFound
			case os.IsPermission(err):
				code = http.StatusForbidden
			default:
				code = http.StatusInternalServerError
			}
		}
		http.Error(writer, http.StatusText(code), code)
	}
}

type userError interface {
	error
	Message() string
}

/**
开启了一个文件服务器，指定tcp端口
内含异常处理，结合：error+panic+recover
*/
func main() {
	http.HandleFunc("/",
		errWrapper(filelisting.HandleFileListing))

	err := http.ListenAndServe(":8888", nil)

	if err != nil {
		panic(err)
	}
}
```

## 测试

### 1. 单元测试

- 测试文件必须以_test结尾，如norepeating_test.go
- 测试方法必须以Test开头，如TestNoRepeating(t *testing.T){}

样例

```go
// triangle.go
func CalTriangle(a, b int) int {
	return int(math.Sqrt(float64(a*a + b*b)))
}

// triangle_test.go
func TestTriangle(t *testing.T) {
	tests := []struct{ a, b, c int }{
		{3, 4, 5},
		{3, 4, 4},
		{3, 4, 3},
		{5, 12, 13},
		{8, 15, 17},
		{12, 35, 37},
		{30000, 40000, 50000},
	}

	for _, tt := range tests {
		if res := CalTriangle(tt.a, tt.b); res != tt.c {
			t.Errorf("CalTriangle(%d, %d)=%d, but expect %d", tt.a, tt.b, res, tt.c)
		}
	}
}
```

输出

```
=== RUN   TestTriangle
    triangle_test.go:20: CalTriangle(3, 4)=5, but expect 4
    triangle_test.go:20: CalTriangle(3, 4)=5, but expect 3
--- FAIL: TestTriangle (0.00s)
FAIL
```

### 2. 代码覆盖率

使用idea或命令行生成代码覆盖率

```sh
# 生成代码覆盖率文件
go test -coverprofile=c.out
# 以网页的形式查看覆盖率输出文件
go tool cover -html=c.out
```

### 3. 性能测试

- 方法名以BenchMark开头
- 命名行执行：go test -bench .

```go
func BenchmarkMaxLenNoRepeatString(b *testing.B) {
	s := "黑化肥挥发发灰会花飞灰化肥挥发发黑会飞花"
	expect := 8
  // b.N 表示系统决定运行次数
	for i := 0; i < b.N; i++ {
		if res := MaxLenNoRepeatString(s); res != expect {
			b.Errorf("MaxLenNoRepeatString(%s)=%d, but expect %d", s, res, expect)
		}
	}
}
```

输出

```go
goos: darwin
goarch: amd64
pkg: hellogo/test/norepeating
cpu: Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz
BenchmarkMaxLenNoRepeatString
BenchmarkMaxLenNoRepeatString-12    	 1214302	      1071 ns/op
PASS
```

### 4. 使用pprof进行优化

pprof可以通过性能测试生成对应文件，从而查看分析耗时步骤，需下载插件才能查看svg文件：www.graphviz.org

go test -bench . -cpuprofile cpu.out 生成性能分析文件

go tool pprof cpu.out  通过工具查看输出文件，进入命令行交互模式，输入"web"查看网页版文件

> 箭头越粗，方框越大表示耗时占比越高

![image-20210630004503285](../../image/image-20210630004503285.png)

### 5. 测试http服务器

分为两种，直接调方法接口，和模拟服务器调用http接口两种

### 6. 生成文档

```sh
# 查看接口queue的方法，无注释
go doc queue
# 查看方法IsEmpty
go doc IsEmpty
```

**godoc**

可以生成网页版的本地程序文档。文档中可以将注释生成对应文档

- 增加空 // 表示换行
- 注释行前面增加tab表示为代码框

> godoc 在1.13移除了，可通过go get golang.org/x/tools/cmd/godoc安装go

通过命令开启生成网页版文档：Godoc -http :6060

**Example test**

简单的测试用例文件，通过注释标记期望结果。第一行为”Output:“，下面为期望结果，均使用行注释

```go
func ExampleQueue_Pop() {
	q := Queue{1}
	q.Push(2)
	q.Push(3)
	fmt.Println(q.Pop())
	fmt.Println(q.Pop())
	fmt.Println(q.IsEmpty())

	fmt.Println(q.Pop())
	fmt.Println(q.IsEmpty())

	// Output:
	//1
	//2
	//false
	//3
	//true
}
```

##  Goroutine

### 1. Goroutine

go并发编程，通过go func(){}()并发开启多个协程操作

线程和协程的区别

- 协程是一种轻量级线程，完全由用户控制调度（非抢占式）

- 线程是抢占式的，协程时非抢占式的，只能自己释放资源
- 线程进程都是同步的，协程是异步的
- 一个线程包含多个协程

```go
func main() {
	const conc = 10
	var a [conc]int
	for i := 0; i < conc; i++ {
		go func(j int) {
			for true {
				a[j]++
			}
		}(i)
	}
	time.Sleep(time.Millisecond)
	fmt.Println(a)
}
```

> 协程是非抢占式的，对于go1.14版本之前，上述代码会因为协程一直占用cpu没有释放（线程资源大于cpu核数，mac i7 6核测试，因超线程的存在，一核两用，在1.14版本之前12线程开始程序就死循环），导致main函数无法获取cpu资源，也无法停止程序。
>
> go1.14版本后进行了优化， 可以异步抢占
>
> 也可以使用runtime.Gosched()主动释放cpu资源

go run -race test.go 可以查看数据访问冲突

### 2. go语言的调度器

goruntine提供了协程并发执行，协程是非抢占式的。但是go语言调度器控制了部分抢占式调度时机进行cpu切换，切换的点如下（包含，还有其他未列举的点）

- IO，select
- channel
- 等待锁
- 函数调用（有时）
- runtime.Gosched()

## channel

### 1. 基本用法

c -< 'a' 向channel c发数据

<-c 从channel c中接收数据

channel缓存区，超过指定缓存区才死机

Chan<- int 定义只发送数据类型的channel

<-chan int 定义只接受数据类型的channel

**channel close**

调用close(c) 关闭channel，但channel仍会接收数据0，可通过

1. n, ok := <-c，通过ok判断是否结束
2. for n := range c，range自动判断channel是否关闭

**不要通过共享内存来通信，要通过通信来共享内存**

```go
package main

import (
	"fmt"
	"time"
)

func worker(c chan int, id int) {
	//for {
	//	n, ok := <-c
	// 	通过ok判定channel是否关闭
	//	if !ok {
	//		break
	//	}
	//	fmt.Printf("%d receiver %d\n", id, n)
	//}
	// range自动判定channel是否关闭
	for n := range c {
		fmt.Printf("%d receiver %d\n", id, n)
	}
}

func createWorker(id int) chan int {
	// 定义channel，返回类型int
	c := make(chan int)
	// 调用监听方法
	go worker(c, id)
	return c
}

func chanDemo() {
	const n = 10
	var channels [n]chan int
	for i := 0; i < n; i++ {
		channels[i] = createWorker(i + 1)
	}

	for i := 0; i < n; i++ {
		// 向channel发送数据
		channels[i] <- 'a' + i
	}

	for i := 0; i < n; i++ {
		channels[i] <- 'A' + i
	}
	// 睡眠一定时间，保证go runtine方法打印出接收数据
	time.Sleep(time.Millisecond)
}

func bufferChan() {
  // channel缓存区大小3，发送消息超过3仍未被接收则发生deadlock
	c := make(chan int, 3)
	go worker(c, 10)
	c <- 1
	c <- 2
	c <- 3
	c <- 4

	time.Sleep(time.Millisecond)
}

func chanClose() {
	c := make(chan int)
	go worker(c, 11)
	c <- 1
	c <- 2
	c <- 3

	close(c)
	time.Sleep(time.Millisecond)
}

func main() {
	//bufferChan()
	chanClose()
}
```

### 2. 等待channel关闭

1. 定义一个channel来进行通信，进而共享内存，通过channel关闭结果
2. 使用系统提供的等待channel，sync.WaitGroup
   1. Add(): 设置通信阈值
   2. Done(): 发送完成消息
   3. Wait(): 等待所有消息

### 3. select的使用

- 接收各种channel的动作。对于为nil的channel，select仍然可以使用，不过不会进入对应case
- 定时器的使用：time.Tick(time.Second)，返回一个只收的channel，定时接收数据
- time.After(time.Second)，在指定时间后接收数据

### 4. 传统同步模式

go一般使用channel来解决并发通信，也可以使用传统加锁同步模式实现并发编程

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type atomicInt struct {
	value int
	lock  sync.Mutex
}

func (a *atomicInt) get() int {
	a.lock.Lock()
	defer a.lock.Unlock()
	return a.value
}

func (a *atomicInt) increment() {
	a.lock.Lock()
	defer a.lock.Unlock()
	a.value++
}

func main() {
	var a atomicInt
	a.increment()
	go func() {
		a.increment()
	}()
	time.Sleep(time.Millisecond)
	fmt.Println(a.get())
}

```

### 5. 并发模式

以下实现一个并发处理多个channel的程序，通过将多个channel的数据发送到统一channel，异步处理。

合并channel的方式如下

1. 每个channel开启一个go runtine，将数据传递给统一channel。开启了多个go runtine，适用于未知channel个数的聚合
2. 利用select，枚举各种channnel输出，仅开启一个go runtine，适用于已知个数且个数很少的channel合并

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func msgGen(name string) chan string {
	c := make(chan string)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
			c <- fmt.Sprintf("%s send %d", name, i)
			i++
		}
	}()
	return c
}

// 不确定channel的个数
func fallIn(chs ...chan string) chan string {
	c := make(chan string)
	for _, channel := range chs {
		go func(in chan string) {
			for {
				c <- <-in
			}
		}(channel)
	}
	return c
}

// 确定channel的个数，使用select
func fallInSelect(c1, c2 chan string) chan string {
	c := make(chan string)
	go func() {
		for {
			select {
			case m := <-c1:
				c <- m
			case m := <-c2:
				c <- m
			}
		}
	}()
	return c
}

// 并行发送消息，通过统一channel接收多个channel的数据，打印
func main() {
	c1 := msgGen("service1")
	c2 := msgGen("service2")
	c3 := msgGen("service3")
	c := fallIn(c1, c2, c3)
	//c := fallInSelect(c1, c2)
	for {
		fmt.Println(<-c)
	}
}
```

### 6. 非阻塞等待/超时等待/退出

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// chan struct{} 可以直接传递空数据
func msgGen(name string, done chan struct{}) chan string {
	c := make(chan string)
	go func() {
		i := 0
		for {
			select {
			case <-time.After(time.Duration(rand.Intn(1000)) * time.Millisecond):
				c <- fmt.Sprintf("%s send %d", name, i)
				i++
			// 等待结束信号
			case <-done:
				fmt.Println("cleaning up")
				time.Sleep(2 * time.Second)
				fmt.Println("cleaning done")
				done <- struct{}{}
				return
			}
		}
	}()
	return c
}

func nonBlockingWait(c chan string) (string, bool) {
	select {
	case n := <-c:
		return n, true
	default:
		return "", false
	}
}

func timeoutWait(c chan string, timeout time.Duration) (string, bool) {
	select {
	case n := <-c:
		return n, true
	case <-time.After(timeout):
		return "", false
	}
}

func main() {
	done := make(chan struct{})
	c := msgGen("service1", done)
	for i := 0; i < 5; i++ {
		if v, ok := timeoutWait(c, time.Duration(500)*time.Millisecond); ok {
			fmt.Println(v)
		} else {
			fmt.Println("timeout")
		}
	}
	// 通知channel数据发送完成
	done <- struct{}{}
	// 等待channel成功关闭
	<-done
}
```

## 标准库

### Fmt.Print

- fmt.Sprintf()：将对应数据转换成字符串返回

### http

- get
  - 添加User-Agent来控制网页类型（pc版，手机版）

Tips:

- import _ "net/http/pprof" 导入但不直接使用
- 访问/debug/pporf
- 使用go tool pprof分析性能
  - web程序引入pprof包，可以完成相关监控。import_ "net/http/pprof"，访问地址：debug/pprof/
  - sh命令：go tool pprof http://localhost:8888/debug/pprof/profile，统计30s的cpu利用率
  - sh命令：go tool pprof http://localhost:8888/debug/pprof/heap，统计30s的堆内存使用情况

### httputil

dumpResponse()

<<<<<<< HEAD
## 其他

- 获取函数全名
  - runtime.FuncForPC(reflect.ValueOf(handler).Pointer()).Name()
=======
### json

包：encoding/json

网络传输一般使用小写单词拼接

序列化建议使用结构体，方便取值

```go
type Order struct {
	ID int32 `json:"id"`
	// 忽略空值
	Name       string       `json:"name,omitempty"`
	TotalPrice float64      `json:"total_price"`
	OrderItems []*OrderItem `json:"order_items"`
}

type OrderItem struct {
	ID    int32   `json:"id"`
	Name  string  `json:"name"`
	Price float64 `json:"price"`
}
```
>>>>>>> 94bf179be16ed1132793f5ec3dfa7582fcd96411

