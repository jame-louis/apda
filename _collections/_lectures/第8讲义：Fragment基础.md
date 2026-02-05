---
title: 第8讲义：Fragment基础
date: 2026-02-05
permalink: /lectures/lecture08
show: true
---


**课程**: Android 移动应用开发入门  
**周次**: 第8周  
**主题**: Fragment基础  
**学时**: 3课时（理论2课时 + 实验1课时）

---

## 📋 本节课学习目标

完成本节课后，学生应该能够：

1. ✅ 理解 Fragment 的概念和使用场景
2. ✅ 掌握 Fragment 的生命周期及其与 Activity 的关系
3. ✅ 使用 FragmentManager 管理 Fragment 事务
4. ✅ 实现 Activity 与 Fragment 之间的双向通信
5. ✅ 掌握 Fragment 参数传递的正确方式
6. ✅ 创建可重用的 Fragment 组件
7. ✅ 使用 Fragment 重构现有应用

---

## 🧩 第一部分：Fragment 概念与使用场景

### 1.1 什么是 Fragment？

**Fragment（片段）** 是 Android 3.0 引入的组件，可以理解为**可重用的界面模块**。

**类比理解**:

```
Activity = 房子
Fragment = 房间

一个 Activity（房子）可以包含多个 Fragment（房间）
- 客厅 Fragment
- 卧室 Fragment  
- 厨房 Fragment

每个 Fragment 可以独立设计、重复使用
```

**官方定义**:

> Fragment 代表应用界面中可重复使用的一部分。Fragment 定义和管理自己的布局，具有自己的生命周期，并且可以处理自己的输入事件。

### 1.2 为什么需要 Fragment？

**问题场景一：平板适配**

```
手机（竖屏）:
┌──────────────┐
│ Activity A   │  ← 联系人列表
└──────────────┘
        ↓ 点击
┌──────────────┐
│ Activity B   │  ← 联系人详情
└──────────────┘

平板（横屏）:
┌───────────────────────────────┐
│  列表      │      详情        │  ← 一个屏幕同时显示
│ Fragment A │   Fragment B     │
└───────────────────────────────┘
```

**问题场景二：模块化开发**

```
❌ 只用 Activity:
每个界面都是独立 Activity → 难以复用 → 代码重复

✅ 使用 Fragment:
界面模块封装为 Fragment → 可在不同 Activity 中复用
```

**问题场景三：底部导航栏**

```
传统方案:
┌──────────────────┐
│  首页 Activity   │ ← 切换时需要销毁/创建 Activity
│  消息 Activity   │ ← 性能差，用户体验不好
│  我的 Activity   │
└──────────────────┘

Fragment 方案:
┌──────────────────┐
│  MainActivity    │
│  ┌────────────┐  │
│  │ 首页 Frag  │  │ ← 切换只替换 Fragment
│  │ 消息 Frag  │  │ ← 快速、流畅
│  │ 我的 Frag  │  │
│  └────────────┘  │
│  [首页][消息][我] │ ← BottomNavigationView
└──────────────────┘
```

### 1.3 Fragment 的优势

```
✅ 模块化：将复杂界面拆分为独立模块
✅ 可重用：同一个 Fragment 可用于不同场景
✅ 动态性：运行时动态添加/移除/替换
✅ 适配性：轻松适配手机和平板
✅ 性能：Fragment 切换比 Activity 切换更快
```

---

## 🔄 第二部分：Fragment 生命周期

### 2.1 Fragment 生命周期图

```
┌─────────────────────────────────────────┐
│          Fragment 生命周期              │
├─────────────────────────────────────────┤
│                                         │
│  onAttach()        ← Fragment 附加到 Activity
│     ↓                                   │
│  onCreate()        ← 创建 Fragment      │
│     ↓                                   │
│  onCreateView()    ← 创建布局           │
│     ↓                                   │
│  onViewCreated()   ← 布局创建完成       │
│     ↓                                   │
│  onStart()         ← Fragment 可见      │
│     ↓                                   │
│  onResume()        ← Fragment 可交互    │
│     ↓                                   │
│  [运行中]                               │
│     ↓                                   │
│  onPause()         ← 失去焦点           │
│     ↓                                   │
│  onStop()          ← 不可见             │
│     ↓                                   │
│  onDestroyView()   ← 销毁布局           │
│     ↓                                   │
│  onDestroy()       ← 销毁 Fragment      │
│     ↓                                   │
│  onDetach()        ← 从 Activity 分离   │
│                                         │
└─────────────────────────────────────────┘
```

### 2.2 关键生命周期方法详解

#### onAttach()

**作用**: Fragment 附加到 Activity 时调用

```kotlin
override fun onAttach(context: Context) {
    super.onAttach(context)
    Log.d("Fragment", "onAttach - Fragment 附加到 Activity")
    
    // 可以在这里获取 Activity 引用
    if (context is MainActivity) {
        // 初始化与 Activity 的通信
    }
}
```

#### onCreate()

**作用**: 创建 Fragment，初始化不依赖视图的数据

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    Log.d("Fragment", "onCreate - Fragment 创建")
    
    // ⚠️ 此时还没有视图，不能操作 UI
    // ✅ 可以接收参数、初始化数据
    val userId = arguments?.getInt("user_id", 0)
}
```

#### onCreateView()

**作用**: 创建并返回 Fragment 的布局

```kotlin
override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    Log.d("Fragment", "onCreateView - 创建布局")
    
    // 加载布局文件
    return inflater.inflate(R.layout.fragment_home, container, false)
}
```

**⚠️ 重要参数**:

- `inflater`: 用于加载布局
- `container`: Fragment 的父容器
- `attachToRoot`: **必须传 false**（系统会自动附加）

#### onViewCreated()

**作用**: 视图创建完成，可以安全地操作 UI

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    Log.d("Fragment", "onViewCreated - 视图已创建")
    
    // ✅ 现在可以 findViewById、设置监听器等
    val textView = view.findViewById<TextView>(R.id.textView)
    textView.text = "Hello Fragment"
    
    view.findViewById<Button>(R.id.button).setOnClickListener {
        // 处理点击
    }
}
```

