---
title: 第9周讲义：导航组件（Navigation Component）
date: 2026-02-05
permalink: /lectures/lecture09
show: true
---


**课程**: Android 移动应用开发入门  
**周次**: 第9周  
**主题**: 导航组件（Navigation Component）  
**学时**: 3课时（理论2课时 + 实验1课时）

---

## 📋 本节课学习目标

完成本节课后，学生应该能够：

1. ✅ 理解 Navigation Component 的核心概念和优势
2. ✅ 创建和配置 Navigation Graph
3. ✅ 使用 NavController 实现 Fragment 导航
4. ✅ 使用 SafeArgs 传递类型安全的参数
5. ✅ 集成 BottomNavigationView 实现底部导航
6. ✅ 处理返回栈和深层链接
7. ✅ 开发具有多个 Tab 的应用框架

---

## 🗺️ 第一部分：Navigation Component 概述

### 1.1 什么是 Navigation Component？

**Navigation Component** 是 Google 推出的**官方导航框架**，简化了 Fragment 之间的导航管理。

**传统导航 vs Navigation Component**:

```
❌ 传统方式（手动管理 Fragment）:
supportFragmentManager.beginTransaction()
    .replace(R.id.container, DetailFragment.newInstance(id))
    .addToBackStack(null)
    .commit()

问题:
- 代码分散，难以维护
- 参数传递容易出错
- 回退栈管理复杂
- 没有可视化工具

✅ Navigation Component:
findNavController().navigate(
    R.id.action_list_to_detail,
    bundleOf("item_id" to id)
)

优势:
- 可视化编辑导航图
- 类型安全的参数传递
- 自动管理回退栈
- 深层链接支持
- 动画配置简单
```

### 1.2 核心组件

```
Navigation Component 三大核心:

┌─────────────────────────────────────┐
│  1. Navigation Graph (导航图)      │
│     nav_graph.xml                   │
│     ├─ Destination (目的地)         │
│     ├─ Action (导航动作)            │
│     └─ Argument (参数)              │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│  2. NavHost (导航宿主)              │
│     FragmentContainerView           │
│     承载导航内容的容器               │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│  3. NavController (导航控制器)      │
│     负责执行导航操作                 │
│     navigate(), popBackStack()      │
└─────────────────────────────────────┘
```

### 1.3 为什么使用 Navigation Component？

```
✅ 可视化: 在 XML 编辑器中看到完整导航流程
✅ 类型安全: SafeArgs 插件生成类型安全的代码
✅ 自动化: 自动处理 Fragment 事务和回退栈
✅ 标准化: Google 官方推荐的导航方案
✅ 深层链接: 轻松实现从外部 URL 打开特定页面
✅ 动画: 内置导航动画，配置简单
```

---

## 🔧 第二部分：配置 Navigation Component

### 2.1 添加依赖

**build.gradle (Project level)**:

```gradle
buildscript {
    repositories {
        google()
    }
    dependencies {
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:2.7.6"
    }
}
```

**build.gradle (Module: app)**:

```gradle
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'androidx.navigation.safeargs.kotlin'  // SafeArgs 插件
}

dependencies {
    def nav_version = "2.7.6"
    
    // Navigation 核心库
    implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
    implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
    
    // BottomNavigationView (如需使用)
    implementation 'com.google.android.material:material:1.11.0'
}
```

### 2.2 创建 Navigation Graph

**步骤**: 右键 `res` → New → Android Resource File

- File name: `nav_graph`
- Resource type: `Navigation`
- 点击 OK

**res/navigation/nav_graph.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <!-- 首页 Fragment -->
    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.myapp.HomeFragment"
        android:label="首页"
        tools:layout="@layout/fragment_home">
        
        <!-- 导航动作：从首页到详情页 -->
        <action
            android:id="@+id/action_home_to_detail"
            app:destination="@id/detailFragment"
            app:enterAnim="@anim/slide_in_right"
            app:exitAnim="@anim/slide_out_left"
            app:popEnterAnim="@anim/slide_in_left"
            app:popExitAnim="@anim/slide_out_right" />
    </fragment>

    <!-- 详情页 Fragment -->
    <fragment
        android:id="@+id/detailFragment"
        android:name="com.example.myapp.DetailFragment"
        android:label="详情"
        tools:layout="@layout/fragment_detail">
        
        <!-- 参数定义 -->
        <argument
            android:name="itemId"
            app:argType="integer"
            android:defaultValue="0" />
            
        <argument
            android:name="itemName"
            app:argType="string"
            app:nullable="true" />
    </fragment>

