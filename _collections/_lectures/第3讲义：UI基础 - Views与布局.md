---
title: 第3讲义：UI基础 - Views与布局
show: true
date: 2025-09-30
permalink: /lectures/lecture03
---


**课程**: Android 移动应用开发入门  
**周次**: 第3周  
**主题**: UI基础 - Views与布局  
**学时**: 3课时（理论2课时 + 实验1课时）

---

## 📋 本节课学习目标

完成本节课后，学生应该能够：

1. ✅ 理解 Android UI 系统的基本架构和组件层次
2. ✅ 掌握常用 View 组件（TextView, Button, ImageView, EditText）的使用
3. ✅ 理解 Android 中的尺寸单位（dp, sp, px）及其应用场景
4. ✅ 使用 XML 创建用户界面布局
5. ✅ 熟练使用 LinearLayout 进行线性布局
6. ✅ 掌握 ConstraintLayout 的约束布局技巧
7. ✅ 实践创建一个完整的登录界面

---

## 📱 第一部分：Android UI 概述

### 1.1 什么是 Android UI？

Android 的用户界面（UI）是用户与应用交互的视觉层。一个好的 UI 设计应该：

- **直观易用**: 用户无需学习即可理解
- **美观**: 符合 Material Design 设计规范
- **响应式**: 适配不同屏幕尺寸和方向
- **性能优良**: 流畅的用户体验

### 1.2 UI 组件层次结构

Android 的所有 UI 组件都继承自 `View` 类，形成一个清晰的层次结构：

```
View (基类)
│
├── ViewGroup (容器类，可包含其他 View)
│   ├── LinearLayout
│   ├── RelativeLayout
│   ├── FrameLayout
│   ├── ConstraintLayout
│   ├── GridLayout
│   └── ...
│
└── Widget (UI 控件)
    ├── TextView
    ├── Button
    ├── ImageView
    ├── EditText
    ├── CheckBox
    ├── RadioButton
    └── ...
```

**关键概念**:

- **View**: 所有 UI 组件的基类，代表屏幕上的一个矩形区域
- **ViewGroup**: View 的子类，可以包含其他 View，是布局容器的基类
- **Widget**: 具体的 UI 控件，如按钮、文本框等

### 1.3 View 的关键属性

所有 View 都具有以下基本属性：

#### 尺寸属性

```xml
android:layout_width="..."
android:layout_height="..."
```

可选值：

- `match_parent`: 与父容器相同大小
- `wrap_content`: 根据内容自适应大小
- 具体数值: 如 `100dp`

#### ID 属性

```xml
android:id="@+id/myView"
```

- 用于在代码中引用该 View
- `@+id/` 表示创建新 ID
- `@id/` 表示引用已存在的 ID

#### 内边距和外边距

```xml
android:padding="16dp"          <!-- 内边距（四周） -->
android:paddingTop="8dp"        <!-- 顶部内边距 -->
android:paddingBottom="8dp"     <!-- 底部内边距 -->
android:paddingStart="16dp"     <!-- 起始边内边距 -->
android:paddingEnd="16dp"       <!-- 结束边内边距 -->

android:layout_margin="16dp"    <!-- 外边距（四周） -->
android:layout_marginTop="8dp"  <!-- 顶部外边距 -->
<!-- 其他外边距属性类似 -->
```

**区别**:

- **Padding**: 内容与边界的距离（内边距）
- **Margin**: View 与其他 View 的距离（外边距）

#### 可见性

```xml
android:visibility="visible|invisible|gone"
```

- `visible`: 可见（默认）
- `invisible`: 不可见但占据空间
- `gone`: 不可见且不占据空间

---

## 📏 第二部分：尺寸单位详解

### 2.1 Android 中的尺寸单位

Android 支持多种尺寸单位，选择正确的单位对于跨设备适配至关重要。

#### dp (Density-independent Pixels) - 密度无关像素

**定义**:

- 与屏幕密度无关的抽象单位
- 1 dp 在 160 dpi 屏幕上等于 1 物理像素

**特点**:

- ✅ **推荐使用** - 适配不同密度屏幕
- ✅ 用于定义 View 的宽度、高度、边距等
- ✅ 在不同密度设备上保持相似的物理尺寸

**使用场景**:

```xml
android:layout_width="100dp"
android:layout_height="50dp"
android:padding="16dp"
android:layout_margin="8dp"
```

#### sp (Scale-independent Pixels) - 缩放无关像素

**定义**:

- 与 dp 类似，但会根据用户的字体大小偏好缩放

**特点**:

- ✅ **专门用于文字大小**
- ✅ 尊重用户的可访问性设置
- ✅ 提升应用的可用性