#### onDestroyView()

**作用**: 视图即将销毁

```kotlin
override fun onDestroyView() {
    super.onDestroyView()
    Log.d("Fragment", "onDestroyView - 视图销毁")
    
    // ✅ 在这里释放视图相关资源
    // 例如: 取消订阅、移除监听器
}
```

### 2.3 Fragment 与 Activity 生命周期对比

```
Activity              Fragment
────────────────────────────────────
onCreate()            onAttach()
                      onCreate()
                      onCreateView()
                      onViewCreated()
onStart()             onStart()
onResume()            onResume()

[运行中]              [运行中]

onPause()             onPause()
onStop()              onStop()
                      onDestroyView()
onDestroy()           onDestroy()
                      onDetach()
```

**关键点**:

- Fragment 的生命周期方法总是**晚于**或**等于** Activity
- Fragment 比 Activity 多了视图相关的生命周期方法

---

## 🛠️ 第三部分：创建和使用 Fragment

### 3.1 创建 Fragment 类

```kotlin
package com.example.myapp

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.fragment.app.Fragment

class HomeFragment : Fragment() {

    // ① 创建视图
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_home, container, false)
    }

    // ② 视图创建完成后
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // 操作 UI
        view.findViewById<TextView>(R.id.titleText).text = "首页"
    }
}
```

### 3.2 创建 Fragment 布局

**fragment_home.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    android:gravity="center">

    <TextView
        android:id="@+id/titleText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="首页"
        android:textSize="24sp"
        android:textStyle="bold"
        android:textColor="#028090" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="这是首页 Fragment"
        android:textSize="16sp"
        android:layout_marginTop="16dp" />

</LinearLayout>
```

### 3.3 在 Activity 中添加 Fragment

#### 方法一：在 XML 中静态添加（不推荐）

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- 静态添加 Fragment -->
    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.myapp.HomeFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

**缺点**: 无法动态替换，灵活性差

#### 方法二：在代码中动态添加（推荐）

**activity_main.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- Fragment 容器 -->
    <FrameLayout
        android:id="@+id/fragmentContainer"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

**MainActivity.kt**:

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 第一次创建时才添加 Fragment（避免旋转屏幕后重复添加）
        if (savedInstanceState == null) {
            val fragment = HomeFragment()
            
            supportFragmentManager.beginTransaction()
                .add(R.id.fragmentContainer, fragment)
                .commit()
        }
    }
}
```

---

## 📦 第四部分：Fragment 事务管理

### 4.1 FragmentManager 和 FragmentTransaction

**FragmentManager**: Fragment 的管理器，负责添加、移除、替换 Fragment

```kotlin
// 获取 FragmentManager
val fragmentManager = supportFragmentManager  // Activity 中
val fragmentManager = childFragmentManager    // Fragment 中（管理子 Fragment）
```

**FragmentTransaction**: Fragment 事务，封装一系列 Fragment 操作

```kotlin
supportFragmentManager.beginTransaction()  // 开始事务
    .add(...)                              // 添加操作
    .replace(...)                          // 替换操作
    .remove(...)                           // 移除操作
    .commit()                              // 提交事务
```

### 4.2 常用事务操作

#### add() — 添加 Fragment

```kotlin
val fragment = HomeFragment()

supportFragmentManager.beginTransaction()
    .add(R.id.fragmentContainer, fragment, "home")  // tag 可选，用于查找
    .commit()
```

**效果**: 新 Fragment 添加到容器，**不会移除**已有的 Fragment

#### replace() — 替换 Fragment

```kotlin
val fragment = ProfileFragment()

supportFragmentManager.beginTransaction()
    .replace(R.id.fragmentContainer, fragment)
    .commit()
```

**效果**: 移除容器中所有 Fragment，添加新的 Fragment

#### remove() — 移除 Fragment

```kotlin
val fragment = supportFragmentManager.findFragmentById(R.id.fragmentContainer)

if (fragment != null) {
    supportFragmentManager.beginTransaction()
        .remove(fragment)
        .commit()
}
```

#### addToBackStack() — 添加到回退栈

```kotlin
supportFragmentManager.beginTransaction()
    .replace(R.id.fragmentContainer, ProfileFragment())
    .addToBackStack(null)  // 添加到回退栈
    .commit()
```

**效果**: 按返回键时，返回上一个 Fragment 而不是退出 Activity

```
用户操作流程:
┌──────────┐   replace +   ┌──────────┐   按返回键   ┌──────────┐
│ HomeFragment│  addToBackStack  │ProfileFrag│  ──────▶  │ HomeFragment│
└──────────┘   ──────▶    └──────────┘            └──────────┘
```

### 4.3 完整示例：底部导航栏切换

```kotlin
class MainActivity : AppCompatActivity() {

    private val homeFragment = HomeFragment()
    private val messageFragment = MessageFragment()
    private val profileFragment = ProfileFragment()
    
    private var currentFragment: Fragment = homeFragment

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 初始化：添加所有 Fragment 但只显示首页
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .add(R.id.fragmentContainer, homeFragment, "home")
                .add(R.id.fragmentContainer, messageFragment, "message")
                .add(R.id.fragmentContainer, profileFragment, "profile")
                .hide(messageFragment)
                .hide(profileFragment)
                .commit()
        }

        // 底部导航栏点击
        findViewById<BottomNavigationView>(R.id.bottomNav).setOnItemSelectedListener { item ->
            when (item.itemId) {
                R.id.nav_home -> showFragment(homeFragment)
                R.id.nav_message -> showFragment(messageFragment)
                R.id.nav_profile -> showFragment(profileFragment)
            }
            true
        }
    }

    private fun showFragment(fragment: Fragment) {
        if (fragment == currentFragment) return
        
        supportFragmentManager.beginTransaction()
            .hide(currentFragment)
            .show(fragment)
            .commit()
        
        currentFragment = fragment
    }
}
```