</navigation>
```

**关键属性解释**:

- `app:startDestination`: 启动时显示的第一个目的地
- `android:id`: Fragment 的唯一标识
- `android:name`: Fragment 的完整类名
- `app:destination`: 导航目标
- `app:argType`: 参数类型（integer, string, boolean, long, float, reference 等）

### 2.3 在 Activity 中添加 NavHost

**activity_main.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- NavHostFragment - Navigation 的容器 -->
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

**重要属性**:

- `android:name="androidx.navigation.fragment.NavHostFragment"`: 指定为 NavHostFragment
- `app:navGraph="@navigation/nav_graph"`: 关联导航图
- `app:defaultNavHost="true"`: 拦截系统返回键

### 2.4 MainActivity 配置

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var navController: NavController

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 获取 NavController
        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        navController = navHostFragment.navController

        // 可选: 设置 ActionBar 标题自动更新
        setupActionBarWithNavController(navController)
    }

    // 处理返回键（如果需要）
    override fun onSupportNavigateUp(): Boolean {
        return navController.navigateUp() || super.onSupportNavigateUp()
    }
}
```

---

## 🧭 第三部分：Fragment 导航

### 3.1 基本导航

#### 方式一: 使用 Action ID（推荐）

```kotlin
// HomeFragment.kt
class HomeFragment : Fragment() {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        view.findViewById<Button>(R.id.btnGoToDetail).setOnClickListener {
            // 使用在 nav_graph.xml 中定义的 action
            findNavController().navigate(R.id.action_home_to_detail)
        }
    }
}
```

#### 方式二: 使用 Destination ID

```kotlin
// 直接跳转到目标 Fragment（不推荐，因为丢失动画等配置）
findNavController().navigate(R.id.detailFragment)
```

### 3.2 传递参数（不使用 SafeArgs）

```kotlin
// 发送方 - HomeFragment
view.findViewById<Button>(R.id.btnGoToDetail).setOnClickListener {
    val bundle = bundleOf(
        "itemId" to 123,
        "itemName" to "商品A"
    )
    findNavController().navigate(R.id.action_home_to_detail, bundle)
}

// 接收方 - DetailFragment
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    val itemId = arguments?.getInt("itemId", 0) ?: 0
    val itemName = arguments?.getString("itemName", "") ?: ""
    
    view.findViewById<TextView>(R.id.textView).text = "ID: $itemId, 名称: $itemName"
}
```

**缺点**:

- 参数名容易拼错
- 类型不安全
- 没有编译时检查

### 3.3 返回上一页

```kotlin
// 方式一: 返回上一页
findNavController().navigateUp()

// 方式二: 弹出回退栈
findNavController().popBackStack()

// 方式三: 弹出到特定目的地
findNavController().popBackStack(R.id.homeFragment, false)
```

### 3.4 在 RecyclerView 中导航

```kotlin
class ItemAdapter(
    private val items: List<Item>
) : RecyclerView.Adapter<ItemViewHolder>() {

    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) {
        val item = items[position]
        holder.bind(item)
        
        holder.itemView.setOnClickListener {
            val bundle = bundleOf("itemId" to item.id)
            it.findNavController().navigate(
                R.id.action_list_to_detail,
                bundle
            )
        }
    }
}
```

---

## 🔒 第四部分：SafeArgs — 类型安全的参数传递

### 4.1 什么是 SafeArgs？

**SafeArgs** 是 Navigation 的 Gradle 插件，自动生成类型安全的参数传递代码。

**优势**:

```
❌ 传统方式:
val itemId = arguments?.getInt("itemId", 0)  // 字符串拼错不报错
val itemName = arguments?.getString("itemName")  // 类型转换可能出错

✅ SafeArgs:
val args = DetailFragmentArgs.fromBundle(requireArguments())
val itemId = args.itemId      // 编译时检查
val itemName = args.itemName  // 类型安全
```

### 4.2 配置 SafeArgs

**已在 2.1 节配置，确认以下内容**:

**build.gradle (Project)**:

```gradle
classpath "androidx.navigation:navigation-safe-args-gradle-plugin:2.7.6"
```

**build.gradle (Module: app)**:

```gradle
plugins {
    id 'androidx.navigation.safeargs.kotlin'  // Kotlin 版本
    // 或者 Java 版本: id 'androidx.navigation.safeargs'
}
```

### 4.3 使用 SafeArgs

#### 步骤一: 在 Navigation Graph 中定义参数

```xml
<fragment
    android:id="@+id/detailFragment"
    android:name="com.example.myapp.DetailFragment"
    android:label="详情">
    
    <argument
        android:name="itemId"
        app:argType="integer" />
        
    <argument
        android:name="itemName"
        app:argType="string"
        app:nullable="true"
        android:defaultValue="@null" />
        
    <argument
        android:name="price"
        app:argType="float"
        android:defaultValue="0.0" />
</fragment>
```

#### 步骤二: Build 项目生成代码

点击 **Build → Make Project**，SafeArgs 会自动生成以下类:

```
生成的类:
- HomeFragmentDirections     (发送方使用)
- DetailFragmentArgs         (接收方使用)
```

#### 步骤三: 发送参数

