# Golang 学习路径

## 1. 基础语法 (基于通用核心)
- **变量声明**:
  - `var name string = "Go"`
  - **Go 特色: 类型推导 `name := "Go"`**
- **关键字**: 简洁，只有 25 个
- **运算符**: 基本通用，**但没有 `++i` (只有 `i++` 且为语句)**
- **注释**: `//` 和 `/* */` (通用)

## 2. 数据类型 (基于通用核心)
- **基础类型**: `int`, `float64`, `bool`, `string` (基本通用)
- **Go 特色: `rune`** (等同于 `int32`, 代表一个 UTF-8 字符)
- **复合类型**:
  - `struct` (通用概念)
  - `array` (固定长度)
  - **Go 特色: 切片 (Slice)** (动态数组, 核心数据结构)
  - **Go 特色: 字典 (Map)** (内置的 `map[keyType]valueType`)
- **特殊类型**:
  - `nil` (通用概念)
  - **指针 (Pointer)**:
    - (有指针, 像 C)
    - **Go 特色: 但不支持指针运算** (更安全)

## 3. 控制流程 (基于通用核心)
- **条件**:
  - `if / else` (通用)
  - **Go 特色: `if` 可带初始化语句 `if err := foo(); err != nil { ... }`**
  - `switch` (通用, 但 Go 的 `case` 默认带 `break`)
- **循环**:
  - **Go 特色: 只有 `for` 关键字**
  - `for i := 0; i < 10; i++` (同 C 的 for)
  - `for condition { ... }` (同 while)
  - `for { ... }` (无限循环)
  - `for k, v := range items { ... }` (迭代)
- **跳转**:
  - `break`, `continue`, `return` (通用)
  - **Go 特色: `goto`** (保留, 但不推荐)
  - **Go 特色: `defer`** (延迟执行, 用于资源释放)

## 4. 函数 (基于通用核心)
- (基本通用)
- **Go 特色: 支持多返回值** (常用于返回 `(result, error)`)
- **Go 特色: 函数是"一等公民"** (可作参数、返回值)

## 5. 错误处理 (基于通用核心)
- **Go 特色: 非异常 (Exception) 机制**
- 而是使用通用的**多返回值 `error` 类型**
- **Go 特色: `panic` / `recover`** (用于处理真正无法恢复的异常)

## 6. 内存管理 (基于通用核心)
- **Go 特色: 自动垃圾回收 (GC)** (像 Java, Python)
- (有栈和堆，由编译器自动分配)

## 7. [ Go 的核心特色分支 ]

### (1) 并发编程 (Concurrency)
- **Goroutine**:
  - (对应通用的 "线程" 概念)
  - **Go 特色: 极其轻量级的"协程"**, 由 Go 运行时管理
  - 语法: `go someFunction()`
- **Channel (通道)**:
  - (对应通用的 "锁" 概念, 但范式不同)
  - **Go 特色: 用于 Goroutine 间通信的数据结构**
  - 核心理念: "不要通过共享内存来通信，而要通过通信来共享内存"
- **Select**:
  - **Go 特色: 多路复用**, 可同时监听多个 Channel

### (2) 面向对象 (OOP) 范式
- **Go 特色: 没有 `class` (类)**
- **Go 特色: 没有继承 (Inheritance)**
- **封装 (Encapsulation)**:
  - 通过 **`struct` (结构体)** 组织数据
  - 通过 **方法 (Method)** 绑定行为 `func (s *MyStruct) myMethod() { ... }`
  - 通过 **包 (Package)** 控制可见性 (首字母大写 = `public`, 小写 = `private`)
- **多态 (Polymorphism)**:
  - 通过 **接口 (Interface)** 实现
  - **Go 特色: 隐式实现 (Duck Typing)**
  - (一个类型只要实现了接口的所有方法，就被视为实现了该接口，无需显式声明 `implements`)

### (3) 包管理与工具链
- **Go Modules**:
  - (对应通用的 "包管理" 概念)
  - `go.mod` / `go.sum` 文件
- **强大的标准库**:
  - (如 `net/http`, `json`)
- **工具链**:
  - `go build` (编译成**静态链接的单一可执行文件**)
  - `go test` (内置测试框架)
  - `gofmt` (统一的代码格式化)