**优势**: 所有 Fragment 都保持在内存中，切换速度快，状态不丢失

### 4.4 事务提交方式

|方法|说明|适用场景|
|---|---|---|
|`commit()`|异步提交，在主线程空闲时执行|大部分情况|
|`commitNow()`|同步提交，立即执行|需要立即获取结果|
|`commitAllowingStateLoss()`|允许状态丢失的提交|Activity 可能已保存状态时|

```kotlin
// ❌ 错误：Activity onSaveInstanceState 后提交会崩溃
override fun onPause() {
    super.onPause()
    supportFragmentManager.beginTransaction()
        .add(...)
        .commit()  // IllegalStateException!
}

// ✅ 正确：使用 commitAllowingStateLoss
override fun onPause() {
    super.onPause()
    supportFragmentManager.beginTransaction()
        .add(...)
        .commitAllowingStateLoss()  // 允许状态丢失
}
```

---

## 💬 第五部分：Activity 与 Fragment 通信

### 5.1 Activity → Fragment：直接调用

```kotlin
// Activity 中
val fragment = supportFragmentManager.findFragmentById(R.id.fragmentContainer) as? HomeFragment
fragment?.updateData("新数据")

// HomeFragment 中
fun updateData(data: String) {
    view?.findViewById<TextView>(R.id.textView)?.text = data
}
```

**缺点**: Fragment 与 Activity 耦合，不够灵活

### 5.2 Fragment → Activity：接口回调（推荐）

#### 定义接口

```kotlin
// 在 Fragment 中定义接口
class HomeFragment : Fragment() {

    interface OnDataListener {
        fun onDataChanged(data: String)
    }

    private var listener: OnDataListener? = null

    override fun onAttach(context: Context) {
        super.onAttach(context)
        // 确保 Activity 实现了接口
        if (context is OnDataListener) {
            listener = context
        } else {
            throw RuntimeException("$context must implement OnDataListener")
        }
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        view.findViewById<Button>(R.id.button).setOnClickListener {
            // 通知 Activity
            listener?.onDataChanged("Fragment 的数据")
        }
    }

    override fun onDetach() {
        super.onDetach()
        listener = null  // 避免内存泄漏
    }
}
```

#### Activity 实现接口

```kotlin
class MainActivity : AppCompatActivity(), HomeFragment.OnDataListener {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        supportFragmentManager.beginTransaction()
            .add(R.id.fragmentContainer, HomeFragment())
            .commit()
    }

    override fun onDataChanged(data: String) {
        // 接收 Fragment 传来的数据
        Toast.makeText(this, "收到: $data", Toast.LENGTH_SHORT).show()
    }
}
```

### 5.3 Fragment ↔ Fragment：通过 Activity 中转

```kotlin
// Fragment A
class FragmentA : Fragment() {
    
    interface OnMessageListener {
        fun sendToFragmentB(message: String)
    }
    
    private var listener: OnMessageListener? = null
    
    override fun onAttach(context: Context) {
        super.onAttach(context)
        listener = context as? OnMessageListener
    }
    
    private fun sendMessage() {
        listener?.sendToFragmentB("来自 Fragment A 的消息")
    }
}

// Fragment B
class FragmentB : Fragment() {
    
    fun receiveMessage(message: String) {
        view?.findViewById<TextView>(R.id.messageText)?.text = message
    }
}

// Activity 中转
class MainActivity : AppCompatActivity(), FragmentA.OnMessageListener {
    
    override fun sendToFragmentB(message: String) {
        val fragmentB = supportFragmentManager
            .findFragmentByTag("fragmentB") as? FragmentB
        fragmentB?.receiveMessage(message)
    }
}
```

### 5.4 使用 ViewModel 共享数据（推荐，现代方案）

```kotlin
// 共享的 ViewModel
class SharedViewModel : ViewModel() {
    private val _selectedItem = MutableLiveData<String>()
    val selectedItem: LiveData<String> = _selectedItem
    
    fun selectItem(item: String) {
        _selectedItem.value = item
    }
}

// Fragment A - 发送数据
class FragmentA : Fragment() {
    
    private val viewModel: SharedViewModel by activityViewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        view.findViewById<Button>(R.id.button).setOnClickListener {
            viewModel.selectItem("选中的项目")
        }
    }
}

// Fragment B - 接收数据
class FragmentB : Fragment() {
    
    private val viewModel: SharedViewModel by activityViewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        viewModel.selectedItem.observe(viewLifecycleOwner) { item ->
            // 数据变化时自动更新
            view.findViewById<TextView>(R.id.textView).text = item
        }
    }
}
```

**优势**:

- 解耦，Fragment 之间无需互相引用
- 自动处理生命周期
- 代码简洁

---

## 📮 第六部分：Fragment 参数传递

### 6.1 为什么不能用构造函数传参？

```kotlin
// ❌ 错误做法
class UserFragment(private val userId: Int) : Fragment() {
    // 系统重建 Fragment 时会调用无参构造函数，导致崩溃！
}

// ✅ 正确做法：使用 arguments
class UserFragment : Fragment() {
    // 系统可以正确重建
}
```

**原因**: 系统在配置变化（如旋转屏幕）时会销毁并重建 Fragment，此时会调用**无参构造函数**

### 6.2 使用 Bundle 传递参数（标准做法）