```kotlin
// HomeFragment.kt
class HomeFragment : Fragment() {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        view.findViewById<Button>(R.id.btnGoToDetail).setOnClickListener {
            // SafeArgs 自动生成的方法
            val action = HomeFragmentDirections.actionHomeToDetail(
                itemId = 123,
                itemName = "商品A",
                price = 99.9f
            )
            findNavController().navigate(action)
        }
    }
}
```

#### 步骤四: 接收参数

```kotlin
// DetailFragment.kt
class DetailFragment : Fragment() {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // SafeArgs 自动生成的 Args 类
        val args = DetailFragmentArgs.fromBundle(requireArguments())
        
        val itemId = args.itemId
        val itemName = args.itemName
        val price = args.price

        view.findViewById<TextView>(R.id.idText).text = "ID: $itemId"
        view.findViewById<TextView>(R.id.nameText).text = "名称: $itemName"
        view.findViewById<TextView>(R.id.priceText).text = "价格: ¥$price"
    }
}
```

### 4.4 传递自定义对象（Parcelable）

```kotlin
// 定义数据类
@Parcelize
data class Product(
    val id: Int,
    val name: String,
    val price: Float
) : Parcelable

// Navigation Graph 中定义
<argument
    android:name="product"
    app:argType="com.example.myapp.Product" />

// 发送
val product = Product(1, "商品A", 99.9f)
val action = HomeFragmentDirections.actionHomeToDetail(product)
findNavController().navigate(action)

// 接收
val args = DetailFragmentArgs.fromBundle(requireArguments())
val product = args.product
```

---

## 📱 第五部分：BottomNavigationView 集成

### 5.1 创建底部导航菜单

**res/menu/bottom_nav_menu.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    
    <item
        android:id="@+id/homeFragment"
        android:icon="@drawable/ic_home"
        android:title="首页" />
    
    <item
        android:id="@+id/messageFragment"
        android:icon="@drawable/ic_message"
        android:title="消息" />
    
    <item
        android:id="@+id/profileFragment"
        android:icon="@drawable/ic_profile"
        android:title="我的" />
        
</menu>
```

**⚠️ 重要**: Menu Item 的 `id` 必须与 Navigation Graph 中 Fragment 的 `id` **完全一致**

### 5.2 更新 Navigation Graph

```xml
<navigation ...
    app:startDestination="@id/homeFragment">

    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.myapp.HomeFragment"
        android:label="首页" />

    <fragment
        android:id="@+id/messageFragment"
        android:name="com.example.myapp.MessageFragment"
        android:label="消息" />

    <fragment
        android:id="@+id/profileFragment"
        android:name="com.example.myapp.ProfileFragment"
        android:label="我的" />

</navigation>
```

### 5.3 布局文件

**activity_main.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- NavHost -->
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/bottom_nav"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <!-- BottomNavigationView -->
    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_nav"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:background="@android:color/white"
        app:menu="@menu/bottom_nav_menu"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### 5.4 MainActivity 连接

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var navController: NavController

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        navController = navHostFragment.navController

        // 连接 BottomNavigationView 和 NavController
        val bottomNav = findViewById<BottomNavigationView>(R.id.bottom_nav)
        bottomNav.setupWithNavController(navController)
        
        // 就这一行代码！NavigationUI 自动处理切换逻辑
    }
}
```

**`setupWithNavController` 自动做了什么？**:

```
✅ 点击底部导航项 → 自动切换到对应 Fragment
✅ 自动高亮当前选中的 Tab
✅ 自动处理回退栈（不累积相同 Fragment）
✅ 自动防止重复点击同一个 Tab
```

---

## 🎨 第六部分：导航动画

### 6.1 定义动画资源

**res/anim/slide_in_right.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:fromXDelta="100%"
        android:toXDelta="0%"
        android:duration="300" />
</set>
```

**res/anim/slide_out_left.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:fromXDelta="0%"
        android:toXDelta="-100%"
        android:duration="300" />
</set>
```

**res/anim/slide_in_left.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:fromXDelta="-100%"
        android:toXDelta="0%"
        android:duration="300" />
</set>
```

**res/anim/slide_out_right.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:fromXDelta="0%"
        android:toXDelta="100%"
        android:duration="300" />
</set>
```

### 6.2 在 Action 中配置动画

```xml
<action
    android:id="@+id/action_home_to_detail"
    app:destination="@id/detailFragment"
    app:enterAnim="@anim/slide_in_right"
    app:exitAnim="@anim/slide_out_left"
    app:popEnterAnim="@anim/slide_in_left"
    app:popExitAnim="@anim/slide_out_right" />
```

**动画类型解释**:

- `enterAnim`: 目标 Fragment 进入动画
- `exitAnim`: 当前 Fragment 退出动画
- `popEnterAnim`: 按返回键时，上一个 Fragment 进入动画
- `popExitAnim`: 按返回键时，当前 Fragment 退出动画

---

## 🧪 第七部分：实验

### 7.1 实验一：配置 Navigation Graph

**目标**: 从零开始配置一个简单的导航系统。

#### 步骤一: 创建 Fragment