**使用场景**:

```xml
android:textSize="16sp"     <!-- 正文文字 -->
android:textSize="24sp"     <!-- 标题文字 -->
android:textSize="12sp"     <!-- 小字 -->
```

**推荐的文字大小**:

|用途|大小|
|---|---|
|大标题|24-34sp|
|小标题|20-24sp|
|正文|14-16sp|
|说明文字|12-14sp|
|按钮文字|14-16sp|

#### px (Pixels) - 像素

**定义**:

- 屏幕上的实际像素点

**特点**:

- ❌ **不推荐使用**
- ❌ 在不同密度设备上显示大小不一致
- ❌ 破坏跨设备兼容性

**何时使用**:

- 极少数特殊情况（如处理位图像素级操作）
- 一般情况下应避免使用

### 2.2 屏幕密度详解

Android 定义了几种标准密度：

|密度分类|DPI 范围|缩放因子|示例设备|
|---|---|---|---|
|ldpi|~120 dpi|0.75|旧设备|
|mdpi|~160 dpi|1.0|基准密度|
|hdpi|~240 dpi|1.5|中端设备|
|xhdpi|~320 dpi|2.0|高端设备|
|xxhdpi|~480 dpi|3.0|超高端设备|
|xxxhdpi|~640 dpi|4.0|极高端设备|

**转换公式**:

```
px = dp × (dpi / 160)
```

**示例**:

- 在 mdpi (160 dpi) 设备上: 100dp = 100px
- 在 xhdpi (320 dpi) 设备上: 100dp = 200px
- 在 xxhdpi (480 dpi) 设备上: 100dp = 300px

这就是为什么使用 dp 可以保证在不同设备上物理尺寸相近！

---

## 🎨 第三部分：常用 View 组件

### 3.1 TextView - 文本显示

**用途**: 显示文本信息，是最基础和常用的 UI 组件。

#### 基本使用

```xml
<TextView
    android:id="@+id/textView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hello Android!" />
```

#### 常用属性

##### 文本内容

```xml
<!-- 直接设置文本 -->
android:text="显示的文字"

<!-- 引用字符串资源（推荐） -->
android:text="@string/hello_message"
```

**💡 最佳实践**: 将文本字符串放在 `res/values/strings.xml` 中：

```xml
<!-- res/values/strings.xml -->
<resources>
    <string name="hello_message">你好，Android！</string>
    <string name="app_name">我的应用</string>
</resources>
```

##### 文本样式

```xml
<!-- 字体大小 -->
android:textSize="16sp"

<!-- 文字颜色 -->
android:textColor="#028090"
android:textColor="@color/primary"

<!-- 文字样式 -->
android:textStyle="normal|bold|italic"

<!-- 字体 -->
android:fontFamily="sans-serif"
android:fontFamily="@font/my_custom_font"

<!-- 文字对齐 -->
android:textAlignment="start|end|center"

<!-- 行间距 -->
android:lineSpacingExtra="4dp"
android:lineSpacingMultiplier="1.2"
```

##### 文本修饰

```xml
<!-- 全部大写 -->
android:textAllCaps="true"

<!-- 单行显示 -->
android:singleLine="true"

<!-- 最大行数 -->
android:maxLines="3"

<!-- 超出显示省略号 -->
android:ellipsize="end|start|middle|marquee"
```

#### Kotlin 代码操作

```kotlin
val textView = findViewById<TextView>(R.id.textView)

// 设置文本
textView.text = "新的文本"

// 设置文本颜色
textView.setTextColor(Color.parseColor("#028090"))

// 设置文本大小（单位 sp，需要转换）
textView.textSize = 18f

// 设置样式
textView.setTypeface(null, Typeface.BOLD)
```

### 3.2 Button - 按钮

**用途**: 可点击的按钮控件，触发用户操作。

#### 基本使用

```xml
<Button
    android:id="@+id/button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="点击我" />
```

**注意**: Button 继承自 TextView，因此 TextView 的所有属性都适用于 Button。

#### 常用属性

```xml
<Button
    android:id="@+id/loginButton"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="登录"
    android:textSize="16sp"
    android:textColor="@color/white"
    android:backgroundTint="@color/primary"
    android:padding="12dp"
    android:enabled="true" />
```

##### 按钮样式

Android Material Design 提供了多种按钮样式：

```xml
<!-- 填充按钮（默认） -->
<Button
    style="@style/Widget.Material3.Button"
    ... />

<!-- 轮廓按钮 -->
<Button
    style="@style/Widget.Material3.Button.OutlinedButton"
    ... />

<!-- 文本按钮 -->
<Button
    style="@style/Widget.Material3.Button.TextButton"
    ... />

<!-- 图标按钮 -->
<Button
    style="@style/Widget.Material3.Button.Icon"
    app:icon="@drawable/ic_favorite"
    ... />
```

