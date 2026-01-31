---
title: 第2周讲义：Kotlin 编程基础
show: true
date: 2025-09-25
permalink: /lectures/lecture02
---


**课程**: Android 移动应用开发入门  
**周次**: 第2周  
**主题**: Kotlin 编程基础  
**学时**: 3课时（理论2课时 + 实验1课时）

---

## 📋 本节课学习目标

完成本节课后，学生应该能够：

1. ✅ 理解 Kotlin 语言的特点和优势
2. ✅ 掌握 Kotlin 的基本语法（变量、数据类型、函数）
3. ✅ 正确使用 val 和 var 声明变量
4. ✅ 熟练使用控制流结构（if、when、循环）
5. ✅ 理解并应用 Kotlin 的空安全特性
6. ✅ 了解 Kotlin 与 Java 的主要区别
7. ✅ 能够编写简单的 Kotlin 程序

---

## 🎯 第一部分：为什么选择 Kotlin？

### 1.1 Kotlin 简介

**Kotlin** 是由 JetBrains 公司开发的现代编程语言，于 2011 年首次公布，2016 年正式发布 1.0 版本。

**重要里程碑**:

- **2017年**: Google 宣布 Kotlin 成为 Android 官方支持语言
- **2019年**: Google 宣布 Kotlin 成为 Android 开发首选语言（Kotlin-first）
- **现在**: 超过 60% 的专业 Android 开发者使用 Kotlin

### 1.2 Kotlin 的优势

#### 现代化

✨ **简洁的语法**

```kotlin
// Kotlin
val name = "Android"
println("Hello, $name!")

// Java 对比
final String name = "Android";
System.out.println("Hello, " + name + "!");
```

✨ **更少的样板代码**

一个简单的数据类：

```kotlin
// Kotlin - 一行搞定！
data class User(val name: String, val age: Int)

// Java - 需要几十行
public class User {
    private final String name;
    private final int age;
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    @Override
    public boolean equals(Object o) { ... }
    
    @Override
    public int hashCode() { ... }
    
    @Override
    public String toString() { ... }
}
```

✨ **类型推断**

```kotlin
val number = 42                    // 自动推断为 Int
val price = 19.99                  // 自动推断为 Double
val message = "Hello"              // 自动推断为 String
```

✨ **函数式编程支持**

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
val doubled = numbers.map { it * 2 }
val evens = numbers.filter { it % 2 == 0 }
```

#### 安全可靠

🛡️ **空安全（Null Safety）**

Kotlin 的类型系统从根本上消除了 NullPointerException：

```kotlin
var name: String = "Kotlin"
name = null                    // ❌ 编译错误！

var nullableName: String? = "Kotlin"
nullableName = null            // ✅ 可以
```

🛡️ **类型安全**

```kotlin
var age = 25
age = "twenty-five"           // ❌ 编译错误！类型不匹配
```

🛡️ **100% Java 互操作**

- 可以在 Kotlin 中调用 Java 代码
- 可以在 Java 中调用 Kotlin 代码
- 可以在同一项目中混用

### 1.3 Kotlin 在 Android 中的应用

**Google 官方统计**（2024）:

- 超过 95% 的 Android 应用使用 Kotlin
- Google Play 商店排名前1000的应用中，超过 70% 使用 Kotlin
- Jetpack 库完全支持 Kotlin 优先

**主要框架**:

- Jetpack Compose (UI 框架) - 纯 Kotlin
- Kotlin Coroutines (协程) - 异步编程
- Kotlin Multiplatform - 跨平台开发

---

## 💡 第二部分：变量与数据类型

### 2.1 变量声明：val vs var

Kotlin 使用两个关键字声明变量：

#### val - 不可变变量（Immutable）

```kotlin
val name = "Kotlin"
name = "Java"              // ❌ 编译错误：Val cannot be reassigned
```

**特点**:

- 只能赋值一次
- 类似 Java 的 `final` 变量
- **推荐使用** - 函数式编程的核心原则

**为什么推荐使用 val？**

1. **线程安全**: 不可变对象天然线程安全
2. **易于理解**: 值不会改变，代码更容易推理
3. **减少bug**: 防止意外修改
4. **函数式编程**: 符合函数式编程范式

#### var - 可变变量（Mutable）

```kotlin
var age = 25
age = 26                   // ✅ 可以重新赋值
age = "twenty-six"         // ❌ 编译错误：类型不匹配
```

**特点**:

- 可以重新赋值
- 但类型不能改变
- 仅在必要时使用

#### 选择 val 还是 var？

**原则**: 默认使用 `val`，只有在需要重新赋值时才使用 `var`

```kotlin
// ✅ 好的实践
val pi = 3.14159
val userName = getUserName()