**HomeFragment.kt**:

```kotlin
class HomeFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_home, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        view.findViewById<Button>(R.id.btnNavigate).setOnClickListener {
            findNavController().navigate(R.id.action_home_to_detail)
        }
    }
}
```

**DetailFragment.kt**:

```kotlin
class DetailFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_detail, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        view.findViewById<Button>(R.id.btnBack).setOnClickListener {
            findNavController().navigateUp()
        }
    }
}
```

#### 步骤二: 创建 Navigation Graph

**res/navigation/nav_graph.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.navdemo.HomeFragment"
        android:label="Home"
        tools:layout="@layout/fragment_home">
        
        <action
            android:id="@+id/action_home_to_detail"
            app:destination="@id/detailFragment"
            app:enterAnim="@android:anim/slide_in_left"
            app:exitAnim="@android:anim/slide_out_right" />
    </fragment>

    <fragment
        android:id="@+id/detailFragment"
        android:name="com.example.navdemo.DetailFragment"
        android:label="Detail"
        tools:layout="@layout/fragment_detail" />

</navigation>
```

#### 步骤三: 更新 Activity 布局

**activity_main.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph" />

</FrameLayout>
```

#### 步骤四: MainActivity

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

**验证**: 运行应用，点击按钮应该能正常导航。

---

### 7.2 实验二：使用 SafeArgs 传递参数

**目标**: 配置 SafeArgs，实现类型安全的参数传递。

#### 步骤一: 配置插件（确认已添加）

**build.gradle (Module: app)**:

```gradle
plugins {
    id 'androidx.navigation.safeargs.kotlin'
}
```

#### 步骤二: 在 Navigation Graph 中定义参数

```xml
<fragment
    android:id="@+id/detailFragment"
    android:name="com.example.navdemo.DetailFragment"
    android:label="Detail">
    
    <argument
        android:name="itemId"
        app:argType="integer" />
    
    <argument
        android:name="itemTitle"
        app:argType="string" />
    
    <argument
        android:name="itemPrice"
        app:argType="float"
        android:defaultValue="0.0" />
        
</fragment>
```

#### 步骤三: Build 项目

点击 **Build → Rebuild Project**，等待生成代码。

#### 步骤四: 发送参数

**HomeFragment.kt**:

```kotlin
view.findViewById<Button>(R.id.btnNavigate).setOnClickListener {
    val action = HomeFragmentDirections.actionHomeToDetail(
        itemId = 100,
        itemTitle = "笔记本电脑",
        itemPrice = 5999.99f
    )
    findNavController().navigate(action)
}
```

#### 步骤五: 接收参数

**DetailFragment.kt**:

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    
    val args = DetailFragmentArgs.fromBundle(requireArguments())
    
    view.findViewById<TextView>(R.id.tvId).text = "ID: ${args.itemId}"
    view.findViewById<TextView>(R.id.tvTitle).text = "标题: ${args.itemTitle}"
    view.findViewById<TextView>(R.id.tvPrice).text = "价格: ¥${args.itemPrice}"
}
```

**验证**: 运行应用，参数应该正确传递并显示。

---

### 7.3 实验三：实现底部 Tab 导航

**目标**: 创建一个带三个 Tab 的底部导航应用。

#### 步骤一: 创建三个 Fragment

**HomeFragment.kt**:

```kotlin
class HomeFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_home, container, false)
    }
}
```

**MessageFragment.kt**:

```kotlin
class MessageFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_message, container, false)
    }
}
```

**ProfileFragment.kt**:

```kotlin
class ProfileFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_profile, container, false)
    }
}
```

#### 步骤二: 创建底部导航菜单

**res/menu/bottom_nav_menu.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/homeFragment"
        android:icon="@drawable/ic_home"
        android:title="首页" />
    <item
        android:id="@+id/messageFragment"
        android:icon="@drawable/ic_message"
        android:title="消息" />
    <item
        android:id="@+id/profileFragment"
        android:icon="@drawable/ic_person"
        android:title="我的" />
</menu>
```

#### 步骤三: 更新 Navigation Graph

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.navdemo.HomeFragment"
        android:label="首页" />

    <fragment
        android:id="@+id/messageFragment"
        android:name="com.example.navdemo.MessageFragment"
        android:label="消息" />

    <fragment
        android:id="@+id/profileFragment"
        android:name="com.example.navdemo.ProfileFragment"
        android:label="我的" />

</navigation>
```

#### 步骤四: 更新 Activity 布局

**activity_main.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/bottom_nav"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_nav"
        android:layout_width="0dp"
        android:layout_height="56dp"
        android:background="@android:color/white"
        app:menu="@menu/bottom_nav_menu"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### 步骤五: MainActivity

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        val navController = navHostFragment.navController

        val bottomNav = findViewById<BottomNavigationView>(R.id.bottom_nav)
        bottomNav.setupWithNavController(navController)
    }
}
```

**验证**: 运行应用，点击底部导航应该能切换 Fragment。