```kotlin
// UserFragment.kt
class UserFragment : Fragment() {

    companion object {
        private const val ARG_USER_ID = "user_id"
        private const val ARG_USER_NAME = "user_name"
        
        // 静态工厂方法
        fun newInstance(userId: Int, userName: String): UserFragment {
            val fragment = UserFragment()
            val args = Bundle()
            args.putInt(ARG_USER_ID, userId)
            args.putString(ARG_USER_NAME, userName)
            fragment.arguments = args
            return fragment
        }
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // 读取参数
        val userId = arguments?.getInt(ARG_USER_ID, 0) ?: 0
        val userName = arguments?.getString(ARG_USER_NAME, "") ?: ""
        
        view.findViewById<TextView>(R.id.userIdText).text = "ID: $userId"
        view.findViewById<TextView>(R.id.userNameText).text = "姓名: $userName"
    }
}

// Activity 中使用
val fragment = UserFragment.newInstance(123, "张三")
supportFragmentManager.beginTransaction()
    .add(R.id.fragmentContainer, fragment)
    .commit()
```

### 6.3 传递复杂对象

#### 方式一：Parcelable（推荐）

```kotlin
@Parcelize
data class User(
    val id: Int,
    val name: String,
    val email: String
) : Parcelable

// Fragment
companion object {
    private const val ARG_USER = "user"
    
    fun newInstance(user: User): UserFragment {
        val fragment = UserFragment()
        val args = Bundle()
        args.putParcelable(ARG_USER, user)
        fragment.arguments = args
        return fragment
    }
}

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    
    val user = arguments?.getParcelable<User>(ARG_USER)
    // 使用 user 对象
}
```

#### 方式二: Serializable（不推荐，性能差）

```kotlin
data class User(
    val id: Int,
    val name: String
) : Serializable

// 传递
args.putSerializable(ARG_USER, user)

// 接收
val user = arguments?.getSerializable(ARG_USER) as? User
```

---

## 🧪 第七部分：实验

### 7.1 实验一：创建可重用的 DetailFragment

**目标**: 创建一个通用的详情展示 Fragment，可复用于显示不同类型的数据。

#### DetailFragment.kt

```kotlin
package com.example.fragmentdemo

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Button
import android.widget.TextView
import androidx.fragment.app.Fragment

class DetailFragment : Fragment() {

    companion object {
        private const val ARG_TITLE = "title"
        private const val ARG_CONTENT = "content"
        
        fun newInstance(title: String, content: String): DetailFragment {
            val fragment = DetailFragment()
            val args = Bundle()
            args.putString(ARG_TITLE, title)
            args.putString(ARG_CONTENT, content)
            fragment.arguments = args
            return fragment
        }
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_detail, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        val title = arguments?.getString(ARG_TITLE, "")
        val content = arguments?.getString(ARG_CONTENT, "")
        
        view.findViewById<TextView>(R.id.detailTitle).text = title
        view.findViewById<TextView>(R.id.detailContent).text = content
        
        view.findViewById<Button>(R.id.btnClose).setOnClickListener {
            // 通知 Activity 关闭此 Fragment
            (activity as? OnCloseListener)?.onCloseDetail()
        }
    }

    interface OnCloseListener {
        fun onCloseDetail()
    }
}
```

#### fragment_detail.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp"
    android:background="@android:color/white">

    <TextView
        android:id="@+id/detailTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="标题"
        android:textSize="24sp"
        android:textStyle="bold"
        android:textColor="#028090"
        android:layout_marginBottom="16dp" />

    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:background="#E0E0E0"
        android:layout_marginBottom="16dp" />

    <TextView
        android:id="@+id/detailContent"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:text="内容"
        android:textSize="16sp"
        android:textColor="#424242"
        android:lineSpacingExtra="4dp" />

    <Button
        android:id="@+id/btnClose"
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:text="关闭"
        android:textSize="16sp" />

</LinearLayout>
```

#### 在 Activity 中使用

```kotlin
class MainActivity : AppCompatActivity(), DetailFragment.OnCloseListener {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        findViewById<Button>(R.id.btnShowNews).setOnClickListener {
            showDetail("新闻标题", "这是新闻的详细内容...")
        }

        findViewById<Button>(R.id.btnShowArticle).setOnClickListener {
            showDetail("文章标题", "这是文章的详细内容...")
        }
    }

    private fun showDetail(title: String, content: String) {
        val fragment = DetailFragment.newInstance(title, content)
        
        supportFragmentManager.beginTransaction()
            .replace(R.id.fragmentContainer, fragment)
            .addToBackStack(null)
            .commit()
    }

    override fun onCloseDetail() {
        supportFragmentManager.popBackStack()
    }
}
```

---

### 7.2 实验二：实现 Fragment 切换（底部导航）

**目标**: 创建一个带底部导航栏的应用，点击切换不同 Fragment。

#### 三个 Fragment

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
        
        view.findViewById<TextView>(R.id.titleText).text = "🏠 首页"
        view.findViewById<TextView>(R.id.contentText).text = "欢迎来到首页"
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

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        view.findViewById<TextView>(R.id.titleText).text = "💬 消息"
        view.findViewById<TextView>(R.id.contentText).text = "你有 3 条新消息"
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

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        view.findViewById<TextView>(R.id.titleText).text = "👤 我的"
        view.findViewById<TextView>(R.id.contentText).text = "个人中心"
    }
}
```

#### 通用 Fragment 布局

**fragment_common.xml** (三个 Fragment 共用):

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="24dp">

    <TextView
        android:id="@+id/titleText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="标题"
        android:textSize="32sp"
        android:textStyle="bold"
        android:textColor="#028090" />

    <TextView
        android:id="@+id/contentText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="内容"
        android:textSize="18sp"
        android:textColor="#757575"
        android:layout_marginTop="16dp" />