// ❌ 不好的实践（不必要的 var）
var pi = 3.14159           // pi 不会变，应该用 val

// ✅ 合理使用 var
var count = 0
for (i in 1..10) {
    count += i
}
```

### 2.2 类型推断

Kotlin 编译器可以自动推断变量类型：

```kotlin
// 类型推断
val message = "Hello"                  // String
val count = 42                         // Int
val price = 19.99                      // Double
val isValid = true                     // Boolean
val items = listOf("A", "B", "C")      // List<String>

// 显式类型声明（可选）
val message: String = "Hello"
val count: Int = 42
val price: Double = 19.99
```

**何时需要显式声明类型？**

1. **初始值不明确时**:

```kotlin
val result: Int = if (x > 0) x else -x
```

2. **希望使用父类型时**:

```kotlin
val numbers: List<Int> = mutableListOf(1, 2, 3)  // 使用 List 而不是 MutableList
```

3. **提高代码可读性时**:

```kotlin
fun processData(): Result<Data> { ... }

val result: Result<Data> = processData()  // 明确返回类型
```

### 2.3 基本数据类型

Kotlin 的所有类型都是对象，没有 Java 的原始类型（primitive types）。

#### 数字类型

|类型|大小|范围|示例|
|---|---|---|---|
|Byte|8位|-128 to 127|`val b: Byte = 127`|
|Short|16位|-32768 to 32767|`val s: Short = 32767`|
|Int|32位|-2³¹ to 2³¹-1|`val i = 42`|
|Long|64位|-2⁶³ to 2⁶³-1|`val l = 123456789L`|
|Float|32位|单精度浮点数|`val f = 3.14f`|
|Double|64位|双精度浮点数|`val d = 3.14159`|

**示例**:

```kotlin
val byte: Byte = 127
val short: Short = 32767
val int = 42                    // Int (默认)
val long = 123456789L           // Long (需要 L 后缀)

val float = 3.14f               // Float (需要 f 后缀)
val double = 3.14159            // Double (默认)

// 下划线提高可读性
val million = 1_000_000
val hexBytes = 0xFF_EC_DE_5E
```

**类型转换**:

Kotlin 不支持隐式类型转换，必须显式调用转换函数：

```kotlin
val intValue: Int = 42
val longValue: Long = intValue.toLong()
val doubleValue: Double = intValue.toDouble()
val stringValue: String = intValue.toString()

// ❌ 错误的写法
val longValue: Long = intValue  // 编译错误
```

转换函数：

- `toByte()`, `toShort()`, `toInt()`, `toLong()`
- `toFloat()`, `toDouble()`
- `toChar()`, `toString()`

#### 布尔类型

```kotlin
val isValid: Boolean = true
val isComplete: Boolean = false

// 布尔运算
val result = true && false      // false (与)
val result = true || false      // true (或)
val result = !true              // false (非)
```

#### 字符类型

```kotlin
val letter: Char = 'A'
val digit: Char = '9'
val emoji: Char = '😀'

// 特殊字符
val newline: Char = '\n'
val tab: Char = '\t'
val quote: Char = '\''
```

### 2.4 字符串（String）

#### 基本用法

```kotlin
val name = "Kotlin"
val greeting = "Hello, World!"

// 字符串拼接
val message = "Hello" + " " + "Kotlin"