---

## 📝 第八部分：课后作业 — 开发三Tab应用框架

### 8.1 作业概述

开发一个具有三个 Tab 的应用框架，要求使用 Navigation Component 和 SafeArgs。

#### 应用主题（任选其一）

**主题一：电商应用框架**

```
Tab 1: 首页 (商品列表)
Tab 2: 购物车
Tab 3: 我的

功能:
- 首页显示商品列表（RecyclerView）
- 点击商品跳转详情页（使用 SafeArgs 传递商品信息）
- 详情页可添加到购物车
- 购物车显示已添加商品
- 我的页面显示用户信息
```

**主题二：新闻应用框架**

```
Tab 1: 新闻列表
Tab 2: 分类
Tab 3: 我的收藏

功能:
- 新闻列表显示标题和摘要
- 点击跳转新闻详情（使用 SafeArgs 传递新闻ID）
- 详情页可收藏
- 分类页显示不同分类的新闻
- 收藏页显示已收藏的新闻
```

**主题三：笔记应用框架**

```
Tab 1: 笔记列表
Tab 2: 分类
Tab 3: 设置

功能:
- 笔记列表显示所有笔记
- 点击跳转笔记详情/编辑（使用 SafeArgs）
- 分类页显示不同分类的笔记
- 设置页包含主题、字体大小等设置
```

### 8.2 必需功能

**✅ 底部导航**:

- 使用 BottomNavigationView
- 至少 3 个 Tab
- 正确配置 Navigation Graph

**✅ 导航跳转**:

- 从列表页跳转到详情页
- 使用 SafeArgs 传递参数
- 详情页可以正确接收参数

**✅ 回退栈管理**:

- 按返回键能正确返回上一页
- 不会累积相同的 Tab Fragment

**✅ 数据传递**:

- 至少传递 2 种不同类型的参数（如 Int + String）
- 使用 SafeArgs 实现类型安全

**✅ UI 设计**:

- 布局合理美观
- 使用 Material Design 组件

### 8.3 完整参考实现：电商应用框架

#### 项目结构

```
app/src/main/
├── java/com.example.shopapp/
│   ├── MainActivity.kt
│   ├── HomeFragment.kt
│   ├── CartFragment.kt
│   ├── ProfileFragment.kt
│   ├── ProductDetailFragment.kt
│   ├── Product.kt
│   └── ProductAdapter.kt
├── res/
│   ├── layout/
│   │   ├── activity_main.xml
│   │   ├── fragment_home.xml
│   │   ├── fragment_cart.xml
│   │   ├── fragment_profile.xml
│   │   ├── fragment_product_detail.xml
│   │   └── item_product.xml
│   ├── menu/
│   │   └── bottom_nav_menu.xml
│   ├── navigation/
│   │   └── nav_graph.xml
│   └── drawable/
│       └── (各种图标)
```

#### Product.kt

```kotlin
package com.example.shopapp

import android.os.Parcelable
import kotlinx.parcelize.Parcelize

@Parcelize
data class Product(
    val id: Int,
    val name: String,
    val price: Float,
    val description: String,
    val imageResId: Int
) : Parcelable
```

#### nav_graph.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <!-- 首页 -->
    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.shopapp.HomeFragment"
        android:label="首页"
        tools:layout="@layout/fragment_home">
        
        <action
            android:id="@+id/action_home_to_detail"
            app:destination="@id/productDetailFragment"
            app:enterAnim="@anim/slide_in_right"
            app:exitAnim="@anim/slide_out_left"
            app:popEnterAnim="@anim/slide_in_left"
            app:popExitAnim="@anim/slide_out_right" />
    </fragment>

    <!-- 购物车 -->
    <fragment
        android:id="@+id/cartFragment"
        android:name="com.example.shopapp.CartFragment"
        android:label="购物车"
        tools:layout="@layout/fragment_cart" />

    <!-- 我的 -->
    <fragment
        android:id="@+id/profileFragment"
        android:name="com.example.shopapp.ProfileFragment"
        android:label="我的"
        tools:layout="@layout/fragment_profile" />

    <!-- 商品详情 -->
    <fragment
        android:id="@+id/productDetailFragment"
        android:name="com.example.shopapp.ProductDetailFragment"
        android:label="商品详情"
        tools:layout="@layout/fragment_product_detail">
        
        <argument
            android:name="product"
            app:argType="com.example.shopapp.Product" />
    </fragment>

</navigation>
```

#### bottom_nav_menu.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/homeFragment"
        android:icon="@drawable/ic_home"
        android:title="首页" />
    <item
        android:id="@+id/cartFragment"
        android:icon="@drawable/ic_shopping_cart"
        android:title="购物车" />
    <item
        android:id="@+id/profileFragment"
        android:icon="@drawable/ic_person"
        android:title="我的" />
</menu>
```

#### HomeFragment.kt

