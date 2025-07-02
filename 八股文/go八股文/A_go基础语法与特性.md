[TOC]

## 一、 变量与常量

### 1.1 变量声明

Go 语言是静态类型语言，在声明变量时需要指定其类型，或者由编译器通过初始值推断出类型。

#### a. `var` 关键字

`var` 是最基本的变量声明方式，可用于函数内外。

- **只声明，不初始化**：变量会被赋予其类型的“零值”。

  ```go
  // 声明一个整数变量，默认为 0
  var age int

  // 声明一个字符串变量，默认为 ""
  var name string

  // 声明一个布尔变量，默认为 false
  var isAdmin bool
  ```

- **声明并初始化**：编译器可以自动推断类型。

  ```go
  // 显式指定类型
  var city string = "Beijing"

  // 自动推断类型
  var country = "China" // 编译器推断出 country 是 string 类型
  ```

- **批量声明**：

  ```go
  var (
      user string = "admin"
      id   int    = 1
      isActive bool   = true
  )
  ```

#### b. `:=` 简短声明

`:=` 是 `var` 的一种简写形式，**只能在函数内部使用**。它会自动推断类型，并且必须在声明时初始化。

```go
func main() {
    // 声明并初始化，自动推断类型
    name := "GoLang"
    age := 10

    // 等价于
    // var name = "GoLang"
    // var age = 10

    // 注意：`:=` 左侧至少要有一个是新声明的变量
    name, email := "Go", "go@example.com" // 正确，email是新变量
    // name, age := "Go", 11 // 编译错误，name 和 age 都已经声明过
}
```

#### c. `new` 函数

`new(T)` 用于分配内存，它返回一个指向类型 `T` 的零值的**指针**。

```go
// 创建一个指向 int 零值 (0) 的指针
p := new(int)
fmt.Println(*p) // 输出: 0

*p = 100
fmt.Println(*p) // 输出: 100

// 常用于为结构体分配内存
type User struct{
    Name string
}
user := new(User)
user.Name = "Tom" // Go 语言会自动解引用
```

### 1.2 常量声明

常量在编译时就已经确定其值，程序运行时不能修改。

#### a. `const` 关键字

```go
const pi = 3.14159
const version string = "1.0.0"

// 批量声明
const (
    StatusOK   = 200
    StatusNotFound = 404
)
```

#### b. `iota` 常量生成器

`iota` 是一个特殊的常量，可以被编译器修改。在每个 `const` 声明块中，`iota` 的初始值为 0，每新增一行常量声明，`iota` 的值就加 1。

```go
const (
    A = iota // A = 0
    B = iota // B = 1
    C = iota // C = 2
)

const (
    Read   = 1 << iota // 1 << 0 = 1
    Write  = 1 << iota // 1 << 1 = 2
    Execute = 1 << iota // 1 << 2 = 4
)
```

## 二、 基础数据类型

- **布尔型**: `bool` (true, false)
- **数字类型**:
  - **整型**: `int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8` (byte), `uint16`, `uint32`, `uint64`
  - **浮点型**: `float32`, `float64`
  - **复数**: `complex64`, `complex128`
- **字符串**: `string`
- **其他**: `rune` (等同于 `int32`, 表示一个 Unicode 码点), `uintptr`

## 三、 复合数据类型

### 3.1 数组 (Array)

数组是**固定长度**的、**相同类型**元素的集合。由于长度固定，在 Go 中不常用，通常被切片代替。

```go
// 声明一个长度为 3 的整数数组，所有元素初始化为 0
var arr [3]int

// 声明并初始化
arr2 := [3]int{1, 2, 3}

// 编译器自动计算长度
arr3 := [...]int{4, 5, 6}

fmt.Println(arr2[0]) // 访问元素
arr2[0] = 100        // 修改元素
```

### 3.2 切片 (Slice)

切片是对底层数组一个连续片段的引用，它提供了更强大、灵活的功能。切片是**引用类型**。

- **`len()`**: 切片的长度，即包含的元素个数。
- **`cap()`**: 切片的容量，即从切片的起始元素到底层数组末尾的元素个数。

#### a. 创建切片

```go
// 1. 从数组创建
arr := [...]int{1, 2, 3, 4, 5}
slice := arr[1:4] // 包含索引 1, 2, 3 的元素，即 [2, 3, 4]

// 2. 使用 make 创建
// 创建一个长度为 5，容量为 10 的整数切片
slice2 := make([]int, 5, 10)

// 3. 字面量创建
slice3 := []int{10, 20, 30}
```

#### b. `append` 和扩容

`append` 用于向切片追加元素。如果追加后超过了切片的容量 (`cap`)，Go 会自动分配一个新的、更大的底层数组，并将所有元素复制过去。这就是**切片扩容**。

```go
slice := []int{1, 2, 3}
slice = append(slice, 4)

// 扩容策略（大致规则）:
// - 如果新长度小于 1024，容量会翻倍。
// - 如果新长度大于等于 1024，容量会增加 1.25 倍。
```

### 3.3 `map`

`map` 是键值对的无序集合，也称为哈希表或字典。它是**引用类型**。

#### a. 创建和初始化

`map` 必须使用 `make` 或字面量进行初始化后才能使用。