// 多行字符串
val text = "Line 1\nLine 2\nLine 3"
```

#### 字符串模板（String Templates）

这是 Kotlin 最实用的特性之一：

```kotlin
val name = "Alice"
val age = 25

// 简单变量插值
val message = "Hello, $name!"              // "Hello, Alice!"

// 表达式插值
val info = "Age: $age, Next year: ${age + 1}"

// 函数调用
val len = "Length: ${name.length}"

// 复杂表达式
val result = "Max: ${if (a > b) a else b}"
```

**对比 Java**:

```java
// Java
String message = "Hello, " + name + "!";
String info = "Age: " + age + ", Next year: " + (age + 1);

// Kotlin
val message = "Hello, $name!"
val info = "Age: $age, Next year: ${age + 1}"
```

#### 原始字符串（Raw Strings）

使用三重引号 `"""` 创建多行字符串，保留所有格式：

```kotlin
val text = """
    This is line 1
    This is line 2
    This is line 3
"""

// 去除前导空格
val text = """
    |This is line 1
    |This is line 2
    |This is line 3
""".trimMargin()

// JSON 示例
val json = """
    {
        "name": "Alice",
        "age": 25,
        "city": "Beijing"
    }
"""
```

#### 字符串操作

```kotlin
val str = "Kotlin"

// 长度
val length = str.length                 // 6

// 访问字符
val first = str[0]                      // 'K'
val last = str[str.length - 1]          // 'n'

// 子串
val sub = str.substring(0, 3)           // "Kot"

// 大小写
val upper = str.uppercase()             // "KOTLIN"
val lower = str.lowercase()             // "kotlin"

// 包含
val contains = str.contains("tl")       // true

// 替换
val replaced = str.replace("Kot", "Jav") // "Javlin"

// 分割
val parts = "a,b,c".split(",")          // ["a", "b", "c"]

// 去除空格
val trimmed = "  hello  ".trim()        // "hello"
```

---

## 🔧 第三部分：函数

### 3.1 函数声明

基本语法：

```kotlin
fun functionName(param1: Type1, param2: Type2): ReturnType {
    // 函数体
    return result
}
```

**示例**:

```kotlin
// 基本函数
fun greet(name: String): String {
    return "Hello, $name!"
}

// 调用
val message = greet("Kotlin")
println(message)                 // "Hello, Kotlin!"
```

### 3.2 单表达式函数

当函数体只有一个表达式时，可以简化：

```kotlin
// 常规写法
fun add(a: Int, b: Int): Int {
    return a + b
}

// 单表达式函数
fun add(a: Int, b: Int): Int = a + b

// 类型推断（可省略返回类型）
fun add(a: Int, b: Int) = a + b
```

**更多示例**:

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b

fun isEven(n: Int) = n % 2 == 0

fun double(x: Int) = x * 2

fun square(x: Int) = x * x
```

### 3.3 无返回值函数

使用 `Unit` 类型（类似 Java 的 `void`），可以省略：

```kotlin
// 显式声明 Unit
fun printMessage(message: String): Unit {
    println(message)
}

// 省略 Unit（推荐）
fun printMessage(message: String) {
    println(message)
}

// 单表达式函数
fun printDouble(x: Int) = println(x * 2)
```

### 3.4 默认参数

Kotlin 支持函数参数的默认值：

```kotlin
fun greet(name: String = "Guest", greeting: String = "Hello") {
    println("$greeting, $name!")
}

// 调用方式
greet()                          // "Hello, Guest!"
greet("Alice")                   // "Hello, Alice!"
greet("Bob", "Hi")              // "Hi, Bob!"
greet(greeting = "Hey")         // "Hey, Guest!"
```

**实用示例**:

```kotlin
fun createUser(
    name: String,
    age: Int = 0,
    city: String = "Unknown",
    isActive: Boolean = true
) {
    println("User: $name, $age, $city, Active: $isActive")
}

createUser("Alice")
createUser("Bob", age = 25)
createUser("Charlie", city = "Beijing", age = 30)
```

### 3.5 命名参数（Named Arguments）

调用函数时可以指定参数名，提高可读性：

```kotlin
fun sendEmail(
    to: String,
    subject: String,
    body: String,
    cc: String? = null,
    bcc: String? = null
) {
    // 发送邮件
}

// 使用命名参数
sendEmail(
    to = "alice@example.com",
    subject = "Meeting Reminder",
    body = "Don't forget the meeting at 3 PM",
    cc = "bob@example.com"
)

// 混合使用位置参数和命名参数
sendEmail(
    "alice@example.com",
    "Meeting Reminder",
    body = "Don't forget...",
    cc = "bob@example.com"
)
```

**优点**:

1. 提高代码可读性
2. 参数顺序灵活
3. 避免参数混淆

### 3.6 可变参数（Vararg）

使用 `vararg` 关键字接受可变数量的参数：

```kotlin
fun sum(vararg numbers: Int): Int {
    var total = 0
    for (num in numbers) {
        total += num
    }
    return total
}

// 调用
val result1 = sum(1, 2, 3)              // 6
val result2 = sum(1, 2, 3, 4, 5)        // 15

// 展开数组
val numbers = intArrayOf(1, 2, 3, 4, 5)
val result3 = sum(*numbers)             // 15 (使用 * 展开)
```

---

## 🔀 第四部分：控制流

### 4.1 if 表达式

Kotlin 的 `if` 是表达式，可以返回值！

#### 基本用法

```kotlin
val a = 10
val b = 20

// 传统用法
if (a > b) {
    println("a is larger")
} else {
    println("b is larger")
}

// 作为表达式
val max = if (a > b) a else b

// 多行表达式
val max = if (a > b) {
    println("Choosing a")
    a
} else {
    println("Choosing b")
    b
}
```

#### 不需要三元运算符

Java 的三元运算符 `condition ? a : b` 在 Kotlin 中不存在，因为 `if` 表达式就足够了：

```kotlin
// Java
int max = (a > b) ? a : b;

// Kotlin
val max = if (a > b) a else b
```

#### 实用示例

```kotlin
// 成绩等级
val grade = if (score >= 90) "A"
            else if (score >= 80) "B"
            else if (score >= 70) "C"
            else if (score >= 60) "D"
            else "F"

// 验证输入
val input = readLine()
val number = if (input != null && input.toIntOrNull() != null) {
    input.toInt()
} else {
    0
}
```

### 4.2 when 表达式

`when` 是 Kotlin 对 switch 的升级版，更强大、更灵活。

#### 基本用法

```kotlin
val x = 3

when (x) {
    1 -> println("One")
    2 -> println("Two")
    3 -> println("Three")
    else -> println("Other")
}

// 作为表达式
val result = when (x) {
    1 -> "One"
    2 -> "Two"
    3 -> "Three"
    else -> "Other"
}
```

#### 多个值

```kotlin
when (x) {
    1, 2 -> println("One or Two")
    3 -> println("Three")
    in 4..10 -> println("Between 4 and 10")
    else -> println("Other")
}
```

#### 范围和条件

```kotlin
val age = 25

when (age) {
    in 0..12 -> println("Child")
    in 13..17 -> println("Teenager")
    in 18..64 -> println("Adult")
    in 65..120 -> println("Senior")
    else -> println("Invalid age")
}

// 使用任意表达式
when {
    age < 18 -> println("Minor")
    age in 18..64 -> println("Adult")
    else -> println("Senior")
}
```

#### 智能类型转换

```kotlin
fun describe(obj: Any): String = when (obj) {
    1 -> "One"
    "Hello" -> "Greeting"
    is Long -> "Long number"
    is String -> "String of length ${obj.length}"
    is IntArray -> "Array of ints"
    else -> "Unknown"
}
```

#### 替代 if-else 链

```kotlin
// ❌ 不好的写法
val description = if (x == 1) {
    "One"
} else if (x == 2) {
    "Two"
} else if (x in 3..10) {
    "Between 3 and 10"
} else {
    "Other"
}

// ✅ 更好的写法
val description = when (x) {
    1 -> "One"
    2 -> "Two"
    in 3..10 -> "Between 3 and 10"
    else -> "Other"
}
```

### 4.3 for 循环

#### 范围（Ranges）

```kotlin
// 闭区间：1 到 5（包含5）
for (i in 1..5) {
    println(i)              // 1, 2, 3, 4, 5
}

// 开区间：1 到 4（不包含5）
for (i in 1 until 5) {
    println(i)              // 1, 2, 3, 4
}

// 递减
for (i in 5 downTo 1) {
    println(i)              // 5, 4, 3, 2, 1
}

// 步长
for (i in 1..10 step 2) {
    println(i)              // 1, 3, 5, 7, 9
}

// 递减 + 步长
for (i in 10 downTo 1 step 2) {
    println(i)              // 10, 8, 6, 4, 2
}
```

#### 遍历集合

```kotlin
val fruits = listOf("Apple", "Banana", "Cherry")

// 遍历元素
for (fruit in fruits) {
    println(fruit)
}

// 遍历索引和元素
for ((index, fruit) in fruits.withIndex()) {
    println("$index: $fruit")
}

// 遍历索引
for (i in fruits.indices) {
    println("$i: ${fruits[i]}")
}
```

#### 遍历 Map

```kotlin
val map = mapOf(
    "name" to "Alice",
    "age" to "25",
    "city" to "Beijing"
)

for ((key, value) in map) {
    println("$key = $value")
}
```

### 4.4 while 循环

```kotlin
// while 循环
var i = 1
while (i <= 5) {
    println(i)
    i++
}

// do-while 循环
var j = 1
do {
    println(j)
    j++
} while (j <= 5)
```

**实用示例**:

```kotlin
// 读取输入直到输入有效
var input: String?
do {
    print("Enter a positive number: ")
    input = readLine()
    val number = input?.toIntOrNull()
} while (number == null || number <= 0)

println("You entered: $input")
```

### 4.5 break 和 continue

```kotlin
// break - 跳出循环
for (i in 1..10) {
    if (i == 5) break
    println(i)              // 1, 2, 3, 4
}

// continue - 跳过当前迭代
for (i in 1..10) {
    if (i % 2 == 0) continue
    println(i)              // 1, 3, 5, 7, 9
}

// 标签（用于嵌套循环）
outer@ for (i in 1..3) {
    for (j in 1..3) {
        if (i == 2 && j == 2) break@outer
        println("$i, $j")
    }
}
```

---

## 🛡️ 第五部分：空安全（Null Safety）

这是 Kotlin 最重要的特性之一！

### 5.1 为什么需要空安全？

**NullPointerException (NPE)** 是 Java 开发中最常见的错误：

```java
// Java - 容易出错
String name = getName();
int length = name.length();      // 💥 如果 name 为 null，抛出 NPE
```

Kotlin 通过类型系统从根本上解决了这个问题。

### 5.2 可空类型（Nullable Types）

Kotlin 的类型系统区分可空和非空引用：

```kotlin
// 非空类型 - 不能为 null
var name: String = "Kotlin"
name = null                      // ❌ 编译错误

// 可空类型 - 可以为 null
var nullableName: String? = "Kotlin"
nullableName = null              // ✅ 允许
```

**关键点**:

- `String` 表示非空字符串
- `String?` 表示可空字符串
- `?` 是类型的一部分

### 5.3 安全调用操作符 `?.`

```kotlin
val name: String? = null

// ❌ 错误的写法
val length = name.length         // 编译错误

// ✅ 安全调用
val length = name?.length        // 返回 Int? (可能是 null)
```

**工作原理**:

- 如果 `name` 不为 null，返回 `name.length`
- 如果 `name` 为 null，返回 `null`

**链式调用**:

```kotlin
val city = person?.address?.city

// 等价于
val city = if (person != null && person.address != null) {
    person.address.city
} else {
    null
}
```

### 5.4 Elvis 操作符 `?:`

提供默认值：

```kotlin
val name: String? = null

// 如果 name 为 null，使用 "Guest"
val displayName = name ?: "Guest"

// 等价于
val displayName = if (name != null) name else "Guest"
```

**组合使用**:

```kotlin
val length = name?.length ?: 0

// name 为 null 时，length = 0
// name 不为 null 时，length = name.length
```

**实用示例**:

```kotlin
fun greet(name: String?) {
    val displayName = name ?: "Guest"
    println("Hello, $displayName!")
}

greet("Alice")                   // "Hello, Alice!"
greet(null)                      // "Hello, Guest!"
```

### 5.5 非空断言 `!!`

**警告**: 谨慎使用！

```kotlin
val name: String? = "Kotlin"
val length = name!!.length       // 如果 name 为 null，抛出 NPE
```

**何时使用**:

- 你 100% 确定值不为 null
- 通常不应该使用，除非有充分理由

**更好的替代方案**:

```kotlin
// ❌ 不好
val length = name!!.length

// ✅ 更好
val length = name?.length ?: 0

// ✅ 或者显式检查
if (name != null) {
    val length = name.length     // 智能转换为 String
}
```

### 5.6 安全转换 `as?`

```kotlin
val obj: Any = "Hello"

// 不安全的转换
val str: String = obj as String  // 如果类型不匹配，抛出异常

// 安全的转换
val str: String? = obj as? String // 如果类型不匹配，返回 null
```

**实用示例**:

```kotlin
fun printLength(obj: Any) {
    val str = obj as? String
    val length = str?.length ?: 0
    println("Length: $length")
}

printLength("Hello")             // Length: 5
printLength(123)                 // Length: 0
```

### 5.7 let 函数

只在非 null 时执行代码块：

```kotlin
val name: String? = "Kotlin"

name?.let {
    println("Name is $it")
    println("Length is ${it.length}")
}

// 如果 name 为 null，let 块不会执行
```

**实用示例**:

```kotlin
fun processUser(name: String?) {
    name?.let {
        println("Processing user: $it")
        saveToDatabase(it)
        sendWelcomeEmail(it)
    } ?: println("No user to process")
}
```

### 5.8 空安全最佳实践

1. **优先使用非空类型**:

```kotlin
// ✅ 好
fun greet(name: String) { ... }

// ❌ 不好（除非真的需要可空）
fun greet(name: String?) { ... }
```

2. **使用默认值**:

```kotlin
val name = getName() ?: "Unknown"
```

3. **提前返回**:

```kotlin
fun process(data: Data?) {
    val validData = data ?: return
    // 现在 validData 是非空的
}
```

4. **避免 `!!`**:

```kotlin
// ❌ 避免
val length = name!!.length

// ✅ 更好
val length = name?.length ?: 0
```

---

## ⚖️ 第六部分：Kotlin vs Java

### 6.1 对比总结

|特性|Kotlin|Java|
|---|---|---|
|变量声明|`val` / `var`|`final` / 变量|
|空安全|内置（`String?`）|需要手动检查或使用注解|
|字符串模板|`"Hello $name"`|`"Hello " + name`|
|类型推断|完全支持|部分支持（Java 10+）|
|函数声明|`fun name() {}`|`type name() {}`|
|默认参数|✅ 支持|❌ 需要重载|
|命名参数|✅ 支持|❌ 不支持|
|when vs switch|更强大，支持任意类型|仅支持整数、字符、枚举、字符串|
|扩展函数|✅ 支持|❌ 不支持|
|数据类|`data class` 一行|需要大量样板代码|
|智能转换|自动转换|需要手动转换|

### 6.2 代码对比示例

#### 示例1：简单的数据类

**Kotlin**:

```kotlin
data class Person(val name: String, val age: Int)
```

**Java**:

```java
public class Person {
    private final String name;
    private final int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + '}';
    }
}
```

#### 示例2：空值处理

**Kotlin**:

```kotlin
val length = name?.length ?: 0
```

**Java**:

```java
int length = (name != null) ? name.length() : 0;
```

#### 示例3：列表过滤

**Kotlin**:

```kotlin
val adults = people.filter { it.age >= 18 }
```

**Java**:

```java
List<Person> adults = people.stream()
    .filter(person -> person.getAge() >= 18)
    .collect(Collectors.toList());
```

### 6.3 互操作性

Kotlin 和 Java 可以在同一项目中无缝协作：

**在 Kotlin 中调用 Java**:

```kotlin
// Java 类
public class JavaClass {
    public String getName() { return "Java"; }
}

// Kotlin 中使用
val javaObj = JavaClass()
val name = javaObj.name          // 自动转换为属性访问
```

**在 Java 中调用 Kotlin**:

```kotlin
// Kotlin 文件
class KotlinClass {
    fun greet(name: String) = "Hello, $name!"
}
```

```java
// Java 中使用
KotlinClass kotlinObj = new KotlinClass();
String greeting = kotlinObj.greet("World");
```

---

## 📝 实验练习

### 练习1：变量和类型

```kotlin
fun main() {
    // 练习：声明以下变量
    // 1. 不可变变量 name，值为你的名字
    // 2. 可变变量 age，值为你的年龄
    // 3. 使用类型推断声明 price = 99.99
    // 4. 使用字符串模板打印："我是 [name]，今年 [age] 岁"
    
    // 你的代码:
    
}
```

<details> <summary>点击查看答案</summary>

```kotlin
fun main() {
    val name = "Alice"
    var age = 25
    val price = 99.99
    
    println("我是 $name，今年 $age 岁")
}
```

</details>

### 练习2：函数

创建以下函数：

```kotlin
// 1. 计算两个数的最大值
fun max(a: Int, b: Int): Int {
    // 你的代码
}

// 2. 判断一个数是否为偶数
fun isEven(number: Int): Boolean {
    // 你的代码
}

// 3. 格式化用户信息（使用默认参数）
fun formatUser(name: String, age: Int = 0, city: String = "Unknown"): String {
    // 你的代码
}
```

<details> <summary>点击查看答案</summary>

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b

fun isEven(number: Int) = number % 2 == 0

fun formatUser(name: String, age: Int = 0, city: String = "Unknown") =
    "User: $name, Age: $age, City: $city"
```

</details>

### 练习3：控制流

```kotlin
// 使用 when 实现成绩等级判断
fun getGrade(score: Int): String {
    // 90-100: A
    // 80-89: B
    // 70-79: C
    // 60-69: D
    // <60: F
    
    // 你的代码
}

// 计算 1 到 n 的和
fun sumUpTo(n: Int): Int {
    // 你的代码
}
```

<details> <summary>点击查看答案</summary>

```kotlin
fun getGrade(score: Int) = when {
    score >= 90 -> "A"
    score >= 80 -> "B"
    score >= 70 -> "C"
    score >= 60 -> "D"
    else -> "F"
}

fun sumUpTo(n: Int): Int {
    var sum = 0
    for (i in 1..n) {
        sum += i
    }
    return sum
}

// 或使用区间的 sum() 函数
fun sumUpTo(n: Int) = (1..n).sum()
```

</details>

---

## 📚 课后作业

### 作业要求

完成以下5个编程练习，提交 Kotlin 代码文件。

#### 题目1：最大值函数

编写一个函数 `max`，接受两个整数参数，返回较大的那个。

- 要求使用 `if` 表达式
- 使用单表达式函数形式

```kotlin
fun max(a: Int, b: Int): Int = // 你的代码

fun main() {
    println(max(10, 20))        // 应该输出 20
    println(max(-5, 3))         // 应该输出 3
}
```

#### 题目2：成绩等级

使用 `when` 表达式编写函数 `getLetterGrade`，根据分数返回等级：

- 90-100: "A"
- 80-89: "B"
- 70-79: "C"
- 60-69: "D"
- 0-59: "F"
- 其他: "Invalid"

```kotlin
fun getLetterGrade(score: Int): String = // 你的代码

fun main() {
    println(getLetterGrade(95))    // A
    println(getLetterGrade(82))    // B
    println(getLetterGrade(55))    // F
}
```

#### 题目3：求和函数

编写函数 `sumRange`，计算从 1 到 n 的所有整数之和。

- 使用 `for` 循环
- 参数 n 为正整数

```kotlin
fun sumRange(n: Int): Int {
    // 你的代码
}

fun main() {
    println(sumRange(5))       // 15 (1+2+3+4+5)
    println(sumRange(10))      // 55
}
```

#### 题目4：空安全处理

编写函数 `getStringLength`，接受一个可空字符串参数，返回字符串长度。

- 如果字符串为 null，返回 0
- 使用空安全操作符

```kotlin
fun getStringLength(str: String?): Int = // 你的代码

fun main() {
    println(getStringLength("Kotlin"))    // 6
    println(getStringLength(null))        // 0
    println(getStringLength(""))          // 0
}
```

#### 题目5：用户信息格式化

编写函数 `formatUserInfo`，接受用户姓名、年龄和城市，返回格式化的字符串。

- 使用字符串模板
- 年龄和城市有默认值
- 使用命名参数调用

```kotlin
fun formatUserInfo(
    name: String,
    age: Int = 18,
    city: String = "Beijing"
): String {
    // 返回格式："Name: [name], Age: [age], City: [city]"
    // 你的代码
}

fun main() {
    println(formatUserInfo("Alice"))
    println(formatUserInfo("Bob", age = 25))
    println(formatUserInfo("Charlie", city = "Shanghai", age = 30))
}
```

### 提交要求

**文件格式**:

- 创建一个 Kotlin 文件：`Week2_Homework.kt`
- 包含所有5个函数及测试代码

**提交内容**:

1. 源代码文件
2. 运行截图（显示所有测试输出）

**截止时间**: 第3周周一上课前

**评分标准**:

- 功能正确性（50%）
- 代码风格（30%） - 遵循 Kotlin 最佳实践
- 注释说明（20%）

---

## 🎯 课后思考题

1. **为什么 Kotlin 推荐使用 `val` 而不是 `var`？列举至少3个理由。**
    
2. **Kotlin 的空安全如何从根本上解决 NullPointerException 问题？**
    
3. **`when` 表达式相比 Java 的 `switch` 有哪些优势？**
    
4. **什么情况下应该使用 `!!` 非空断言操作符？为什么要谨慎使用？**
    
5. **函数的默认参数和命名参数如何提高代码的可读性和灵活性？**
    

---

## 📚 扩展阅读

### 官方文档

- [Kotlin 官方文档](https://kotlinlang.org/docs/home.html)
- [Kotlin 基础教程](https://kotlinlang.org/docs/basic-syntax.html)
- [Kotlin 习惯用法](https://kotlinlang.org/docs/idioms.html)

### 在线练习

- [Kotlin Koans](https://play.kotlinlang.org/koans) - 交互式练习
- [Kotlin Playground](https://play.kotlinlang.org/) - 在线编程环境

### 推荐书籍

- _Kotlin in Action_ by Dmitry Jemerov and Svetlana Isakova
- _Head First Kotlin_ by Dawn Griffiths and David Griffiths
- _Atomic Kotlin_ by Bruce Eckel and Svetlana Isakova

### 视频教程

- [Kotlin for Beginners](https://www.youtube.com/playlist?list=PLlxmoA0rQ-LwgK1JsnMsakYNACYGa1cjR) - Google Developers
- [Kotlin Course](https://www.youtube.com/watch?v=F9UC9DY-vIU) - freeCodeCamp

---

## 🎓 下周预告

**第3周：UI基础 - Views与布局**

- Android UI 组件体系
- 常用 View 组件（TextView, Button, ImageView, EditText）
- LinearLayout 线性布局
- ConstraintLayout 约束布局
- 实践：创建登录界面

**预习建议**:

- 巩固本周的 Kotlin 知识
- 了解 Android 的 XML 布局文件
- 思考：如何使用代码创建 UI？

---

**课程资料更新时间**: 2026年1月  
**讲师**: [教师姓名]  
**联系方式**: [邮箱]

_Keep coding in Kotlin! 🚀_