```kotlin
package com.example.shopapp

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment
import androidx.navigation.fragment.findNavController
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class HomeFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_home, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        val recyclerView = view.findViewById<RecyclerView>(R.id.recyclerView)
        recyclerView.layoutManager = LinearLayoutManager(context)

        // 模拟商品数据
        val products = listOf(
            Product(1, "iPhone 15 Pro", 7999f, "最新款苹果手机", R.drawable.phone),
            Product(2, "MacBook Pro", 12999f, "专业笔记本电脑", R.drawable.laptop),
            Product(3, "AirPods Pro", 1999f, "主动降噪耳机", R.drawable.airpods),
            Product(4, "iPad Air", 4599f, "轻薄平板电脑", R.drawable.ipad),
            Product(5, "Apple Watch", 2999f, "智能手表", R.drawable.watch)
        )

        val adapter = ProductAdapter(products) { product ->
            // 使用 SafeArgs 导航到详情页
            val action = HomeFragmentDirections.actionHomeToDetail(product)
            findNavController().navigate(action)
        }
        recyclerView.adapter = adapter
    }
}
```

#### ProductDetailFragment.kt

```kotlin
package com.example.shopapp

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Button
import android.widget.ImageView
import android.widget.TextView
import android.widget.Toast
import androidx.fragment.app.Fragment
import androidx.navigation.fragment.findNavController
import androidx.navigation.fragment.navArgs

class ProductDetailFragment : Fragment() {

    private val args: ProductDetailFragmentArgs by navArgs()

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_product_detail, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        val product = args.product

        view.findViewById<ImageView>(R.id.productImage).setImageResource(product.imageResId)
        view.findViewById<TextView>(R.id.productName).text = product.name
        view.findViewById<TextView>(R.id.productPrice).text = "¥${product.price}"
        view.findViewById<TextView>(R.id.productDescription).text = product.description

        view.findViewById<Button>(R.id.btnAddToCart).setOnClickListener {
            Toast.makeText(context, "已添加到购物车", Toast.LENGTH_SHORT).show()
            // 这里可以使用 SharedViewModel 将商品添加到购物车
        }

        view.findViewById<Button>(R.id.btnBack).setOnClickListener {
            findNavController().navigateUp()
        }
    }
}
```

#### CartFragment.kt

```kotlin
package com.example.shopapp

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.fragment.app.Fragment

class CartFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_cart, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        view.findViewById<TextView>(R.id.emptyText).text = "购物车为空\n快去首页添加商品吧～"
    }
}
```

#### ProfileFragment.kt

```kotlin
package com.example.shopapp

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android:view.ViewGroup
import android.widget.TextView
import androidx.fragment.app.Fragment

class ProfileFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_profile, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        view.findViewById<TextView>(R.id.usernameText).text = "用户123456"
        view.findViewById<TextView>(R.id.emailText).text = "user@example.com"
    }
}
```

#### MainActivity.kt

```kotlin
package com.example.shopapp

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.navigation.fragment.NavHostFragment
import androidx.navigation.ui.setupWithNavController
import com.google.android.material.bottomnavigation.BottomNavigationView

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        val navController = navHostFragment.navController

        val bottomNav = findViewById<BottomNavigationView>(R.id.bottom_nav)
        bottomNav.setupWithNavController(navController)
    }
}
```

### 8.4 评分标准

|项目|分值|评分细则|
|---|---|---|
|Navigation Graph 配置|25%|正确配置所有目的地和动作|
|SafeArgs 使用|25%|正确使用 SafeArgs 传递和接收参数|
|BottomNavigation|20%|底部导航正常工作，切换流畅|
|导航逻辑|15%|跳转和返回逻辑正确，回退栈合理|
|UI 设计|10%|界面美观，布局合理|
|代码质量|5%|代码结构清晰，命名规范|

#### 加分项（可选，每项+5分，上限+15分）

- ✨ 使用 SharedViewModel 在 Fragment 间共享数据
- ✨ 添加导航动画效果
- ✨ 实现深层链接（Deep Link）
- ✨ 使用 DataBinding 或 ViewBinding
- ✨ 添加侧滑菜单（Navigation Drawer）

#### 提交要求

1. **源代码** — 完整 Android 项目（ZIP 压缩包）
2. **运行截图** — 至少6张（三个Tab页面、详情页、导航过程、参数传递）
3. **说明文档** — README.md（功能介绍、Navigation Graph 设计、SafeArgs 使用说明）
4. **导航图截图** — Navigation Editor 的可视化截图

**文件命名**: `姓名_学号_Week9_Navigation应用.zip`  
**截止时间**: 第10周周一上课前

---

## 💡 第九部分：最佳实践与常见坑

### 9.1 Navigation Component 常见坑