#### 点击事件处理

**方法 1: XML 中设置 onClick**

```xml
<Button
    android:id="@+id/button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="点击"
    android:onClick="onButtonClick" />
```

```kotlin
// MainActivity.kt
class MainActivity : AppCompatActivity() {
    fun onButtonClick(view: View) {
        Toast.makeText(this, "按钮被点击了！", Toast.LENGTH_SHORT).show()
    }
}
```

**方法 2: 在 Kotlin 代码中设置监听器（推荐）**

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        val button = findViewById<Button>(R.id.button)
        
        // 方式 1: 传统写法
        button.setOnClickListener(object : View.OnClickListener {
            override fun onClick(v: View?) {
                Toast.makeText(this@MainActivity, "按钮被点击", Toast.LENGTH_SHORT).show()
            }
        })
        
        // 方式 2: Lambda 表达式（推荐）
        button.setOnClickListener {
            Toast.makeText(this, "按钮被点击", Toast.LENGTH_SHORT).show()
        }
        
        // 方式 3: 方法引用
        button.setOnClickListener(this::handleButtonClick)
    }
    
    private fun handleButtonClick(view: View) {
        Toast.makeText(this, "按钮被点击", Toast.LENGTH_SHORT).show()
    }
}
```

### 3.3 ImageView - 图片显示

**用途**: 显示图片资源。

#### 基本使用

```xml
<ImageView
    android:id="@+id/imageView"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:src="@drawable/logo"
    android:contentDescription="@string/logo_description" />
```

**💡 重要**: 始终为 ImageView 设置 `contentDescription`，这对于视障用户的可访问性至关重要。

#### scaleType 属性

`scaleType` 控制图片如何缩放和显示：

```xml
<!-- 保持宽高比，缩放至完全显示，可能留白 -->
android:scaleType="fitCenter"

<!-- 保持宽高比，缩放至填满，可能裁剪 -->
android:scaleType="centerCrop"

<!-- 保持宽高比，居中显示，不缩放 -->
android:scaleType="center"

<!-- 拉伸填满（不推荐，会变形） -->
android:scaleType="fitXY"

<!-- 其他选项 -->
android:scaleType="centerInside|fitStart|fitEnd"
```

**常用场景**:

- 头像、缩略图 → `centerCrop`
- Logo、图标 → `fitCenter` 或 `center`
- 背景图 → `centerCrop`

#### 在代码中设置图片

```kotlin
val imageView = findViewById<ImageView>(R.id.imageView)

// 方式 1: 从资源 ID 加载
imageView.setImageResource(R.drawable.ic_launcher)

// 方式 2: 从 Bitmap 加载
val bitmap = BitmapFactory.decodeResource(resources, R.drawable.photo)
imageView.setImageBitmap(bitmap)

// 方式 3: 从 Drawable 加载
val drawable = ContextCompat.getDrawable(this, R.drawable.icon)
imageView.setImageDrawable(drawable)

// 设置着色
imageView.setColorFilter(Color.parseColor("#028090"))
```

### 3.4 EditText - 文本输入

**用途**: 接收用户的文本输入。

#### 基本使用

```xml
<EditText
    android:id="@+id/editText"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="请输入内容"
    android:inputType="text" />
```

#### 常用属性

##### 提示文本

```xml
<!-- 占位提示文字 -->
android:hint="请输入用户名"
android:hint="@string/username_hint"

<!-- 提示文字颜色 -->
android:textColorHint="#999999"
```

##### 输入类型 (inputType)

不同的 `inputType` 会调用不同的键盘：

```xml
<!-- 普通文本 -->
android:inputType="text"

<!-- 多行文本 -->
android:inputType="textMultiLine"

<!-- 密码 -->
android:inputType="textPassword"

<!-- 数字 -->
android:inputType="number"

<!-- 带小数的数字 -->
android:inputType="numberDecimal"

<!-- 电话号码 -->
android:inputType="phone"

<!-- 电子邮件 -->
android:inputType="textEmailAddress"

<!-- 网址 -->
android:inputType="textUri"

<!-- 组合使用（大写 + 自动纠错） -->
android:inputType="textCapWords|textAutoCorrect"
```

##### 其他重要属性

```xml
<!-- 最大长度 -->
android:maxLength="20"

<!-- 行数 -->
android:lines="3"           <!-- 固定行数 -->
android:minLines="2"        <!-- 最小行数 -->
android:maxLines="5"        <!-- 最大行数 -->

