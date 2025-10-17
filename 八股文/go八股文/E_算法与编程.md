[TOC]

## 一、 Go 与面向对象编程 (OOP)

Go 语言虽然没有像 Java/C++ 那样的 `class` 关键字，但它通过其他方式优雅地支持了面向对象编程的核心思想：封装、继承和多态。

### 1.1 封装 (Encapsulation)

Go 通过**包 (package)** 和**标识符的大小写**来实现封装。

-   **首字母大写**: 标识符（如变量、常量、函数、结构体、接口、方法）如果首字母大写，表示它是**公开的 (Public)**，可以被包外的代码访问。
-   **首字母小写**: 如果首字母小写，表示它是**私有的 (Private)**，只能在同一个包内部访问。

**示例 (`person/person.go`)**:
```go
package person

// Person 是一个公开的结构体
type Person struct {
    Name string // 公开字段
    age  int    // 私有字段
}

// NewPerson 是一个公开的构造函数
func NewPerson(name string, age int) *Person {
    if age < 0 {
        age = 0
    }
    return &Person{Name: name, age: age}
}

// GetAge 是一个公开的方法，用于访问私有字段 age
func (p *Person) GetAge() int {
    return p.age
}
```

### 1.2 继承 (Inheritance) -> 组合

Go 语言没有传统的继承，而是推崇**组合优于继承**的思想。它通过**结构体嵌入 (Struct Embedding)** 来实现代码复用。

当一个结构体嵌入到另一个结构体中时，外部结构体可以直接访问内部结构体的字段和方法，就好像是自己的一样。

```go
// 定义一个基础的 Animal 结构体
type Animal struct {
    Name string
}

func (a *Animal) Speak() {
    fmt.Printf("%s makes a sound.\n", a.Name)
}

// Dog 结构体嵌入了 Animal
type Dog struct {
    Animal // 嵌入，而不是 `animal Animal`
    Breed  string
}

// Dog 可以有自己的方法
func (d *Dog) Bark() {
    fmt.Println("Woof!")
}

// Dog 也可以“重写”嵌入类型的方法
func (d *Dog) Speak() {
    fmt.Printf("%s the %s barks.\n", d.Name, d.Breed)
}

func main() {
    d := Dog{
        Animal: Animal{Name: "Buddy"},
        Breed:  "Golden Retriever",
    }

    d.Speak() // 调用的是 Dog 的 Speak 方法
    d.Bark()  // 调用 Dog 自己的方法
    
    // 也可以显式调用被嵌入的 Animal 的方法
    d.Animal.Speak()
}
```

### 1.3 多态 (Polymorphism)

Go 通过**接口 (Interface)** 完美地实现了多态。接口定义了一组行为（方法集），任何实现了这些行为的类型都可以被看作是该接口类型。

(此部分在 `A_go基础语法与特性.md` 中已有详细介绍，这里作为回顾。)

## 二、 常见设计模式的 Go 实现

### 2.1 单例模式 (Singleton)

确保一个类只有一个实例，并提供一个全局访问点。在 Go 中，最佳实践是使用 `sync.Once` 来实现线程安全的单例。

```go
import "sync"

type singleton struct{}

var instance *singleton
var once sync.Once

// GetInstance 使用 sync.Once 来确保创建实例的代码只执行一次
func GetInstance() *singleton {
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}
```

### 2.2 工厂模式 (Factory)

工厂模式用于创建对象，将对象的创建逻辑与使用逻辑分离。

```go
// 定义一个接口
type PaymentMethod interface {
    Pay(amount float64)
}

// 实现1: Cash
type Cash struct{}
func (c *Cash) Pay(amount float64) { fmt.Println("Paid with cash") }

// 实现2: CreditCard
type CreditCard struct{}
func (cc *CreditCard) Pay(amount float64) { fmt.Println("Paid with credit card") }

// 工厂函数
func GetPaymentMethod(method string) (PaymentMethod, error) {
    switch method {
    case "cash":
        return &Cash{}, nil
    case "credit":
        return &CreditCard{}, nil
    default:
        return nil, fmt.Errorf("payment method %s not recognized", method)
    }
}

func main() {
    payment, _ := GetPaymentMethod("credit")
    payment.Pay(100.0)
}
```

### 2.3 选项模式 (Options Pattern)

当一个结构体有许多配置项，特别是可选配置项时，选项模式是一种非常优雅的解决方案。

```go
type Server struct {
    Host string
    Port int
    Timeout time.Duration
}

// Option 是一个函数类型，用于修改 Server 对象
type Option func(*Server)

// WithHost 是一个返回 Option 的函数
func WithHost(host string) Option {
    return func(s *Server) {
        s.Host = host
    }
}

func WithPort(port int) Option {
    return func(s *Server) {
        s.Port = port
    }
}

// NewServer 是构造函数，接收任意数量的 Option
func NewServer(opts ...Option) *Server {
    // 设置默认值
    s := &Server{
        Host: "localhost",
        Port: 8080,
        Timeout: time.Second * 30,
    }

    // 应用所有传入的选项
    for _, opt := range opts {
        opt(s)
    }
    return s
}

func main() {
    // 只需传入需要自定义的选项，非常灵活
    server := NewServer(WithPort(9000))
}
```

## 三、 常见算法的 Go 实现思路

### 3.1 排序 (Sorting)

Go 的 `sort` 包提供了强大的排序功能。

-   **对基本类型切片排序**:
    ```go
    s := []int{5, 2, 6, 3, 1, 4}
    sort.Ints(s) // s 现在是 [1, 2, 3, 4, 5, 6]

    names := []string{"Bob", "Alice", "Charlie"}
    sort.Strings(names) // names 现在是 ["Alice", "Bob", "Charlie"]
    ```

-   **对自定义结构体切片排序**:
    需要让你的类型实现 `sort.Interface` 接口，它包含三个方法：`Len()`, `Less(i, j int) bool`, `Swap(i, j int)`。

    ```go
    type Person struct {
        Name string
        Age  int
    }

    // 定义一个 Person 切片的别名
    type ByAge []Person

    func (a ByAge) Len() int           { return len(a) }
    func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }
    func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }

    func main() {
        people := []Person{
            {"Bob", 31},
            {"John", 42},
            {"Alice", 25},
        }
        sort.Sort(ByAge(people))
        // people 现在按年龄升序排序
    }
    ```

### 3.2 二分查找 (Binary Search)

二分查找是一种在**有序数组**中查找某一特定元素的搜索算法。

```go
func BinarySearch(arr []int, target int) int {
    low, high := 0, len(arr)-1

    for low <= high {
        mid := low + (high-low)/2 // 防止溢出
        if arr[mid] == target {
            return mid
        } else if arr[mid] < target {
            low = mid + 1
        } else {
            high = mid - 1
        }
    }
    return -1 // 未找到
}
```

### 3.3 双指针 (Two Pointers)

双指针是一种常见的算法技巧，通过使用两个指针在数据结构中移动来解决问题，通常能将 O(n^2) 的复杂度降低到 O(n)。

**示例：在有序数组中寻找和为定值的两个数**

```go
func FindSum(arr []int, target int) (int, int) {
    left, right := 0, len(arr)-1

    for left < right {
        sum := arr[left] + arr[right]
        if sum == target {
            return arr[left], arr[right]
        } else if sum < target {
            left++
        } else {
            right--
        }
    }
    return -1, -1 // 未找到
}
```