```kotlin
// ─── 坑①: Menu Item ID 与 Fragment ID 不一致 ───
// ❌ 错误
// menu.xml:  android:id="@+id/home"
// nav_graph.xml:  android:id="@+id/homeFragment"
// 结果: setupWithNavController 无法工作

// ✅ 正确: ID 必须完全一致
// menu.xml:  android:id="@+id/homeFragment"
// nav_graph.xml:  android:id="@+id/homeFragment"

// ─── 坑②: 忘记 Build 项目导致 SafeArgs 类找不到 ───
// 修改 nav_graph.xml 后必须 Build 项目
// Build → Make Project (Ctrl+F9)

// ─── 坑③: 在 Fragment 中直接使用 Activity 的 NavController ───
// ❌ 错误
class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        (activity as MainActivity).navController.navigate(...)  // 耦合
    }
}

// ✅ 正确: 使用 Fragment 自己的 NavController
class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        findNavController().navigate(...)  // 解耦
    }
}

// ─── 坑④: 在非 Fragment 视图中调用 findNavController ───
// ❌ 错误 (在 RecyclerView.ViewHolder 中)
class MyViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    init {
        itemView.setOnClickListener {
            findNavController().navigate(...)  // 找不到 NavController!
        }
    }
}

// ✅ 正确: 使用 View.findNavController()
class MyViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    init {
        itemView.setOnClickListener {
            it.findNavController().navigate(...)  // OK
        }
    }
}

// ─── 坑⑤: BottomNavigation 重复添加相同 Fragment 到回退栈 ───
// 使用 setupWithNavController 自动避免此问题
// 如果手动处理，需要检查当前 destination:

bottomNav.setOnItemSelectedListener { item ->
    if (navController.currentDestination?.id != item.itemId) {
        navController.navigate(item.itemId)
    }
    true
}
```

### 9.2 SafeArgs 最佳实践

```kotlin
// ─── 最佳实践①: 使用 data class 封装多个参数 ───
// ❌ 不推荐: 传递多个独立参数
<argument android:name="userId" app:argType="integer" />
<argument android:name="userName" app:argType="string" />
<argument android:name="userEmail" app:argType="string" />
<argument android:name="userAge" app:argType="integer" />

// ✅ 推荐: 使用 Parcelable 封装
<argument android:name="user" app:argType="com.example.User" />

@Parcelize
data class User(
    val id: Int,
    val name: String,
    val email: String,
    val age: Int
) : Parcelable

// ─── 最佳实践②: 设置合理的默认值 ───
<argument
    android:name="category"
    app:argType="string"
    android:defaultValue="all" />  // 有默认值，参数可选

// ─── 最佳实践③: 使用 nullable 处理可选参数 ───
<argument
    android:name="searchQuery"
    app:argType="string"
    app:nullable="true"
    android:defaultValue="@null" />

// 接收时
val query = args.searchQuery
if (query != null) {
    performSearch(query)
} else {
    showAllItems()
}
```

### 9.3 回退栈管理技巧

```kotlin
// ─── 技巧①: 清空回退栈并跳转到首页 ───
navController.navigate(R.id.homeFragment) {
    popUpTo(R.id.nav_graph) {
        inclusive = true  // 清空所有回退栈
    }
}

// ─── 技巧②: 跳转时避免累积相同 Fragment ───
navController.navigate(R.id.detailFragment) {
    popUpTo(R.id.detailFragment) {
        inclusive = true  // 移除已有的同类 Fragment
    }
}

// ─── 技巧③: 登录后跳转首页（不允许返回登录页）───
navController.navigate(R.id.action_login_to_home) {
    popUpTo(R.id.loginFragment) {
        inclusive = true  // 移除登录页
    }
}
```

### 9.4 性能优化

```kotlin
// ─── 优化①: 使用 ViewBinding 替代 findViewById ───
// build.gradle
android {
    buildFeatures {
        viewBinding = true
    }
}

// Fragment 中
class HomeFragment : Fragment() {
    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!

    override fun onCreateView(...): View {
        _binding = FragmentHomeBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.button.setOnClickListener {
            findNavController().navigate(...)
        }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null  // 避免内存泄漏
    }
}

// ─── 优化②: 懒加载 Fragment ───
class LazyFragment : Fragment() {
    private var isDataLoaded = false

    override fun onResume() {
        super.onResume()
        if (!isDataLoaded && isVisible) {
            loadData()
            isDataLoaded = true
        }
    }
}
```

---

## 🔄 第十部分：Navigation 进阶技巧

### 10.1 深层链接（Deep Link）

**作用**: 从外部 URL 直接打开应用的特定页面

#### 步骤一: 在 Navigation Graph 中定义

```xml
<fragment
    android:id="@+id/productDetailFragment"
    android:name="com.example.shopapp.ProductDetailFragment">
    
    <argument
        android:name="productId"
        app:argType="integer" />
    
    <!-- 定义深层链接 -->
    <deepLink
        android:id="@+id/deeplink_product"
        app:uri="myapp://product/{productId}" />
        
</fragment>
```

#### 步骤二: 在 AndroidManifest.xml 中声明

```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" />
    </intent-filter>
</activity>
```

#### 步骤三: 测试深层链接