<!-- 自动换行 -->
android:scrollbars="vertical"

<!-- 禁用拼写检查 -->
android:inputType="textNoSuggestions"
```

#### 获取和设置文本

```kotlin
val editText = findViewById<EditText>(R.id.editText)

// 获取输入的文本
val text = editText.text.toString()

// 设置文本
editText.setText("默认文本")

// 清空文本
editText.text.clear()

// 监听文本变化
editText.addTextChangedListener(object : TextWatcher {
    override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {
        // 文本变化前
    }
    
    override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
        // 文本变化中
        Log.d("EditText", "当前文本: $s")
    }
    
    override fun afterTextChanged(s: Editable?) {
        // 文本变化后
    }
})
```

#### 实用示例：表单验证

```kotlin
fun validateForm() {
    val usernameEdit = findViewById<EditText>(R.id.usernameEdit)
    val passwordEdit = findViewById<EditText>(R.id.passwordEdit)
    
    val username = usernameEdit.text.toString().trim()
    val password = passwordEdit.text.toString()
    
    when {
        username.isEmpty() -> {
            usernameEdit.error = "用户名不能为空"
            usernameEdit.requestFocus()
        }
        username.length < 3 -> {
            usernameEdit.error = "用户名至少3个字符"
            usernameEdit.requestFocus()
        }
        password.isEmpty() -> {
            passwordEdit.error = "密码不能为空"
            passwordEdit.requestFocus()
        }
        password.length < 6 -> {
            passwordEdit.error = "密码至少6个字符"
            passwordEdit.requestFocus()
        }
        else -> {
            // 验证通过，执行登录
            performLogin(username, password)
        }
    }
}
```

---

## 📐 第四部分：LinearLayout - 线性布局

### 4.1 LinearLayout 概述

LinearLayout 是最简单和最常用的布局之一，它将子 View 按照线性顺序（水平或垂直）排列。

**特点**:

- ✅ 简单直观
- ✅ 适合简单的线性排列
- ✅ 支持权重分配
- ❌ 不适合复杂嵌套（影响性能）

### 4.2 orientation 方向

LinearLayout 有两个方向：

#### 垂直方向 (vertical)

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    
    <!-- 子 View 从上到下排列 -->
    <TextView ... />
    <EditText ... />
    <Button ... />
    
</LinearLayout>
```

视觉效果：

```
┌─────────────┐
│  TextView   │
├─────────────┤
│  EditText   │
├─────────────┤
│   Button    │
└─────────────┘
```

#### 水平方向 (horizontal)

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">
    
    <!-- 子 View 从左到右排列 -->
    <Button ... />
    <Button ... />
    <Button ... />
    
</LinearLayout>
```

视觉效果：

```
┌──────┬──────┬──────┐
│Button│Button│Button│
└──────┴──────┴──────┘
```

### 4.3 gravity 和 layout_gravity

这两个属性经常容易混淆，需要明确区分：

#### gravity - 内容对齐

控制 **子 View 在容器中的对齐方式**：

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">  <!-- 子 View 居中 -->
    
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="按钮" />
        
</LinearLayout>
```

常用值：

- `center`: 水平和垂直居中
- `center_horizontal`: 水平居中
- `center_vertical`: 垂直居中
- `start|top`: 左上角
- `end|bottom`: 右下角

#### layout_gravity - 自身对齐

控制 **View 自己在父容器中的位置**：

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"  <!-- 按钮自己居中 -->
        android:text="按钮" />
        
</LinearLayout>
```

**💡 记忆技巧**:

- `gravity` = 我管别人怎么排（容器视角）
- `layout_gravity` = 我自己怎么排（View 视角）

### 4.4 weight 权重分配

`layout_weight` 是 LinearLayout 最强大的特性之一，用于按比例分配剩余空间。

#### 基本用法

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">
    
    <Button
        android:layout_width="0dp"          <!-- 使用 weight 时设为 0dp -->
        android:layout_height="wrap_content"
        android:layout_weight="1"           <!-- 占 1 份 -->
        android:text="按钮1" />
    
    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="2"           <!-- 占 2 份 -->
        android:text="按钮2" />
        
</LinearLayout>
```

**结果**: 按钮1 占 1/3 宽度，按钮2 占 2/3 宽度

#### 权重计算原理

1. 首先，LinearLayout 计算所有 **非 weight** 子 View 的尺寸
2. 然后，将**剩余空间**按照 weight 比例分配

**示例**:

```xml
<LinearLayout
    android:layout_width="match_parent"  <!-- 假设屏幕宽度 300dp -->
    android:layout_height="wrap_content"
    android:orientation="horizontal">
    
    <Button
        android:layout_width="100dp"     <!-- 固定 100dp -->
        android:layout_height="wrap_content"
        android:text="固定" />
    
    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"        <!-- 占剩余的 1/2 -->
        android:text="弹性1" />
    
    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"        <!-- 占剩余的 1/2 -->
        android:text="弹性2" />
        
</LinearLayout>
```

**计算**:

- 总宽度: 300dp
- 固定按钮: 100dp
- 剩余空间: 200dp
- 弹性1: 200dp × 1/2 = 100dp
- 弹性2: 200dp × 1/2 = 100dp

#### 常见应用场景

**场景 1: 均分空间**

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">
    
    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="1" />
    
    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="2" />
    
    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="3" />
        
</LinearLayout>
```

结果: 三个按钮平均分配宽度

**场景 2: 左固定右弹性**

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">
    
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="标签:" />
    
    <EditText
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:hint="填充剩余空间" />
        
</LinearLayout>
```

### 4.5 LinearLayout 完整示例

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    android:gravity="center_horizontal">
    
    <!-- Logo -->
    <ImageView
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:src="@drawable/logo"
        android:layout_marginBottom="24dp"
        android:contentDescription="@string/app_logo" />
    
    <!-- 标题 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="欢迎登录"
        android:textSize="24sp"
        android:textStyle="bold"
        android:layout_marginBottom="32dp" />
    
    <!-- 用户名 -->
    <EditText
        android:id="@+id/usernameEdit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入用户名"
        android:inputType="text"
        android:padding="12dp"
        android:layout_marginBottom="16dp" />
    
    <!-- 密码 -->
    <EditText
        android:id="@+id/passwordEdit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入密码"
        android:inputType="textPassword"
        android:padding="12dp"
        android:layout_marginBottom="24dp" />
    
    <!-- 按钮组 -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        
        <Button
            android:id="@+id/registerButton"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="注册"
            android:layout_marginEnd="8dp"
            style="@style/Widget.Material3.Button.OutlinedButton" />
        
        <Button
            android:id="@+id/loginButton"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="登录"
            android:layout_marginStart="8dp" />
            
    </LinearLayout>
    
</LinearLayout>
```

---

## 🎯 第五部分：ConstraintLayout - 约束布局

### 5.1 为什么需要 ConstraintLayout？

传统布局（如 LinearLayout、RelativeLayout）在构建复杂界面时存在一些问题：

**问题 1: 深层嵌套**

```xml
<!-- 传统方式 - 需要多层嵌套 -->
<LinearLayout orientation="vertical">
    <LinearLayout orientation="horizontal">
        <LinearLayout orientation="vertical">
            <TextView ... />
            <EditText ... />
        </LinearLayout>
        <Button ... />
    </LinearLayout>
    <LinearLayout orientation="horizontal">
        ...
    </LinearLayout>
</LinearLayout>
```

**问题**:

- 布局层级深，影响性能
- XML 代码难以维护
- 布局计算耗时增加

**ConstraintLayout 解决方案 - 扁平化**:

```xml
<!-- ConstraintLayout - 扁平化布局 -->
<androidx.constraintlayout.widget.ConstraintLayout>
    <TextView ... />
    <EditText ... />
    <Button ... />
    <!-- 所有 View 都在同一层级 -->
</androidx.constraintlayout.widget.ConstraintLayout>
```

### 5.2 ConstraintLayout 核心概念

**约束 (Constraint)** 是 ConstraintLayout 的核心：每个 View 通过约束关系确定自己的位置。

**基本规则**:

- 每个 View 必须至少有 **1 个水平约束** 和 **1 个垂直约束**
- 约束可以相对于：父布局、其他 View、辅助线(Guideline)、屏障(Barrier)

### 5.3 基本约束类型

#### 1. 相对于父布局的约束

```xml
<TextView
    android:id="@+id/textView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hello"
    
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintStart_toStartOf="parent" />
```

**效果**: TextView 在左上角

**常用属性**:

```xml
app:layout_constraintTop_toTopOf="parent"       <!-- 顶部对齐父布局顶部 -->
app:layout_constraintBottom_toBottomOf="parent" <!-- 底部对齐父布局底部 -->
app:layout_constraintStart_toStartOf="parent"   <!-- 起始边对齐父布局起始边 -->
app:layout_constraintEnd_toEndOf="parent"       <!-- 结束边对齐父布局结束边 -->
```

**💡 提示**: 使用 `Start/End` 而不是 `Left/Right`，以支持从右到左（RTL）的语言。

#### 2. 相对于其他 View 的约束

```xml
<Button
    android:id="@+id/button1"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="按钮1"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintStart_toStartOf="parent" />

