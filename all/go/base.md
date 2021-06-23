## 环境搭建

### 安装

https://studygolang.com/dl

配置环境变量

```go
// 查看go版本
go version
// 查看go环境变量
go env
// 更换go代理地址为国内。代理含义，首选goproxy，否则再直接拉取
go env -w GOPROXY=https://goproxy.cn,direct
// go依赖管理，运用新model机制
go env -w GO111MODULE=on
// 安装goimports
go get -v golang.org/x/tools/cmd/goimports
```

### IDE

idea+go插件

插件安装

- go
- file watcher

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

### 面向对象

#### 1. 结构体和方法

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

#### 2. 包和封装

- 使用驼峰命名法
- 首字母大写表示public，首字母小写表示private。适用于结构体、结构体方法、结构体变量

#### 3. 扩展已有类型

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

#### 4. 使用内嵌扩展已有类型

扩展方法

- 定义别名：最简单
- 组合：最常用
- 内嵌：可节省很多代码

### Go语言的依赖管理

#### 1. 依赖管理

依赖管理的三个阶段：GOPATH，GOVENDOR，go mod

#### 2. GOPATH和GOVENDOR

包寻找路径: GOVENDOR > GOROOT > GOPATH

gopath包含整个包文件，但没有版本控制

govendor在每个项目下新增vendor目录，来实现版本控制，完成不同项目对统一包的不同版本引用。第三方管理工具（将对应版本号拷贝到vendor目录下）：glide, dep, do dep...

缺点：依赖管理不完善，GOPATH不具备版本号管理，GOVENDOR产生了依赖拷贝，项目臃肿

#### 3. go mod

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

#### 4. 目录管理

go中一个目录下只能有一个main函数，项目才能整体运行（可以存在多个main函数单独运行）

go build ./... 对当前目录及子目录下的所有go文件进行编译检查，不生成结果

go install ./... 对当前目录及子目录下的所有文件执行并生成结果（目标文件可执行，结果为输出），文件输出至${GOPATH}/bin目录下