</LinearLayout>
```

#### MainActivity

```kotlin
class MainActivity : AppCompatActivity() {

    private val homeFragment = HomeFragment()
    private val messageFragment = MessageFragment()
    private val profileFragment = ProfileFragment()
    
    private var currentFragment: Fragment = homeFragment

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 初始化：添加所有 Fragment，隐藏除首页外的其他
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .add(R.id.fragmentContainer, homeFragment, "home")
                .add(R.id.fragmentContainer, messageFragment, "message")
                .add(R.id.fragmentContainer, profileFragment, "profile")
                .hide(messageFragment)
                .hide(profileFragment)
                .commit()
        }

        // 底部导航栏
        setupBottomNavigation()
    }

    private fun setupBottomNavigation() {
        val bottomNav = findViewById<LinearLayout>(R.id.bottomNav)
        
        findViewById<LinearLayout>(R.id.navHome).setOnClickListener {
            switchFragment(homeFragment)
        }
        
        findViewById<LinearLayout>(R.id.navMessage).setOnClickListener {
            switchFragment(messageFragment)
        }
        
        findViewById<LinearLayout>(R.id.navProfile).setOnClickListener {
            switchFragment(profileFragment)
        }
    }

    private fun switchFragment(targetFragment: Fragment) {
        if (targetFragment == currentFragment) return
        
        supportFragmentManager.beginTransaction()
            .hide(currentFragment)
            .show(targetFragment)
            .commit()
        
        currentFragment = targetFragment
    }
}
```

#### activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- Fragment 容器 -->
    <FrameLayout
        android:id="@+id/fragmentContainer"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <!-- 底部导航栏 -->
    <LinearLayout
        android:id="@+id/bottomNav"
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:orientation="horizontal"
        android:background="#FFFFFF"
        android:elevation="8dp">

        <!-- 首页 -->
        <LinearLayout
            android:id="@+id/navHome"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:orientation="vertical"
            android:gravity="center"
            android:background="?attr/selectableItemBackground">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="🏠"
                android:textSize="24sp" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="首页"
                android:textSize="12sp"
                android:textColor="#757575" />

        </LinearLayout>

        <!-- 消息 -->
        <LinearLayout
            android:id="@+id/navMessage"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:orientation="vertical"
            android:gravity="center"
            android:background="?attr/selectableItemBackground">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="💬"
                android:textSize="24sp" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="消息"
                android:textSize="12sp"
                android:textColor="#757575" />

        </LinearLayout>

        <!-- 我的 -->
        <LinearLayout
            android:id="@+id/navProfile"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:orientation="vertical"
            android:gravity="center"
            android:background="?attr/selectableItemBackground">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="👤"
                android:textSize="24sp" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="我的"
                android:textSize="12sp"
                android:textColor="#757575" />

        </LinearLayout>

    </LinearLayout>

</LinearLayout>
```

---

## 📝 第八部分：课后作业 — 使用 Fragment 重构应用

### 8.1 作业概述

选择之前完成的一个应用（记事本、新闻列表、联系人等），使用 Fragment 进行重构。

#### 功能要求

**必需功能**:

- ✅ 至少使用 2 个 Fragment
- ✅ 实现 Fragment 之间的切换
- ✅ 正确传递参数
- ✅ 实现 Fragment 与 Activity 的通信
- ✅ 使用回退栈（按返回键返回上一个 Fragment）

**推荐重构方案**:

**方案一：记事本应用重构**

```
MainActivity
├── NoteListFragment      (笔记列表)
└── NoteDetailFragment    (笔记详情)

流程:
启动 → 显示列表 → 点击笔记 → 替换为详情 Fragment → 返回键回到列表
```

**方案二：新闻应用重构**

```
MainActivity
├── NewsListFragment      (新闻列表)
├── NewsDetailFragment    (新闻详情)
└── CategoryFragment      (分类筛选)

底部导航:
[新闻列表] [分类筛选] [收藏夹]
```

**方案三：联系人应用重构**

```
MainActivity (平板适配)
├── ContactListFragment   (联系人列表)
└── ContactDetailFragment (联系人详情)

手机: 列表和详情占满屏，切换显示
平板: 列表和详情左右分栏，同时显示
```

### 8.2 参考实现：记事本重构

#### NoteListFragment.kt

```kotlin
package com.example.notepad

import android.content.Context
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.floatingactionbutton.FloatingActionButton

class NoteListFragment : Fragment() {

    interface OnNoteSelectedListener {
        fun onNoteSelected(noteId: Int, title: String, content: String)
        fun onCreateNewNote()
    }

    private var listener: OnNoteSelectedListener? = null
    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: NoteAdapter

    override fun onAttach(context: Context) {
        super.onAttach(context)
        if (context is OnNoteSelectedListener) {
            listener = context
        }
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_note_list, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        recyclerView = view.findViewById(R.id.recyclerView)
        recyclerView.layoutManager = LinearLayoutManager(context)

        // 加载笔记数据
        val notes = loadNotes()
        adapter = NoteAdapter(notes) { note ->
            listener?.onNoteSelected(note.id, note.title, note.content)
        }
        recyclerView.adapter = adapter

        // 新建按钮
        view.findViewById<FloatingActionButton>(R.id.fabAdd).setOnClickListener {
            listener?.onCreateNewNote()
        }
    }

    private fun loadNotes(): List<Note> {
        // 从文件或数据库加载
        return NoteManager.listNotes(requireContext().filesDir).map { fileName ->
            val note = NoteManager.readNote(requireContext().filesDir, fileName)
            Note(
                id = NoteManager.getTimestamp(fileName).toInt(),
                title = note?.first ?: "",
                content = note?.second ?: ""
            )
        }
    }

    fun refreshList() {
        val notes = loadNotes()
        adapter.updateData(notes)
    }

    override fun onDetach() {
        super.onDetach()
        listener = null
    }
}
```