```bash
adb shell am start -W -a android.intent.action.VIEW -d "myapp://product/123"
```

### 10.2 Navigation Drawer（侧滑菜单）

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var drawerLayout: DrawerLayout
    private lateinit var navController: NavController

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        drawerLayout = findViewById(R.id.drawer_layout)
        val navView = findViewById<NavigationView>(R.id.nav_view)

        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        navController = navHostFragment.navController

        // 连接 NavigationView 和 NavController
        navView.setupWithNavController(navController)

        // 设置 ActionBar 和 Drawer
        val appBarConfiguration = AppBarConfiguration(
            setOf(R.id.homeFragment, R.id.profileFragment),  // 顶层目的地
            drawerLayout
        )
        setupActionBarWithNavController(navController, appBarConfiguration)
    }
}
```

### 10.3 使用 ViewModel 共享数据

```kotlin
// SharedViewModel.kt
class CartViewModel : ViewModel() {
    private val _cartItems = MutableLiveData<List<Product>>()
    val cartItems: LiveData<List<Product>> = _cartItems

    fun addToCart(product: Product) {
        val currentList = _cartItems.value?.toMutableList() ?: mutableListOf()
        currentList.add(product)
        _cartItems.value = currentList
    }

    fun removeFromCart(product: Product) {
        val currentList = _cartItems.value?.toMutableList() ?: return
        currentList.remove(product)
        _cartItems.value = currentList
    }
}

// 在 Fragment 中使用（共享 Activity 级别的 ViewModel）
class ProductDetailFragment : Fragment() {
    private val cartViewModel: CartViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        binding.btnAddToCart.setOnClickListener {
            cartViewModel.addToCart(args.product)
            Toast.makeText(context, "已添加到购物车", Toast.LENGTH_SHORT).show()
        }
    }
}

class CartFragment : Fragment() {
    private val cartViewModel: CartViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        cartViewModel.cartItems.observe(viewLifecycleOwner) { items ->
            // 更新购物车 UI
            adapter.updateItems(items)
        }
    }
}
```

### 10.4 条件导航（根据状态跳转不同页面）

```kotlin
class SplashFragment : Fragment() {
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        lifecycleScope.launch {
            delay(2000)  // 模拟启动延迟
            
            val isLoggedIn = checkLoginStatus()
            
            if (isLoggedIn) {
                findNavController().navigate(R.id.action_splash_to_home)
            } else {
                findNavController().navigate(R.id.action_splash_to_login)
            }
        }
    }
}
```

---

## 🎯 课后思考题

1. **Navigation Component 相比传统的 FragmentTransaction 有哪些优势？**
    
2. **为什么 BottomNavigationView 的 Menu Item ID 必须与 Navigation Graph 中的 Fragment ID 一致？**
    
3. **SafeArgs 是如何实现类型安全的？它生成了哪些类和方法？**
    
4. **如何避免底部导航切换时重复创建 Fragment？Navigation Component 是如何处理的？**
    
5. **你在开发三Tab应用时遇到了什么问题？是如何解决的？（写入 README.md）**
    

---

## 📚 扩展阅读

### 官方文档

- [Navigation Component 概览](https://developer.android.com/guide/navigation)
- [使用 SafeArgs 传递数据](https://developer.android.com/guide/navigation/navigation-pass-data)
- [Navigation UI](https://developer.android.com/guide/navigation/navigation-ui)
- [深层链接](https://developer.android.com/guide/navigation/navigation-deep-link)

### 推荐文章

- [Single Activity: Why, When, and How](https://www.youtube.com/watch?v=2k8x8V77CrU)
- [Navigation Best Practices](https://developer.android.com/guide/navigation/navigation-best-practices)
- [Conditional Navigation](https://developer.android.com/guide/navigation/navigation-conditional)

---

## 🎓 下周预告

**第10周：网络通信基础**

- HTTP 协议基础
- Retrofit 库介绍与配置
- REST API 调用
- JSON 数据解析（Gson / Moshi）
- 实践：配置 Retrofit，调用公开 API 获取数据

**预习建议**:

- 复习 Kotlin 的协程基础
- 了解 REST API 的基本概念
- 了解 JSON 数据格式

**预习作业**: 尝试访问一个公开 API 查看返回的 JSON 数据：

```
https://jsonplaceholder.typicode.com/posts/1
```

在浏览器中打开，观察返回的数据结构。

**提前准备**: 在 `build.gradle` 中添加 Retrofit 依赖（下周会详细讲解）：

```gradle
dependencies {
    // Retrofit
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    
    // OkHttp (日志拦截器)
    implementation 'com.squareup.okhttp3:logging-interceptor:4.11.0'
}
```

---

**课程资料更新时间**: 2026年2月  
**讲师**: [教师姓名]  
**联系方式**: [邮箱]

_Navigation Component 是现代 Android 开发的必备技能！掌握 SafeArgs 和可视化导航图能大大提高开发效率。建议多练习，尝试构建复杂的导航结构。加油！🚀_