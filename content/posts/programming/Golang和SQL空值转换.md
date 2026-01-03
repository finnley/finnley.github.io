+++
title = 'Protobuf安装'
date = 2025-11-06T11:34:31+08:00
draft = true
categories = [ "Programming" ]
tags = [ "protobuf", "programming" ]
+++

```
dao.User{
		Id:           u.Id,
		Email:        sql.NullString{String: u.Email, Valid: u.Email != ""},
		Password:     sql.NullString{String: u.Password, Valid: u.Password != ""},
		PhoneNumber:  sql.NullString{String: u.PhoneNumber, Valid: u.PhoneNumber != ""},
		WechatOpenID: sql.NullString{String: u.WechatOpenID, Valid: u.WechatOpenID != ""},
		// Ctime 在 DAO 层面可以通过 gorm 的 hook 自动填充
		CreatedAt: u.Ctime.UnixMilli(),
	}
```

---
这是一个非常好的问题！`sql.NullString` 是 Go 语言 `database/sql` 标准库中一个“开箱即用”的工具，专门用来解决您之前遇到的 `""` (空字符串) 和 `NULL` (数据库空值) 之间的问题。

您可以把它理解为一个**“可空字符串”的容器**。

一个普通的 Go `string` 变量**不能**表示 `NULL`，它只有两种状态：
1.  有内容 (例如 `"1@qq.com"`)
2.  空字符串 (`""`)

而数据库的 `VARCHAR` 字段有**三种**状态：
1.  有内容 (例如 `'1@qq.com'`)
2.  空字符串 (`''`)
3.  `NULL` (表示“没有值”或“未知”)

`sql.NullString` 就是用来**连接这两种不同概念**的桥梁。

---

### 两个属性的含义

`sql.NullString` 结构体非常简单，只有两个属性（字段）：

#### 1. `String` (类型: `string`)

* **作用：** 存储实际的字符串值。
* **在您的代码中 (`String: u.Email`)：** 这行代码的意思是：“把 `u.Email` 变量（比如 `"1@qq.com"` 或者 `""`）的值，放进 `sql.NullString` 容器的 `String` 属性里。”

#### 2. `Valid` (类型: `bool`)

* **作用：** 这是一个“开关”或“标志位”。它告诉 GORM (或 `database/sql` 驱动) 这个字符串值是否**有效**。
* **`true` 的含义：** “这个值是有效的，请使用 `String` 属性里的值 (例如 `"1@qq.com"`) 插入到数据库。”
* **`false` 的含义：** “这个值是**无效**的，请**忽略** `String` 属性里的内容，转而向数据库插入 `NULL`。”

---

### 深入解析您这行代码

现在我们把它们组合起来，分析您这行关键代码：

`sql.NullString{String: u.Email, Valid: u.Email != ""}`

这行代码在**创建（INSERT）**或**更新（UPDATE）**数据时，动态地构建了一个 `sql.NullString` 容器。

我们来模拟两个场景：

#### 场景一：用户使用邮箱注册

1.  您的 `domain.User` 对象的 `u.Email` 值为 `"1@qq.com"`。
2.  代码开始构建 `sql.NullString`：
    * `String: u.Email` 变成了 `String: "1@qq.com"`。
    * `Valid: u.Email != ""` (即 `"1@qq.com" != ""`) 这个判断的结果是 `true`。
3.  **最终结果：**
    * GORM 拿到了 `sql.NullString{String: "1@qq.com", Valid: true}`。
    * GORM 查看 `Valid`，发现是 `true`，于是它取出 `String` 属性的值 `"1@qq.com"`，并执行 SQL：`INSERT INTO users (email, ...) VALUES ('1@qq.com', ...)`。
    * **这符合预期。**

#### 场景二：用户使用手机号注册（Email 为空）

1.  您的 `domain.User` 对象的 `u.Email` 值为 `""` (空字符串)。
2.  代码开始构建 `sql.NullString`：
    * `String: u.Email` 变成了 `String: ""`。
    * `Valid: u.Email != ""` (即 `"" != ""`) 这个判断的结果是 `false`。
3.  **最终结果：**
    * GORM 拿到了 `sql.NullString{String: "", Valid: false}`。
    * GORM 查看 `Valid`，发现是 `false`，于是它**忽略** `String` 属性的值，并执行 SQL：`INSERT INTO users (email, ...) VALUES (NULL, ...)`。
    * **这正是您想要的！**