#### NoteDetailFragment.kt

```kotlin
package com.example.notepad

import android.content.Context
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.fragment.app.Fragment

class NoteDetailFragment : Fragment() {

    companion object {
        private const val ARG_NOTE_ID = "note_id"
        private const val ARG_NOTE_TITLE = "note_title"
        private const val ARG_NOTE_CONTENT = "note_content"
        private const val ARG_IS_NEW = "is_new"

        fun newInstance(noteId: Int, title: String, content: String): NoteDetailFragment {
            val fragment = NoteDetailFragment()
            val args = Bundle()
            args.putInt(ARG_NOTE_ID, noteId)
            args.putString(ARG_NOTE_TITLE, title)
            args.putString(ARG_NOTE_CONTENT, content)
            args.putBoolean(ARG_IS_NEW, false)
            fragment.arguments = args
            return fragment
        }

        fun newInstanceForCreate(): NoteDetailFragment {
            val fragment = NoteDetailFragment()
            val args = Bundle()
            args.putBoolean(ARG_IS_NEW, true)
            fragment.arguments = args
            return fragment
        }
    }

    interface OnNoteSavedListener {
        fun onNoteSaved()
    }

    private var listener: OnNoteSavedListener? = null
    private lateinit var titleEdit: EditText
    private lateinit var contentEdit: EditText

    override fun onAttach(context: Context) {
        super.onAttach(context)
        if (context is OnNoteSavedListener) {
            listener = context
        }
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_note_detail, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        titleEdit = view.findViewById(R.id.titleEdit)
        contentEdit = view.findViewById(R.id.contentEdit)

        // 如果是编辑模式，填充数据
        val isNew = arguments?.getBoolean(ARG_IS_NEW, true) ?: true
        if (!isNew) {
            titleEdit.setText(arguments?.getString(ARG_NOTE_TITLE, ""))
            contentEdit.setText(arguments?.getString(ARG_NOTE_CONTENT, ""))
        }

        // 保存按钮
        view.findViewById<Button>(R.id.btnSave).setOnClickListener {
            saveNote()
        }
    }

    private fun saveNote() {
        val title = titleEdit.text.toString().trim()
        val content = contentEdit.text.toString()

        if (title.isEmpty()) {
            Toast.makeText(context, "标题不能为空", Toast.LENGTH_SHORT).show()
            return
        }

        val isNew = arguments?.getBoolean(ARG_IS_NEW, true) ?: true
        if (isNew) {
            NoteManager.createNote(requireContext().filesDir, title, content)
        } else {
            val noteId = arguments?.getInt(ARG_NOTE_ID, 0) ?: 0
            val fileName = "note_$noteId.txt"
            NoteManager.updateNote(requireContext().filesDir, fileName, title, content)
        }

        Toast.makeText(context, "保存成功", Toast.LENGTH_SHORT).show()
        listener?.onNoteSaved()
        
        // 返回列表
        parentFragmentManager.popBackStack()
    }

    override fun onDetach() {
        super.onDetach()
        listener = null
    }
}
```

#### MainActivity.kt

```kotlin
package com.example.notepad

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity(), 
    NoteListFragment.OnNoteSelectedListener,
    NoteDetailFragment.OnNoteSavedListener {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .add(R.id.fragmentContainer, NoteListFragment())
                .commit()
        }
    }

    override fun onNoteSelected(noteId: Int, title: String, content: String) {
        val fragment = NoteDetailFragment.newInstance(noteId, title, content)
        
        supportFragmentManager.beginTransaction()
            .replace(R.id.fragmentContainer, fragment)
            .addToBackStack(null)
            .commit()
    }

    override fun onCreateNewNote() {
        val fragment = NoteDetailFragment.newInstanceForCreate()
        
        supportFragmentManager.beginTransaction()
            .replace(R.id.fragmentContainer, fragment)
            .addToBackStack(null)
            .commit()
    }

    override fun onNoteSaved() {
        // 刷新列表
        val listFragment = supportFragmentManager
            .findFragmentById(R.id.fragmentContainer) as? NoteListFragment
        listFragment?.refreshList()
    }
}
```

### 8.3 评分标准

|项目|分值|评分细则|
|---|---|---|
|Fragment 使用|30%|至少使用 2 个 Fragment，职责划分清晰|
|参数传递|20%|使用 Bundle 正确传递参数，遵循最佳实践|
|通信机制|20%|正确实现 Fragment 与 Activity 的通信|
|事务管理|15%|正确使用事务，添加到回退栈|
|代码质量|15%|代码结构清晰，无内存泄漏|

#### 加分项（可选，每项+5分，上限+15分）

- ✨ 实现平板适配（左右分栏显示）
- ✨ 使用 ViewModel 共享数据
- ✨ Fragment 切换带动画效果
- ✨ 处理配置变化（旋转屏幕）数据不丢失
- ✨ 代码注释详细，README 完整

#### 提交要求

1. **源代码** — 完整 Android 项目（ZIP 压缩包）
2. **运行截图** — 至少5张（列表页、详情页、切换效果、返回效果、重构前后对比）
3. **说明文档** — README.md（重构思路、Fragment 设计、遇到的问题和解决方法）

**文件命名**: `姓名_学号_Week8_Fragment重构.zip`  
**截止时间**: 第9周周一上课前

---

## 💡 第九部分：最佳实践与常见坑

### 9.1 Fragment 常见坑