<Button
    android:id="@+id/button2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="按钮2"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintStart_toEndOf="@id/button1" />  <!-- 在 button1 右边 -->
```

**效果**: button2 在 button1 的右侧，顶部对齐

**可用约束**:

```xml
app:layout_constraintStart_toEndOf="@id/view"    <!-- 我的起始边 对齐 目标的结束边 -->
app:layout_constraintEnd_toStartOf="@id/view"    <!-- 我的结束边 对齐 目标的起始边 -->
app:layout_constraintTop_toBottomOf="@id/view"   <!-- 我的顶部 对齐 目标的底部 -->
app:layout_constraintBottom_toTopOf="@id/view"   <!-- 我的底部 对齐 目标的顶部 -->
```

### 5.4 居中和偏移

#### 居中

通过设置**相反方向**的约束实现居中：

```xml
<!-- 水平居中 -->
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent" />

<!-- 垂直居中 -->
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toBottomOf="parent" />

<!-- 完全居中 -->
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent" />
```

#### 偏移 (Bias)

在居中的基础上添加偏移：

```xml
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintHorizontal_bias="0.3" />  <!-- 30% 位置 -->
```

- `bias = 0.0`: 完全靠左/上
- `bias = 0.5`: 居中（默认）
- `bias = 1.0`: 完全靠右/下

### 5.5 尺寸约束

ConstraintLayout 提供了特殊的尺寸值：

```xml
<!-- 0dp (MATCH_CONSTRAINT) - 根据约束计算尺寸 -->
<TextView
    android:layout_width="0dp"  <!-- 宽度由约束决定 -->
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent" />

<!-- 结果: TextView 宽度等于父布局宽度 -->
```

**尺寸比例**:

```xml
<!-- 宽高比 16:9 -->
<ImageView
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintDimensionRatio="16:9"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

### 5.6 链 (Chains)

链可以在一组 View 之间均匀分布空间。

#### 创建链

```xml
<Button
    android:id="@+id/button1"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toStartOf="@id/button2"
    app:layout_constraintHorizontal_chainStyle="spread" />

<Button
    android:id="@+id/button2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toEndOf="@id/button1"
    app:layout_constraintEnd_toStartOf="@id/button3" />

<Button
    android:id="@+id/button3"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toEndOf="@id/button2"
    app:layout_constraintEnd_toEndOf="parent" />
```

#### 链样式

```xml
app:layout_constraintHorizontal_chainStyle="spread"        <!-- 均匀分布 -->
app:layout_constraintHorizontal_chainStyle="spread_inside" <!-- 两端不留空 -->
app:layout_constraintHorizontal_chainStyle="packed"        <!-- 紧密排列 -->
```

### 5.7 辅助工具

#### Guideline - 辅助线

不可见的辅助线，用于对齐：

```xml
<!-- 垂直辅助线，距离起始边 30% -->
<androidx.constraintlayout.widget.Guideline
    android:id="@+id/guideline"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    app:layout_constraintGuide_percent="0.3" />

<!-- 使用辅助线 -->
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="@id/guideline" />
```

**定位方式**:

```xml
app:layout_constraintGuide_begin="100dp"   <!-- 距离起始边 100dp -->
app:layout_constraintGuide_end="100dp"     <!-- 距离结束边 100dp -->
app:layout_constraintGuide_percent="0.5"   <!-- 50% 位置 -->
```

#### Barrier - 屏障

根据多个 View 的边界创建屏障：

```xml
<androidx.constraintlayout.widget.Barrier
    android:id="@+id/barrier"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:barrierDirection="end"
    app:constraint_referenced_ids="textView1,textView2,textView3" />

<!-- 确保 Button 在所有 TextView 之后 -->
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toEndOf="@id/barrier" />
```

