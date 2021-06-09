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