```kotlin
// ─── 坑①: 使用构造函数传参 ───
// ❌ 错误
class MyFragment(private val userId: Int) : Fragment() {
    // 系统重建时会调用无参构造函数，导致崩溃
}

// ✅ 正确：使用 newInstance + Bundle
class MyFragment : Fragment() {
    companion object {
        fun newInstance(userId: Int): MyFragment {
            val fragment = MyFragment()
            val args = Bundle()
            args.putInt("user_id", userId)
            fragment.arguments = args
            return fragment
        }
    }
}

// ─── 坑②: 在 onCreateView 中操作 UI ───
// ❌ 错误
override fun onCreateView(...): View? {
    val view = inflater.inflate(R.layout.fragment_home, container, false)
    view.findViewById<TextView>(R.id.textView).text = "Hello"  // 可能崩溃
    return view
}

// ✅ 正确：在 onViewCreated 中操作 UI
override fun onCreateView(...): View? {
    return inflater.inflate(R.layout.fragment_home, container, false)
}

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    view.findViewById<TextView>(R.id.textView).text = "Hello"  // 安全
}

// ─── 坑③: 忘记检查 null ───
// ❌ 错误
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    
    val data = loadData()
    // Fragment 可能已经 detach，view 变为 null
    view?.findViewById<TextView>(R.id.textView)?.text = data  // 仍可能崩溃
}

// ✅ 正确：使用 viewLifecycleOwner
class MyFragment : Fragment() {
    
    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!
    
    override fun onCreateView(...): View {
        _binding = FragmentHomeBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null  // 避免内存泄漏
    }
}

// ─── 坑④: 直接持有 Activity 引用 ───
// ❌ 错误
class MyFragment : Fragment() {
    private lateinit var mainActivity: MainActivity  // 内存泄漏！
    
    override fun onAttach(context: Context) {
        super.onAttach(context)
        mainActivity = context as MainActivity
    }
}

// ✅ 正确：使用接口回调
class MyFragment : Fragment() {
    
    private var listener: OnDataListener? = null
    
    override fun onAttach(context: Context) {
        super.onAttach(context)
        listener = context as? OnDataListener
    }
    
    override fun onDetach() {
        super.onDetach()
        listener = null  // 及时释放
    }
}

// ─── 坑⑤: 在错误的时机访问 View ───
// ❌ 错误
class MyFragment : Fragment() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        view?.findViewById<TextView>(R.id.textView)  // view = null!
    }
}

// ✅ 正确：在 onViewCreated 之后访问
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    view.findViewById<TextView>(R.id.textView).text = "OK"
}
```

### 9.2 Fragment 性能优化

```kotlin
// ─── 优化①: 懒加载（避免一次性加载所有 Fragment） ───
class LazyFragment : Fragment() {
    
    private var isDataLoaded = false
    
    override fun onResume() {
        super.onResume()
        if (!isDataLoaded && isVisible) {
            loadData()
            isDataLoaded = true
        }
    }
    
    private fun loadData() {
        // 加载数据
    }
}

// ─── 优化②: 使用 ViewPager2 + Fragment 缓存 ───
viewPager2.offscreenPageLimit = 1  // 预加载相邻 1 页

// ─── 优化③: 避免重复创建 Fragment ───
class MainActivity : AppCompatActivity() {
    
    // ❌ 每次都创建新实例
    fun showFragment() {
        val fragment = HomeFragment()  // 重复创建
        supportFragmentManager.beginTransaction()
            .replace(R.id.container, fragment)
            .commit()
    }
    
    // ✅ 复用实例
    private val homeFragment = HomeFragment()
    
    fun showFragment() {
        supportFragmentManager.beginTransaction()
            .replace(R.id.container, homeFragment)
            .commit()
    }
}
```

### 9.3 Fragment 内存泄漏预防

```kotlin
class MyFragment : Fragment() {
    
    // ❌ 静态引用会导致内存泄漏
    companion object {
        private var callback: (() -> Unit)? = null  // 危险！
    }
    
    // ❌ Handler 内存泄漏
    private val handler = Handler(Looper.getMainLooper())
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        handler.postDelayed({
            // Fragment 可能已销毁，但 Handler 还在执行
        }, 10000)
    }
    
    // ✅ 正确：在 onDestroyView 中清理
    override fun onDestroyView() {
        super.onDestroyView()
        handler.removeCallbacksAndMessages(null)  // 移除所有回调
    }
}
```

### 9.4 Fragment 与 Activity 解耦技巧

```kotlin
// ❌ Fragment 直接调用 Activity 方法（耦合）
class MyFragment : Fragment() {
    fun updateTitle() {
        (activity as? MainActivity)?.setTitle("新标题")  // 耦合了 MainActivity
    }
}

// ✅ 使用接口解耦
class MyFragment : Fragment() {
    
    interface TitleProvider {
        fun updateTitle(title: String)
    }
    
    private var titleProvider: TitleProvider? = null
    
    override fun onAttach(context: Context) {
        super.onAttach(context)
        titleProvider = context as? TitleProvider
    }
    
    fun updateTitle() {
        titleProvider?.updateTitle("新标题")  // 任何实现接口的 Activity 都可以
    }
}

// ✅ 或者使用 ViewModel（更现代）
class SharedViewModel : ViewModel() {
    private val _title = MutableLiveData<String>()
    val title: LiveData<String> = _title
    
    fun setTitle(title: String) {
        _title.value = title
    }
}

// Fragment 中
class MyFragment : Fragment() {
    private val viewModel: SharedViewModel by activityViewModels()
    
    fun updateTitle() {
        viewModel.setTitle("新标题")
    }
}

// Activity 中
class MainActivity : AppCompatActivity() {
    private val viewModel: SharedViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        viewModel.title.observe(this) { title ->
            supportActionBar?.title = title
        }
    }
}
```

---

## 🔄 第十部分：Fragment 进阶技巧

### 10.1 Fragment 动画