### 5.8 ConstraintLayout 完整示例

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <!-- Logo - 顶部居中 -->
    <ImageView
        android:id="@+id/logo"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:src="@drawable/logo"
        android:contentDescription="@string/app_logo"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="32dp" />

    <!-- 标题 - Logo 下方居中 -->
    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="欢迎登录"
        android:textSize="24sp"
        android:textStyle="bold"
        app:layout_constraintTop_toBottomOf="@id/logo"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="16dp" />

    <!-- 用户名标签 -->
    <TextView
        android:id="@+id/usernameLabel"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="用户名"
        app:layout_constraintTop_toBottomOf="@id/title"
        app:layout_constraintStart_toStartOf="parent"
        android:layout_marginTop="32dp" />

    <!-- 用户名输入框 - 标签右侧，填充剩余宽度 -->
    <EditText
        android:id="@+id/usernameEdit"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:hint="请输入用户名"
        android:inputType="text"
        app:layout_constraintTop_toTopOf="@id/usernameLabel"
        app:layout_constraintBottom_toBottomOf="@id/usernameLabel"
        app:layout_constraintStart_toEndOf="@id/usernameLabel"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginStart="16dp" />

    <!-- 密码标签 -->
    <TextView
        android:id="@+id/passwordLabel"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="密码"
        app:layout_constraintTop_toBottomOf="@id/usernameEdit"
        app:layout_constraintStart_toStartOf="parent"
        android:layout_marginTop="16dp" />

    <!-- 密码输入框 -->
    <EditText
        android:id="@+id/passwordEdit"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:hint="请输入密码"
        android:inputType="textPassword"
        app:layout_constraintTop_toTopOf="@id/passwordLabel"
        app:layout_constraintBottom_toBottomOf="@id/passwordLabel"
        app:layout_constraintStart_toEndOf="@id/passwordLabel"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginStart="16dp" />

    <!-- 创建屏障 - 确保按钮在标签之后 -->
    <androidx.constraintlayout.widget.Barrier
        android:id="@+id/labelBarrier"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:barrierDirection="end"
        app:constraint_referenced_ids="usernameLabel,passwordLabel" />

    <!-- 注册按钮 -->
    <Button
        android:id="@+id/registerButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="注册"
        style="@style/Widget.Material3.Button.OutlinedButton"
        app:layout_constraintTop_toBottomOf="@id/passwordEdit"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toStartOf="@id/loginButton"
        android:layout_marginTop="24dp"
        android:layout_marginEnd="8dp" />

    <!-- 登录按钮 -->
    <Button
        android:id="@+id/loginButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="登录"
        app:layout_constraintTop_toTopOf="@id/registerButton"
        app:layout_constraintStart_toEndOf="@id/registerButton"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginStart="8dp" />

    <!-- 忘记密码 - 底部居中 -->
    <TextView
        android:id="@+id/forgotPassword"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="忘记密码？"
        android:textColor="@color/primary"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginBottom="32dp" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

---

## 💡 第六部分：布局最佳实践

### 6.1 性能优化

#### 1. 减少布局层级

**❌ 不好的做法**:

```xml
<LinearLayout>
    <LinearLayout>
        <LinearLayout>
            <TextView ... />
        </LinearLayout>
    </LinearLayout>
</LinearLayout>
```

**✅ 好的做法**:

```xml
<ConstraintLayout>
    <TextView ... />
</ConstraintLayout>
```

#### 2. 使用 `<merge>` 标签

当根布局会被嵌入到其他布局时：

```xml
<!-- custom_view.xml -->
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView ... />
    <Button ... />
</merge>
```

#### 3. 使用 `<include>` 复用布局

```xml
<!-- header.xml -->
<LinearLayout ...>
    <ImageView ... />
    <TextView ... />
</LinearLayout>

<!-- 在主布局中使用 -->
<LinearLayout>
    <include layout="@layout/header" />
    <!-- 其他内容 -->
</LinearLayout>
```

### 6.2 可维护性

#### 1. 提取资源

**字符串**:

```xml
<!-- strings.xml -->
<string name="login_button">登录</string>
<string name="username_hint">请输入用户名</string>

<!-- 使用 -->
android:text="@string/login_button"
android:hint="@string/username_hint"
```

**颜色**:

```xml
<!-- colors.xml -->
<color name="primary">#028090</color>
<color name="on_primary">#FFFFFF</color>

<!-- 使用 -->
android:textColor="@color/primary"
android:background="@color/on_primary"
```

**尺寸**:

```xml
<!-- dimens.xml -->
<dimen name="margin_standard">16dp</dimen>
<dimen name="text_size_body">16sp</dimen>

<!-- 使用 -->
android:layout_margin="@dimen/margin_standard"
android:textSize="@dimen/text_size_body"
```

#### 2. 使用样式 (Styles)

定义可复用的样式：

```xml
<!-- styles.xml -->
<style name="PrimaryButton" parent="Widget.Material3.Button">
    <item name="android:layout_width">0dp</item>
    <item name="android:layout_height">wrap_content</item>
    <item name="android:textSize">16sp</item>
    <item name="android:padding">12dp</item>
</style>

<!-- 使用 -->
<Button
    style="@style/PrimaryButton"
    android:text="登录" />
```

### 6.3 调试技巧

#### 1. 使用 tools 命名空间