```go
// 使用 make 创建
m := make(map[string]int)

// 使用字面量创建
m2 := map[string]string{
    "name": "Tom",
    "city": "New York",
}

// 错误：nil map，无法赋值
var m3 map[string]int
// m3["key"] = 1 // 会导致 panic
```

#### b. 常用操作

```go
m := make(map[string]int)

// 插入或更新
m["age"] = 25

// 读取
age := m["age"]

// 删除
delete(m, "age")

// 判断 key 是否存在
value, ok := m["city"]
if ok {
    fmt.Println("City is:", value)
} else {
    fmt.Println("City not found.")
}

// 遍历
for key, value := range m2 {
    fmt.Printf("Key: %s, Value: %s\n", key, value)
}
```

### 3.4 结构体 (Struct)

结构体是自定义的、将多个不同类型的字段聚合在一起形成的数据类型。

```go
type Person struct {
    Name string
    Age  int
}

func main() {
    // 初始化
    p1 := Person{Name: "Alice", Age: 30}
    p2 := Person{"Bob", 25} // 必须按顺序提供所有字段的值

    // 访问字段
    fmt.Println(p1.Name)
}
```

## 四、 控制流

### 4.1 `if-else`

Go 的 `if` 语句不需要括号，但 `else` 必须和 `if` 的右花括号在同一行。

```go
if score := 95; score >= 90 {
    fmt.Println("Excellent")
} else if score >= 60 {
    fmt.Println("Pass")
} else {
    fmt.Println("Fail")
}
// `score` 变量的作用域只在 if-else 块内
```

### 4.2 `for`

Go 只有 `for` 循环，但它有多种形式。

```go
// 1. 基本形式 (同 C 的 for)
for i := 0; i < 5; i++ {
    fmt.Println(i)
}

// 2. 条件形式 (同 C 的 while)
sum := 1
for sum < 100 {
    sum += sum
}

// 3. 无限循环
for {
    // ...
    break // 需要 break 或 return 来退出
}

// 4. for-range 遍历
items := []string{"a", "b", "c"}
for index, value := range items {
    fmt.Printf("Index: %d, Value: %s\n", index, value)
}
```

### 4.3 `switch`

Go 的 `switch` 功能更强大。它默认在每个 `case` 后自动 `break`，除非使用 `fallthrough`。

```go
// 基本形式
day := "Friday"
switch day {
case "Monday", "Tuesday":
    fmt.Println("Work day")
case "Friday":
    fmt.Println("TGIF!")
default:
    fmt.Println("Weekend")
}

// 无表达式的 switch (替代 if-else if)
score := 80
switch {
case score >= 90:
    fmt.Println("A")
case score >= 80:
    fmt.Println("B")
default:
    fmt.Println("C")
}
```

## 五、 函数、方法、接口

### 5.1 函数 (Function)

- **多返回值**: 函数可以返回多个值，常用于同时返回结果和错误。
- **命名返回值**: 可以为返回值命名，它们在函数开始时被声明并初始化为零值。

```go
// 接收两个 int，返回它们的和与差
func SumAndDiff(a, b int) (int, int) {
    return a + b, a - b
}

// 命名返回值
func GetNameAndAge() (name string, age int) {
    name = "Tom"
    age = 10
    return // 隐式返回 name 和 age
}
```

### 5.2 方法 (Method)

方法是附加到特定类型（接收者）上的函数。接收者可以是值类型或指针类型。

```go
type Circle struct {
    Radius float64
}

// Area 是 Circle 类型的一个方法
// (c Circle) 是接收者
func (c Circle) Area() float64 {
    return 3.14 * c.Radius * c.Radius
}

// 使用指针接收者来修改实例
func (c *Circle) Scale(factor float64) {
    c.Radius *= factor
}
```

### 5.3 接口 (Interface)

接口是一组方法签名的集合。当一个类型实现了接口中的所有方法时，我们就说该类型实现了这个接口。

```go
// 定义一个接口
type Shape interface {
    Area() float64
}

type Rectangle struct {
    Width, Height float64
}

// Rectangle 实现了 Shape 接口
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Circle (见上文) 也实现了 Shape 接口

// 接口可以作为函数参数，实现多态
func PrintArea(s Shape) {
    fmt.Printf("Area is: %f\n", s.Area())
}

func main() {
    c := Circle{Radius: 5}
    r := Rectangle{Width: 3, Height: 4}

    PrintArea(c)
    PrintArea(r)
}
```

## 六、 指针 (Pointer)

指针存储了一个变量的内存地址。

- `&` 操作符：获取一个变量的地址。
- `*` 操作符：获取指针指向地址处的值（解引用）。

```go
func main() {
    x := 100
    p := &x // p 是一个指向 x 的 int 指针

    fmt.Println("Address of x:", p)
    fmt.Println("Value of x:", *p)

    *p = 200 // 通过指针修改 x 的值
    fmt.Println("New value of x:", x)
}
```

## 七、 错误处理

Go 语言通过内置的 `error` 接口类型来处理错误。一个函数如果可能出错，通常会将 `error` 作为其最后一个返回值。

```go
import (
    "fmt"
    "strconv"
)

func main() {
    s := "123a"
    // Atoi (string to integer) 返回一个 int 和一个 error
    num, err := strconv.Atoi(s)
    if err != nil {
        // 如果 err 不是 nil，说明发生了错误
        fmt.Println("Error:", err)
        return
    }

    fmt.Println("Converted number:", num)
}
```