```kotlin
supportFragmentManager.beginTransaction()
    .setCustomAnimations(
        R.anim.slide_in_right,  // 进入动画
        R.anim.slide_out_left,  // 退出动画
        R.anim.slide_in_left,   // 弹出栈时进入动画
        R.anim.slide_out_right  // 弹出栈时退出动画
    )
    .replace(R.id.container, fragment)
    .addToBackStack(null)
    .commit()
```

**slide_in_right.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:fromXDelta="100%"
        android:toXDelta="0%"
        android:duration="300" />
</set>
```

### 10.2 Fragment 与 ViewPager2 结合

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        val viewPager = findViewById<ViewPager2>(R.id.viewPager)
        val adapter = ViewPagerAdapter(this)
        viewPager.adapter = adapter
    }
}

class ViewPagerAdapter(activity: AppCompatActivity) : FragmentStateAdapter(activity) {
    
    override fun getItemCount(): Int = 3
    
    override fun createFragment(position: Int): Fragment {
        return when (position) {
            0 -> HomeFragment()
            1 -> MessageFragment()
            2 -> ProfileFragment()
            else -> HomeFragment()
        }
    }
}
```

### 10.3 DialogFragment

```kotlin
class ConfirmDialogFragment : DialogFragment() {
    
    companion object {
        private const val ARG_MESSAGE = "message"
        
        fun newInstance(message: String): ConfirmDialogFragment {
            val fragment = ConfirmDialogFragment()
            val args = Bundle()
            args.putString(ARG_MESSAGE, message)
            fragment.arguments = args
            return fragment
        }
    }
    
    interface OnConfirmListener {
        fun onConfirm()
    }
    
    override fun onCreateDialog(savedInstanceState: Bundle?): Dialog {
        val message = arguments?.getString(ARG_MESSAGE, "") ?: ""
        
        return AlertDialog.Builder(requireContext())
            .setTitle("确认")
            .setMessage(message)
            .setPositiveButton("确定") { dialog, _ ->
                (activity as? OnConfirmListener)?.onConfirm()
                dialog.dismiss()
            }
            .setNegativeButton("取消") { dialog, _ ->
                dialog.dismiss()
            }
            .create()
    }
}

// 使用
val dialog = ConfirmDialogFragment.newInstance("确定要删除吗？")
dialog.show(supportFragmentManager, "confirm")
```

### 10.4 平板适配（双面板布局）

```kotlin
class MainActivity : AppCompatActivity() {
    
    private var isTwoPane = false
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 检查是否是双面板布局
        isTwoPane = findViewById<View>(R.id.detailContainer) != null
        
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .add(R.id.listContainer, NoteListFragment())
                .commit()
        }
    }
    
    fun showDetail(noteId: Int, title: String, content: String) {
        val fragment = NoteDetailFragment.newInstance(noteId, title, content)
        
        if (isTwoPane) {
            // 平板：在右侧面板显示
            supportFragmentManager.beginTransaction()
                .replace(R.id.detailContainer, fragment)
                .commit()
        } else {
            // 手机：全屏显示
            supportFragmentManager.beginTransaction()
                .replace(R.id.listContainer, fragment)
                .addToBackStack(null)
                .commit()
        }
    }
}
```

**layout/activity_main.xml** (手机):

```xml
<FrameLayout
    android:id="@+id/listContainer"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

**layout-sw600dp/activity_main.xml** (平板):

```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">
    
    <FrameLayout
        android:id="@+id/listContainer"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />
    
    <FrameLayout
        android:id="@+id/detailContainer"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="2" />
        
</LinearLayout>
```

---

## 🎯 课后思考题

1. **Fragment 和 Activity 有什么区别？什么时候应该使用 Fragment？**
    
2. **为什么 Fragment 不能使用构造函数传参？系统重建 Fragment 时会发生什么？**
    
3. **Fragment 的生命周期方法中，哪些可以安全地操作 UI？为什么？**
    
4. **如何实现 Fragment 之间的通信？请列举至少两种方式并说明优缺点。**
    
5. **你在使用 Fragment 重构应用时遇到了什么问题？是如何解决的？（写入 README.md）**
    

---

## 📚 扩展阅读

### 官方文档

- [Fragment 开发指南](https://developer.android.com/guide/fragments)
- [Fragment 生命周期](https://developer.android.com/guide/fragments/lifecycle)
- [Fragment 事务](https://developer.android.com/guide/fragments/transactions)
- [Fragment 通信](https://developer.android.com/guide/fragments/communicate)

### 推荐文章

- [Single Activity Architecture](https://developer.android.com/guide/navigation/navigation-principles)
- [Fragment Best Practices](https://developer.android.com/guide/fragments/best-practices)
- [Avoid Passing Context to ViewModels](https://developer.android.com/topic/architecture/recommendations#avoid-passing-context-to-viewmodels)

---

## 🎓 下周预告

**第9周：导航组件（Navigation Component）**

- Navigation Graph 配置
- Fragment 导航（跳转、返回、参数传递）
- SafeArgs 插件使用
- 底部导航栏（BottomNavigationView）集成
- 实践：配置 Navigation Graph，实现底部 Tab 导航

**预习建议**:

- 复习本周的 Fragment 知识（Navigation Component 基于 Fragment）
- 思考：如何管理复杂的 Fragment 跳转逻辑？
- 了解"声明式导航"的概念

**预习作业**: 尝试在 `build.gradle` 中添加 Navigation Component 依赖：

```gradle
dependencies {
    def nav_version = "2.7.6"
    implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
    implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
}
```

---

**课程资料更新时间**: 2026年2月  
**讲师**: [教师姓名]  
**联系方式**: [邮箱]

_Fragment 是 Android 开发的核心组件！理解生命周期、事务管理和通信机制对后续学习至关重要。建议多练习，尝试不同的重构方案。加油！🚀_