---

### 总结：它是一个“翻译器”

您可以把 `Valid: u.Email != ""` 这段逻辑看作一个**翻译规则**：

| 如果您的Go代码 (`u.Email`) 是... | `Valid` 属性被设为... | GORM/数据库驱动会插入... | 最终结果 |
| :--- | :--- | :--- | :--- |
| `"1@qq.com"` | `true` | `String` 属性的值 (`'1@qq.com'`) | 成功插入邮箱 |
| `""` (空字符串) | `false` | `NULL` | 成功插入`NULL`，**避免了唯一键冲突** |

通过这个机制，您就可以让多个没有邮箱的用户（`email` 字段都为 `NULL`）共存在您的 `users` 表中，完美解决了 `Duplicate entry ''` 的问题。

---

在 Go 使用 **GORM** 处理数据库中的 **NULL 值** 时，**推荐使用 `sql.Null*` 类型（如 `sql.NullString`、`sql.NullInt64` 等）**，而不是普通指针（如 `*string`）。

虽然两者都能实现“空值”效果，但 **`sql.Null*` 是更正确、更安全、更符合 SQL 语义的选择**。

---

### 为什么推荐 `sql.NullString` 而不是 `*string`？

| 对比项 | `*string` | `sql.NullString` |
|--------|----------|------------------|
| **是否区分 `NULL` 和空字符串 `""`** | ❌ 无法区分 | ✅ 能区分 |
| **扫描/赋值行为** | 可能导致 `""` 被当成 `NULL` | 正确处理 `NULL` 和 `""` |
| **JSON 序列化** | `null` 或 `""` 取决于实现 | 通常为 `null`（更符合预期） |
| **GORM 官方推荐** | 不推荐 | 强烈推荐 |
| **数据库语义一致性** | 差 | 好 |

---

### 正确示例：使用 `sql.NullString`

```go
type User struct {
    ID   uint
    Name sql.NullString `gorm:"type:varchar(100)"`
    Email sql.NullString `gorm:"type:varchar(100);unique"`
}

// 插入 NULL
user := User{
    Name:  sql.NullString{String: "", Valid: false}, // 数据库中为 NULL
    Email: sql.NullString{String: "a@example.com", Valid: true},
}
db.Create(&user)

// 查询时自动处理 NULL
var user User
db.First(&user, 1)
if user.Name.Valid {
    fmt.Println(user.Name.String)
} else {
    fmt.Println("Name is NULL")
}
```

---

### 错误示范：使用 `*string`

```go
type User struct {
    ID   uint
    Name *string `gorm:"type:varchar(100)"`
}

// 插入 NULL
name := ""
db.Create(&User{Name: &name}) // 插入的是空字符串 ""，不是 NULL！

var empty *string = nil
db.Create(&User{Name: empty}) // 插入 NULL（但无法区分 "" 和 NULL）
```

> 问题：`*string = &emptyStr` 和 `*string = nil` 在数据库中都可能变成 `NULL` 或 `""`，取决于驱动和 GORM 版本，**行为不一致**。

---

### GORM 官方建议（摘自文档）

> Use `sql.Null*` types to handle `NULL` values properly.  
> Pointers may cause issues with zero values and NULL distinction.

参考：https://gorm.io/docs/data_types.html

---

### 其他常用 `sql.Null*` 类型

```go
import "database/sql"

sql.NullString
sql.NullInt64
sql.NullInt32
sql.NullBool
sql.NullFloat64
sql.NullTime
```

---

### 总结：**用 `sql.NullString`，不要用 `*string`**

| 场景 | 推荐 |
|------|------|
| 需要精确表示 `NULL` | `sql.NullString` |
| 字段允许为空（NULL） | `sql.Null*` |
| 字段必填 | 普通类型（如 `string`） |
| 快速原型（不关心 NULL 语义） | `*string`（不推荐生产） |

---

**最佳实践**：

```go
type User struct {
    ID    uint           `gorm:"primaryKey"`
    Name  sql.NullString `gorm:"size:255"`
    Email sql.NullString `gorm:"size:255;uniqueIndex"`
    Age   sql.NullInt64
    Active sql.NullBool
}
```

这样可以 **完美区分 `NULL`、空值、零值**，是生产环境的首选。