在 XML 中预览数据，不影响运行时：

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:text="预览文本（仅在设计器中显示）"
    android:text="@string/actual_text" />
```

#### 2. Layout Inspector

在 Android Studio 中：

```
Tools → Layout Inspector
```

可以查看运行时的布局层级和属性。

#### 3. 显示布局边界

在开发者选项中启用"显示布局边界"，可以直观看到所有 View 的边界。

---

## 📝 实验作业

### 作业：设计登录界面

**目标**: 使用本节课学习的知识，创建一个美观实用的登录界面。

#### 要求

**功能要求**:

1. 使用 **ConstraintLayout** 作为根布局
2. 必须包含以下元素：
    - 应用 Logo（ImageView）
    - 标题文字（TextView）
    - 用户名输入框（EditText）
    - 密码输入框（EditText）
    - 登录按钮（Button）
    - 注册按钮（Button，可选）

**设计要求**:

1. 布局美观，元素对齐
2. 合理使用 margin 和 padding
3. 使用 dp 和 sp 作为尺寸单位
4. 颜色搭配和谐

**代码要求**:

1. 所有字符串使用 strings.xml
2. 颜色值定义在 colors.xml
3. 代码格式规范，添加注释

#### 提交内容

1. **activity_main.xml** 布局文件完整代码
2. **运行截图** - 展示在模拟器/真机上的实际效果
3. **设计说明**（200字以内）:
    - 布局选择的理由
    - 设计思路
    - 遇到的问题和解决方案

#### 评分标准

|项目|分值|说明|
|---|---|---|
|功能完整性|30%|包含所有必需元素|
|布局合理性|30%|正确使用 ConstraintLayout，布局扁平|
|美观程度|20%|界面美观，元素对齐|
|代码规范|20%|资源提取，命名规范，有注释|

#### 加分项（可选）

- ✨ 添加"忘记密码"链接
- ✨ 添加第三方登录按钮（微信、QQ等图标）
- ✨ 使用自定义主题和样式
- ✨ 添加输入验证提示
- ✨ 创意设计

#### 提交方式

**截止时间**: 第4周周一上课前

**文件命名**: `姓名_学号_Week3_登录界面.zip`

**压缩包包含**:

- `activity_main.xml`
- `strings.xml`
- `colors.xml`
- `screenshot.png`（运行截图）
- `README.txt`（设计说明）

---

## 🎯 课后思考题

1. **为什么 Android 推荐使用 ConstraintLayout 而不是其他布局？列举至少3个原因。**
    
2. **什么情况下应该使用 LinearLayout？什么情况下应该使用 ConstraintLayout？**
    
3. **解释 `match_parent`, `wrap_content`, 和 `0dp` (在 ConstraintLayout 中) 的区别。**
    
4. **在 LinearLayout 中使用 layout_weight 时，为什么要将对应方向的 layout_width/height 设为 0dp？**
    
5. **如何在 ConstraintLayout 中实现以下布局需求：**
    
    - View A 在屏幕正中央
    - View B 在 View A 下方 20dp 处，水平居中
    - View C 和 View D 在 View B 下方，左右均分屏幕宽度

---

## 📚 扩展阅读

### 官方文档

- [View 官方文档](https://developer.android.com/reference/android/view/View)
- [ConstraintLayout 官方指南](https://developer.android.com/training/constraint-layout)
- [布局最佳实践](https://developer.android.com/topic/performance/rendering/optimizing-view-hierarchies)

### Material Design

- [Material Design 组件](https://m3.material.io/components)
- [Material Design 布局指南](https://m3.material.io/foundations/layout/understanding-layout/overview)

### 视频教程

- [Android Developers - Build a Responsive UI](https://www.youtube.com/watch?v=9mUF5P0kxdU)
- [ConstraintLayout 深入教程](https://www.youtube.com/watch?v=XamMbnzI5vE)

### 实用工具

- [Android Asset Studio](https://romannurik.github.io/AndroidAssetStudio/) - 图标生成工具
- [Material Color Tool](https://material.io/resources/color/) - 配色方案工具

---

## 🎓 下周预告

**第4周：Activity生命周期与Intent**

- Activity 生命周期详解
- 生命周期回调方法
- Intent 与 Activity 跳转
- Activity 间数据传递
- 实践：多页面应用开发

**预习建议**:

- 复习本周的 UI 知识，下周会用到
- 思考：为什么需要了解 Activity 生命周期？
- 尝试创建多个 Activity（提前体验）

---

**课程资料更新时间**: 2026年1月  
**讲师**: [教师姓名]  
**联系方式**: [邮箱]

_继续加油！UI 设计是 Android 开发的重要基础！🎨_
