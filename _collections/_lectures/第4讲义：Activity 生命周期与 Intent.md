---
title: 第4讲义：Activity 生命周期与 Intent
show: true
date: 2026-02-03
permalink: /lectures/lecture04
---


**课程**: Android 移动应用开发入门  
**周次**: 第4周  
**主题**: Activity 生命周期与 Intent  
**学时**: 3课时（理论2课时 + 实验1课时）

---

## 📋 本节课学习目标

完成本节课后，学生应该能够：

1. ✅ 深入理解 Activity 生命周期的概念和重要性
2. ✅ 掌握七个主要的生命周期回调方法及其应用场景
3. ✅ 学会监控和调试 Activity 生命周期状态变化
4. ✅ 理解 Intent 的概念及其在 Android 中的作用
5. ✅ 使用显式 Intent 实现 Activity 间的导航跳转
6. ✅ 掌握使用 putExtra 和 getExtra 进行数据传递
7. ✅ 能够创建完整的多页面导航应用

---

## 🎬 第一部分：Activity 概述

### 1.1 什么是 Activity？

**Activity** 是 Android 四大组件之一（Activity、Service、BroadcastReceiver、ContentProvider），也是开发者最常接触的组件。

**定义**:

- Activity 代表应用中的一个**单一屏幕界面**
- 是用户与应用交互的**入口点**
- 包含用户界面（UI）和业务逻辑

**形象比喻**:

```
Activity 就像：
📖 一本书的每一页
🌐 网站的每个网页  
📱 手机应用的每个屏幕
🎬 电影的每个场景
```

**实际例子** - 微信应用结构：

```
微信应用
├── SplashActivity        (启动页)
├── LoginActivity         (登录界面)
├── MainActivity          (主界面 - 聊天列表)
├── ChatActivity          (聊天界面)
├── ContactsActivity      (通讯录)
├── DiscoverActivity      (发现页)
├── ProfileActivity       (个人信息)
└── SettingsActivity      (设置)
```

### 1.2 Activity 的特点

#### 特点1: 单一职责原则

每个 Activity 应该专注于一个特定任务：

```kotlin
// ✅ 好的设计 - 每个 Activity 职责明确
LoginActivity      → 处理用户登录
RegisterActivity   → 处理用户注册  
ProfileActivity    → 显示和编辑用户资料
SettingsActivity   → 管理应用设置

// ❌ 不好的设计 - 一个 Activity 做太多事
MainActivity       → 登录、注册、设置、聊天... (职责混乱)
```

#### 特点2: 独立性

- 每个 Activity 相对独立运行
- 可以被其他应用启动（如果设置了 `android:exported="true"`）
- 通过 Intent 进行组件间通信

#### 特点3: 有生命周期

- Activity 有明确的创建、运行、暂停、停止、销毁过程
- 系统在不同阶段回调特定方法
- 开发者需要正确管理资源和状态

### 1.3 创建 Activity 的步骤

#### 步骤1: 创建 Activity 类

```kotlin
package com.example.myapp

import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    
    private lateinit var textView: TextView
    private lateinit var button: Button
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 初始化 View
        textView = findViewById(R.id.textView)
        button = findViewById(R.id.button)
        
        // 设置监听器
        button.setOnClickListener {
            textView.text = "按钮被点击了！"
        }
    }
}
```

**关键点**:

- 继承自 `AppCompatActivity`（推荐）或 `Activity`
- 重写 `onCreate()` 方法
- 调用 `setContentView()` 设置布局

#### 步骤2: 创建布局文件

创建 `res/layout/activity_main.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    android:gravity="center">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="欢迎来到 MainActivity"
        android:textSize="24sp"
        android:textStyle="bold"
        android:textColor="#028090" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="点击我"
        android:layout_marginTop="24dp" />

</LinearLayout>
```

#### 步骤3: 在 AndroidManifest.xml 中注册

**⚠️ 这一步非常重要！所有 Activity 必须注册才能使用。**

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.MyApp">
        
        <!-- 主 Activity - 应用入口 -->
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <!-- 其他 Activity -->
        <activity
            android:name=".SecondActivity"
            android:exported="false"
            android:label="第二页" />
            
    </application>

</manifest>
```

**重要属性说明**:

|属性|说明|可选值|
|---|---|---|
|`android:name`|Activity 的类名|`.MainActivity`|
|`android:exported`|是否允许其他应用启动|`true` / `false`|
|`android:label`|Activity 显示的标题|字符串|
|`android:theme`|使用的主题|`@style/Theme.xxx`|
|`android:screenOrientation`|屏幕方向|`portrait` / `landscape` / `sensor`|
|`android:launchMode`|启动模式|`standard` / `singleTop` / `singleTask` / `singleInstance`|

**intent-filter 说明**:

- `MAIN` + `LAUNCHER`: 标记为应用的主入口
- 只有启动 Activity 需要这个过滤器

---

## 🔄 第二部分：Activity 生命周期详解

### 2.1 为什么需要生命周期？

#### 现实问题场景

**场景1: 视频播放应用**

```
用户正在看视频
    ↓
突然来电话了
    ↓
问题：视频应该怎么办？

❌ 没有生命周期管理：
   - 视频继续播放（影响通话）
   - 继续消耗流量和电量

✅ 有生命周期管理：
   - onPause() 时自动暂停视频
   - 通话结束后 onResume() 继续播放
```

**场景2: 表单填写**

```
用户正在填写长表单
    ↓
接到微信消息，切换到微信
    ↓  
问题：表单数据会丢失吗？

❌ 没有生命周期管理：
   - 数据全部丢失
   - 用户需要重新填写（体验极差）

✅ 有生命周期管理：
   - onPause() 自动保存草稿
   - 返回时自动恢复数据
```

**场景3: 游戏应用**

```
用户正在玩游戏
    ↓
按 Home 键返回桌面
    ↓
问题：游戏继续运行吗？

❌ 没有生命周期管理：
   - 游戏继续运行（耗电、占内存）
   - 可能被系统强制杀死

✅ 有生命周期管理：
   - onPause() 暂停游戏
   - onStop() 保存游戏进度
   - onDestroy() 释放资源
```

#### 生命周期的作用

通过生命周期管理，我们可以：

1. **资源管理** 📦
    
    - 在合适时机申请资源（相机、GPS、传感器）
    - 在合适时机释放资源
2. **状态保存** 💾
    
    - 保存用户输入的数据
    - 恢复界面状态
3. **性能优化** ⚡
    
    - 避免不必要的后台操作
    - 减少电量和流量消耗
4. **用户体验** 😊
    
    - 提供流畅的交互体验
    - 避免数据丢失和应用崩溃

### 2.2 Activity 的三种状态

#### 状态1: 运行状态 (Running/Resumed)

**特征**:

```
✓ 位于屏幕前台
✓ 完全可见
✓ 可以接收用户输入  
✓ 拥有焦点
```

**示例**:

```kotlin
// MainActivity 正在运行
用户可以：
- 点击按钮 ✓
- 输入文字 ✓
- 滚动列表 ✓
- 播放动画 ✓
```

**视觉表示**:

```
┌─────────────────────────┐
│                         │
│    MainActivity         │  ← Running
│    (前台，可交互)        │
│                         │
└─────────────────────────┘
```

**系统优先级**: 🔴 最高（几乎不会被回收）

#### 状态2: 暂停状态 (Paused)

**特征**:

```
✓ 部分可见或完全可见
✗ 失去焦点
✗ 不能接收用户输入
```

**常见场景**:

**场景A - 对话框覆盖**:

```
┌─────────────────────────┐
│                         │
│    MainActivity         │  ← Paused  
│    (失去焦点但可见)      │
│    ┌─────────────┐      │
│    │ AlertDialog │      │  ← Running
│    │             │      │
│    └─────────────┘      │
│                         │
└─────────────────────────┘
```

**场景B - 半透明 Activity**:

```
┌─────────────────────────┐
│ TransparentActivity     │  ← Running
│    (半透明，有焦点)      │
├─────────────────────────┤
│    MainActivity         │  ← Paused
│    (可见但无焦点)        │
└─────────────────────────┘
```

**场景C - 分屏模式**:

```
┌─────────────────────────┐
│   App A (有焦点)        │  ← Running
├─────────────────────────┤
│   MainActivity          │  ← Paused
│   (可见但无焦点)         │
└─────────────────────────┘
```

**系统优先级**: 🟡 中等（可能被回收，但不常见）

#### 状态3: 停止状态 (Stopped)

**特征**:

```
✗ 完全不可见
✓ 仍在内存中  
✓ 保留所有状态和成员变量
✓ 保存在 Activity 栈中
```

**常见场景**:

```
场景1: 按 Home 键
MainActivity → Stopped
桌面 → Running

场景2: 切换到其他应用
MainActivity → Stopped
其他应用 → Running

场景3: 启动全屏 Activity
MainActivity → Stopped
SecondActivity → Running
```

**视觉表示**:

```
屏幕显示:
┌─────────────────────────┐
│   SecondActivity        │  ← Running (前台)
└─────────────────────────┘

后台（不可见但在内存中）:
[ MainActivity (Stopped) ]
```

**系统优先级**: 🟢 低（内存不足时可能被回收）

### 2.3 生命周期回调方法

Android 通过 **7个回调方法** 管理 Activity 生命周期：

```
完整生命周期流程:

        用户启动应用
             ↓
        onCreate()         ← [1] 创建（只调用一次）
             ↓
        onStart()          ← [2] 变为可见
             ↓
        onResume()         ← [3] 获得焦点，可交互
             ↓
    ╔═══════════════╗
    ║  运行中...     ║     ← 用户正在使用应用
    ╚═══════════════╝
             ↓
        onPause()          ← [4] 失去焦点
             ↓
        onStop()           ← [5] 完全不可见
             ↓
    ╔═══════════════╗
    ║  已停止        ║
    ╚═══════════════╝
             ↓
        两种可能:
        
   1) 重新显示:               2) 销毁:
      onRestart() ─┐             │
             ↓     │             ↓
      onStart() ←─┘         onDestroy()  ← [6] 销毁
             ↓                   
      onResume()
```

**方法对应关系**:

|创建方法|销毁方法|说明|
|---|---|---|
|`onCreate()`|`onDestroy()`|创建 ↔ 销毁|
|`onStart()`|`onStop()`|可见 ↔ 不可见|
|`onResume()`|`onPause()`|前台运行 ↔ 后台暂停|

### 2.4 各个回调方法详解

#### onCreate() - 创建

**调用时机**:

- Activity **第一次被创建**时
- 整个生命周期只调用**一次**

**主要任务**:

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var button: Button
    private lateinit var textView: TextView
    private var counter = 0
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 1. 设置布局文件（必须）
        setContentView(R.layout.activity_main)
        
        // 2. 初始化 View 组件
        button = findViewById(R.id.button)
        textView = findViewById(R.id.textView)
        
        // 3. 设置事件监听器
        button.setOnClickListener {
            counter++
            textView.text = "点击次数: $counter"
        }
        
        // 4. 初始化数据
        loadInitialData()
        
        // 5. 恢复保存的状态（如果有）
        savedInstanceState?.let {
            counter = it.getInt("counter", 0)
            textView.text = "点击次数: $counter"
        }
        
        Log.d("Lifecycle", "onCreate called")
    }
    
    private fun loadInitialData() {
        // 加载初始数据
    }
}
```

**⚠️ 重要提示**:

- 必须调用 `super.onCreate(savedInstanceState)`
- 必须在这里调用 `setContentView()`
- 避免执行耗时操作（会延迟界面显示）
- 耗时操作应该异步执行

**错误示例**:

```kotlin
// ❌ 不要这样做
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    
    // 耗时操作会阻塞 UI 线程
    Thread.sleep(5000)  // 糟糕的做法！
    loadLargeData()     // 同步加载大量数据
}

// ✅ 正确做法
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    
    // 异步加载数据
    lifecycleScope.launch {
        val data = withContext(Dispatchers.IO) {
            loadLargeData()
        }
        updateUI(data)
    }
}
```

#### onStart() - 变为可见

**调用时机**:

- `onCreate()` 之后
- `onRestart()` 之后（从停止状态恢复时）
- Activity 即将对用户可见

**主要任务**:

```kotlin
override fun onStart() {
    super.onStart()
    
    Log.d("Lifecycle", "onStart: Activity is becoming visible")
    
    // 1. 注册广播接收器
    val filter = IntentFilter(Intent.ACTION_TIME_TICK)
    registerReceiver(timeReceiver, filter)
    
    // 2. 初始化动画
    prepareAnimations()
    
    // 3. 开始监听数据变化
    observeData()
}
```

**特点**:

- Activity 可见但还不能交互
- 可能会被多次调用
- 与 `onStop()` 成对出现

#### onResume() - 开始交互

**调用时机**:

- `onStart()` 之后
- Activity 获得焦点，开始与用户交互

**主要任务**:

```kotlin
private var videoView: VideoView? = null
private var mediaPlayer: MediaPlayer? = null

override fun onResume() {
    super.onResume()
    
    Log.d("Lifecycle", "onResume: Activity is now in foreground")
    
    // 1. 恢复视频/音频播放
    videoView?.start()
    mediaPlayer?.start()
    
    // 2. 开始动画
    startAnimations()
    
    // 3. 开始传感器监听
    sensorManager?.registerListener(
        this,
        accelerometer,
        SensorManager.SENSOR_DELAY_NORMAL
    )
    
    // 4. 恢复游戏
    gameEngine?.resume()
    
    // 5. 刷新数据
    refreshData()
}
```

**使用场景**:

- 🎬 开始播放视频/音频
- 🎮 恢复游戏状态
- 📍 开始位置更新
- 📊 刷新显示的数据
- 🔔 注册实时监听器

#### onPause() - 失去焦点

**调用时机**:

- 系统准备启动或恢复另一个 Activity
- Activity 失去焦点（但可能仍然可见）

**主要任务**:

```kotlin
override fun onPause() {
    super.onPause()
    
    Log.d("Lifecycle", "onPause: Activity is losing focus")
    
    // 1. 暂停视频/音频
    videoView?.pause()
    mediaPlayer?.pause()
    
    // 2. 暂停动画
    stopAnimations()
    
    // 3. 暂停游戏
    gameEngine?.pause()
    
    // 4. 保存用户输入的临时数据（草稿）
    saveDraft()
}

private fun saveDraft() {
    val editor = getPreferences(MODE_PRIVATE).edit()
    editor.putString("draft", editText.text.toString())
    editor.putLong("save_time", System.currentTimeMillis())
    editor.apply()  // 异步保存
}
```

**⚠️ 关键注意事项**:

**1. 执行速度要快**

```kotlin
// ✅ 正确 - 快速执行
override fun onPause() {
    super.onPause()
    videoView.pause()
    saveToPreferences()  // SharedPreferences 很快
}

// ❌ 错误 - 耗时操作
override fun onPause() {
    super.onPause()
    saveToDatabase()   // 数据库操作慢
    uploadToServer()   // 网络操作更慢
    // ⚠️ 会阻塞下一个 Activity 的启动！
}
```

**为什么要快？**

```
MainActivity.onPause()  ← 必须快速完成
        ↓
    等待完成...  ← 用户看到卡顿
        ↓
SecondActivity.onCreate()
SecondActivity.onStart()
SecondActivity.onResume()  ← 用户终于看到新界面
```

**2. 不要在这里保存关键数据**

- 关键数据应该在 `onStop()` 中保存
- `onPause()` 可能被中断不执行完

#### onStop() - 完全不可见

**调用时机**:

- Activity 完全被其他 Activity 覆盖
- 用户切换到其他应用
- 用户按下 Home 键

**主要任务**:

```kotlin
override fun onStop() {
    super.onStop()
    
    Log.d("Lifecycle", "onStop: Activity is no longer visible")
    
    // 1. 释放相机资源
    releaseCamera()
    
    // 2. 注销传感器监听
    sensorManager?.unregisterListener(this)
    
    // 3. 取消网络请求
    cancelNetworkRequests()
    
    // 4. 保存数据到数据库（可以耗时一些）
    saveToDatabase()
    
    // 5. 注销广播接收器
    try {
        unregisterReceiver(timeReceiver)
    } catch (e: IllegalArgumentException) {
        // 已经注销过了
    }
    
    // 6. 释放大对象
    releaseLargeObjects()
}

private fun saveToDatabase() {
    lifecycleScope.launch {
        withContext(Dispatchers.IO) {
            database.userDao().updateUser(currentUser)
        }
    }
}

private fun releaseCamera() {
    camera?.release()
    camera = null
}
```

**使用场景**:

- 📷 释放硬件资源（相机、传感器、GPS）
- 💾 保存数据到数据库或文件
- 🌐 停止网络请求
- 🔌 注销广播接收器和监听器
- 🧹 释放占用大量内存的对象

#### onRestart() - 重新启动

**调用时机**:

- Activity 从 Stopped 状态重新变为可见
- 在 `onStart()` 之前调用

**主要任务**:

```kotlin
override fun onRestart() {
    super.onRestart()
    
    Log.d("Lifecycle", "onRestart: Activity is restarting")
    
    // 重新初始化在 onStop 中释放的资源
    reinitializeResources()
    
    // 刷新可能过期的数据
    refreshData()
}
```

**完整流程**:

```
用户切换回应用
    ↓
onRestart()     ← 准备重新启动
    ↓
onStart()       ← 变为可见
    ↓
onResume()      ← 可以交互
    ↓
Activity 重新运行
```

#### onDestroy() - 销毁

**调用时机**:

- 用户关闭 Activity（按返回键）
- 代码中调用 `finish()`
- 系统配置改变（如屏幕旋转）
- 系统需要回收内存

**主要任务**:

```kotlin
override fun onDestroy() {
    super.onDestroy()
    
    Log.d("Lifecycle", "onDestroy: Activity is being destroyed")
    
    // 1. 取消所有协程
    job?.cancel()
    
    // 2. 关闭数据库连接
    database?.close()
    
    // 3. 移除所有回调
    handler.removeCallbacksAndMessages(null)
    
    // 4. 释放所有资源
    cleanupAllResources()
}
```

**⚠️ 重要警告**:

- `onDestroy()` **不保证**一定会被调用
- 系统强制杀死进程时可能不会调用
- **不要依赖** `onDestroy()` 保存重要数据
- 重要数据应该在 `onPause()` 或 `onStop()` 中保存

### 2.5 完整生命周期场景分析

#### 场景1: 首次启动应用

```
用户点击应用图标
    ↓
onCreate()      → 创建 Activity，加载布局，初始化数据
    ↓
onStart()       → Activity 变为可见  
    ↓
onResume()      → Activity 获得焦点，可以交互
    ↓
【用户正在使用应用】
```

**Logcat 输出**:

```
D/MainActivity: onCreate called
D/MainActivity: onStart called
D/MainActivity: onResume called
```

#### 场景2: 按 Home 键

```
【Activity 正在运行】
    ↓
用户按 Home 键
    ↓
onPause()       → 失去焦点，暂停动画/音频
    ↓
onStop()        → 完全不可见，释放资源
    ↓
【Activity 在后台，但保留在内存中】
```

**Logcat 输出**:

```
D/MainActivity: onPause called
D/MainActivity: onStop called
```

#### 场景3: 从后台返回应用

```
【Activity 在后台】
    ↓
用户切换回应用
    ↓
onRestart()     → 准备重新启动
    ↓
onStart()       → 变为可见
    ↓
onResume()      → 可以交互
    ↓
【Activity 重新运行】
```

**Logcat 输出**:

```
D/MainActivity: onRestart called
D/MainActivity: onStart called  
D/MainActivity: onResume called
```

#### 场景4: 按返回键关闭

```
【Activity 正在运行】
    ↓
用户按返回键
    ↓
onPause()       → 失去焦点
    ↓
onStop()        → 完全不可见
    ↓
onDestroy()     → 销毁 Activity
    ↓
【Activity 被完全销毁，从内存中移除】
```

**Logcat 输出**:

```
D/MainActivity: onPause called
D/MainActivity: onStop called
D/MainActivity: onDestroy called
```

#### 场景5: 来电话（对话框形式）

```
【Activity 正在运行】
    ↓
电话来了（系统显示来电对话框）
    ↓
onPause()       → 失去焦点（但仍然可见）
    ↓
【通话对话框显示在前面】
    ↓
挂断电话，对话框消失
    ↓
onResume()      → 重新获得焦点
    ↓
【Activity 继续运行】
```

**Logcat 输出**:

```
D/MainActivity: onPause called
... (通话中，Activity 保持 Paused 状态) ...
D/MainActivity: onResume called
```

**注意**: 如果是全屏通话界面，会调用 `onStop()`

#### 场景6: 屏幕旋转 ⚠️

```
【Activity 竖屏运行】
    ↓
用户旋转屏幕到横屏
    ↓
onPause()       → 旧实例失去焦点
onStop()        → 旧实例不可见
onDestroy()     → 销毁旧实例
    ↓
onCreate()      → 创建新实例（横屏配置）
onStart()       → 新实例可见
onResume()      → 新实例可交互
    ↓
【Activity 以横屏模式运行】
```

**Logcat 输出**:

```
D/MainActivity: onPause called
D/MainActivity: onStop called
D/MainActivity: onDestroy called
D/MainActivity: onCreate called
D/MainActivity: onStart called
D/MainActivity: onResume called
```

**⚠️ 重要**:

- 屏幕旋转会完全**销毁并重建** Activity
- 如果不保存状态，数据会丢失
- 需要使用 `onSaveInstanceState()` 保存状态

**保存和恢复状态**:

```kotlin
// 保存状态
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putInt("counter", counter)
    outState.putString("user_input", editText.text.toString())
}

// 恢复状态  
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    
    savedInstanceState?.let {
        counter = it.getInt("counter", 0)
        val savedInput = it.getString("user_input", "")
        editText.setText(savedInput)
    }
}
```

#### 场景7: 启动另一个 Activity

```
【MainActivity 正在运行】
    ↓
startActivity(intent)
    ↓
MainActivity.onPause()
    ↓
SecondActivity.onCreate()
SecondActivity.onStart()
SecondActivity.onResume()
    ↓
MainActivity.onStop()
    ↓
【SecondActivity 运行，MainActivity 在后台】
```

**Logcat 输出**:

```
D/MainActivity: onPause called
D/SecondActivity: onCreate called
D/SecondActivity: onStart called
D/SecondActivity: onResume called
D/MainActivity: onStop called
```

**重要**: MainActivity 的 `onPause()` **先于** SecondActivity 的 `onCreate()`

---

## 🔍 第三部分：生命周期实践

### 3.1 完整的监控示例

创建一个 Activity 来监控所有生命周期回调：

```kotlin
package com.example.lifecycledemo

import android.os.Bundle
import android.util.Log
import android.widget.Button
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import java.text.SimpleDateFormat
import java.util.*

class LifecycleMonitorActivity : AppCompatActivity() {
    
    private val TAG = "LifecycleMonitor"
    private lateinit var statusText: TextView
    private lateinit var logText: TextView
    private val lifecycleEvents = mutableListOf<String>()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_lifecycle_monitor)
        
        statusText = findViewById(R.id.statusText)
        logText = findViewById(R.id.logText)
        
        findViewById<Button>(R.id.btnClear).setOnClickListener {
            lifecycleEvents.clear()
            updateLog()
        }
        
        logLifecycle("onCreate")
        updateStatus("Created")
    }
    
    override fun onStart() {
        super.onStart()
        logLifecycle("onStart")
        updateStatus("Started - Visible")
    }
    
    override fun onResume() {
        super.onResume()
        logLifecycle("onResume")
        updateStatus("Resumed - Running & Interactive")
    }
    
    override fun onPause() {
        super.onPause()
        logLifecycle("onPause")
        updateStatus("Paused - Lost Focus")
    }
    
    override fun onStop() {
        super.onStop()
        logLifecycle("onStop")
        updateStatus("Stopped - Not Visible")
    }
    
    override fun onRestart() {
        super.onRestart()
        logLifecycle("onRestart")
        updateStatus("Restarting...")
    }
    
    override fun onDestroy() {
        super.onDestroy()
        logLifecycle("onDestroy")
        
        // 打印完整的生命周期序列
        val sequence = lifecycleEvents.joinToString(" → ")
        Log.d(TAG, "=== Complete Lifecycle Sequence ===")
        Log.d(TAG, sequence)
    }
    
    private fun logLifecycle(method: String) {
        val timestamp = SimpleDateFormat("HH:mm:ss.SSS", Locale.getDefault())
            .format(Date())
        val logEntry = "[$timestamp] $method"
        
        Log.d(TAG, "=== $method ===")
        lifecycleEvents.add(method)
        updateLog()
    }
    
    private fun updateStatus(status: String) {
        statusText.text = "Current Status: $status"
    }
    
    private fun updateLog() {
        val eventList = lifecycleEvents.joinToString("\n") { "  • $it" }
        logText.text = """
            Lifecycle Events:
            
            $eventList
            
            Total: ${lifecycleEvents.size} events
        """.trimIndent()
    }
}
```

**布局文件** (`activity_lifecycle_monitor.xml`):

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/statusText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Current Status: Unknown"
        android:textSize="18sp"
        android:textStyle="bold"
        android:textColor="#028090"
        android:padding="12dp"
        android:background="#E3F2FD"
        android:layout_marginBottom="16dp" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <TextView
            android:id="@+id/logText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Lifecycle log will appear here..."
            android:textSize="14sp"
            android:padding="12dp"
            android:background="#F5F5F5"
            android:fontFamily="monospace" />

    </ScrollView>

    <Button
        android:id="@+id/btnClear"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Clear Log"
        android:layout_marginTop="16dp" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="试试这些操作:\n• 按 Home 键\n• 重新打开应用\n• 按返回键\n• 旋转屏幕"
        android:textSize="14sp"
        android:padding="8dp"
        android:background="#FFF3E0" />

</LinearLayout>
```

### 3.2 实验任务

#### 实验1: 观察基本生命周期

**目标**: 理解不同操作触发的生命周期变化

**步骤**:

1. 运行 LifecycleMonitorActivity
2. 执行以下操作并观察 Logcat：

|操作|预期生命周期方法|最终状态|
|---|---|---|
|启动应用|onCreate → onStart → onResume|Running|
|按 Home 键|onPause → onStop|Stopped|
|重新打开|onRestart → onStart → onResume|Running|
|按返回键|onPause → onStop → onDestroy|Destroyed|
|旋转屏幕|销毁并重建|Running (新实例)|

**记录表格**（请填写）:

|操作|实际观察到的生命周期|备注|
|---|---|---|
|启动应用|||
|按 Home 键|||
|重新打开|||
|按返回键|||
|旋转屏幕|||

---

## 🎯 第四部分：Intent 详解

### 4.1 什么是 Intent？

**Intent** 是 Android 中的消息对象，用于在组件之间传递信息和请求。

**Intent 的三大用途**:

```
1. 启动 Activity
   MainActivity → SecondActivity

2. 启动 Service
   Activity → BackgroundService

3. 发送广播
   Application → BroadcastReceiver
```

**Intent 就像一封"请求信"**:

```
"我想打开详情页"      → 启动 DetailActivity
"我想拨打电话"        → 启动系统拨号应用
"我想分享内容"        → 启动分享应用
"我想查看网页"        → 启动浏览器
```

### 4.2 Intent 的两种类型

#### 类型1: 显式 Intent (Explicit Intent)

**明确指定目标组件**：

```kotlin
// 创建显式 Intent
val intent = Intent(this, SecondActivity::class.java)

// 启动 Activity
startActivity(intent)
```

**特点**:

- ✅ 明确指定要启动的组件
- ✅ 用于应用内部的组件跳转
- ✅ 更安全、更明确

#### 类型2: 隐式 Intent (Implicit Intent)

**不指定具体组件，由系统选择合适的组件**：

```kotlin
// 打开网页
val intent = Intent(Intent.ACTION_VIEW).apply {
    data = Uri.parse("https://www.google.com")
}
startActivity(intent)

// 拨打电话
val intent = Intent(Intent.ACTION_DIAL).apply {
    data = Uri.parse("tel:10086")
}
startActivity(intent)

// 发送邮件
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_EMAIL, arrayOf("example@email.com"))
    putExtra(Intent.EXTRA_SUBJECT, "邮件主题")
    putExtra(Intent.EXTRA_TEXT, "邮件内容")
}
startActivity(Intent.createChooser(intent, "选择邮件应用"))

// 分享文本
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "分享的内容")
}
startActivity(Intent.createChooser(intent, "分享到"))
```

### 4.3 使用 Intent 启动 Activity

#### 基本用法

**MainActivity.kt** - 启动另一个 Activity:

```kotlin
package com.example.intentdemo

import android.content.Intent
import android.os.Bundle
import android.widget.Button
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        val button = findViewById<Button>(R.id.buttonGotoSecond)
        
        button.setOnClickListener {
            // 创建 Intent
            val intent = Intent(this, SecondActivity::class.java)
            
            // 启动 Activity
            startActivity(intent)
        }
    }
}
```

**SecondActivity.kt**:

```kotlin
package com.example.intentdemo

import android.os.Bundle
import android.widget.Button
import androidx.appcompat.app.AppCompatActivity

class SecondActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)
        
        val buttonBack = findViewById<Button>(R.id.buttonBack)
        
        buttonBack.setOnClickListener {
            // 结束当前 Activity，返回上一个
            finish()
        }
    }
}
```

**⚠️ 记得在 AndroidManifest.xml 中注册**:

```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>

<activity
    android:name=".SecondActivity"
    android:exported="false" />
```

### 4.4 数据传递 - putExtra & getExtra

Intent 可以携带数据在 Activity 之间传递。

#### 发送数据 (putExtra)

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        val button = findViewById<Button>(R.id.button)
        
        button.setOnClickListener {
            val intent = Intent(this, SecondActivity::class.java)
            
            // 传递不同类型的数据
            intent.putExtra("username", "Alice")
            intent.putExtra("age", 25)
            intent.putExtra("score", 95.5)
            intent.putExtra("isPremium", true)
            intent.putExtra("userId", 123456L)
            
            startActivity(intent)
        }
    }
}
```

#### 接收数据 (getExtra)

```kotlin
class SecondActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)
        
        // 接收数据
        val username = intent.getStringExtra("username") ?: "Guest"
        val age = intent.getIntExtra("age", 0)
        val score = intent.getDoubleExtra("score", 0.0)
        val isPremium = intent.getBooleanExtra("isPremium", false)
        val userId = intent.getLongExtra("userId", 0L)
        
        // 显示接收到的数据
        val textView = findViewById<TextView>(R.id.textView)
        textView.text = """
            Username: $username
            Age: $age
            Score: $score
            Premium: $isPremium
            User ID: $userId
        """.trimIndent()
        
        Log.d("SecondActivity", "Received data: $username, $age")
    }
}
```

### 4.5 支持的数据类型

#### 基本数据类型

|putExtra 方法|getExtra 方法|类型|示例|
|---|---|---|---|
|`putExtra(key, value)`|`getStringExtra(key)`|String?|`intent.putExtra("name", "Alice")`|
|`putExtra(key, value)`|`getIntExtra(key, default)`|Int|`intent.putExtra("age", 25)`|
|`putExtra(key, value)`|`getLongExtra(key, default)`|Long|`intent.putExtra("id", 123L)`|
|`putExtra(key, value)`|`getFloatExtra(key, default)`|Float|`intent.putExtra("price", 9.99f)`|
|`putExtra(key, value)`|`getDoubleExtra(key, default)`|Double|`intent.putExtra("score", 95.5)`|
|`putExtra(key, value)`|`getBooleanExtra(key, default)`|Boolean|`intent.putExtra("isVip", true)`|
|`putExtra(key, value)`|`getCharExtra(key, default)`|Char|`intent.putExtra("grade", 'A')`|

**⚠️ 注意**:

- `getStringExtra()` 返回可空类型 `String?`
- 其他基本类型的 `getExtra` 方法需要提供默认值

#### 数组和集合

```kotlin
// 发送数组
intent.putExtra("numbers", intArrayOf(1, 2, 3, 4, 5))
intent.putExtra("names", arrayOf("Alice", "Bob", "Charlie"))

// 发送 ArrayList
val fruitList = ArrayList<String>()
fruitList.add("Apple")
fruitList.add("Banana")
fruitList.add("Orange")
intent.putStringArrayListExtra("fruits", fruitList)

// 接收数组
val numbers = intent.getIntArrayExtra("numbers")
val names = intent.getStringArrayExtra("names")

// 接收 ArrayList
val fruits = intent.getStringArrayListExtra("fruits")
```

#### 传递对象 - Parcelable（推荐）

```kotlin
// 1. 定义数据类并实现 Parcelable
import android.os.Parcelable
import kotlinx.parcelize.Parcelize

@Parcelize
data class User(
    val id: Int,
    val name: String,
    val email: String,
    val age: Int
) : Parcelable

// 2. 发送对象
val user = User(1, "Alice", "alice@example.com", 25)
intent.putExtra("user", user)

// 3. 接收对象
val user = intent.getParcelableExtra<User>("user")
user?.let {
    Log.d("User", "ID: ${it.id}, Name: ${it.name}, Email: ${it.email}")
    textView.text = "Welcome, ${it.name}!"
}
```

**配置 Parcelize**:

在 `build.gradle.kts` (Module) 中添加：

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("kotlin-parcelize")  // 添加这一行
}
```

#### 传递对象 - Serializable

```kotlin
// 1. 定义数据类并实现 Serializable
data class User(
    val id: Int,
    val name: String,
    val email: String
) : java.io.Serializable

// 2. 发送对象
val user = User(1, "Alice", "alice@example.com")
intent.putExtra("user", user)

// 3. 接收对象
val user = intent.getSerializableExtra("user") as? User
user?.let {
    Log.d("User", "Name: ${it.name}")
}
```

**Parcelable vs Serializable**:

- ✅ **Parcelable**: 性能更好，Android 推荐
- ❌ **Serializable**: 性能较差，但实现简单

### 4.6 使用常量定义 Key

**最佳实践**: 使用常量定义 key，避免拼写错误

```kotlin
class SecondActivity : AppCompatActivity() {
    
    companion object {
        // 定义常量
        const val EXTRA_USERNAME = "extra_username"
        const val EXTRA_AGE = "extra_age"
        const val EXTRA_USER_ID = "extra_user_id"
        
        // 提供便捷方法创建 Intent（推荐）
        fun newIntent(
            context: Context,
            username: String,
            age: Int,
            userId: Long
        ): Intent {
            return Intent(context, SecondActivity::class.java).apply {
                putExtra(EXTRA_USERNAME, username)
                putExtra(EXTRA_AGE, age)
                putExtra(EXTRA_USER_ID, userId)
            }
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)
        
        // 使用常量接收数据
        val username = intent.getStringExtra(EXTRA_USERNAME) ?: "Guest"
        val age = intent.getIntExtra(EXTRA_AGE, 0)
        val userId = intent.getLongExtra(EXTRA_USER_ID, 0L)
        
        displayUserInfo(username, age, userId)
    }
}

// 在 MainActivity 中使用
class MainActivity : AppCompatActivity() {
    private fun openSecondActivity() {
        // 方式1: 使用常量
        val intent = Intent(this, SecondActivity::class.java)
        intent.putExtra(SecondActivity.EXTRA_USERNAME, "Alice")
        intent.putExtra(SecondActivity.EXTRA_AGE, 25)
        intent.putExtra(SecondActivity.EXTRA_USER_ID, 12345L)
        startActivity(intent)
        
        // 方式2: 使用便捷方法（推荐）
        val intent2 = SecondActivity.newIntent(this, "Alice", 25, 12345L)
        startActivity(intent2)
    }
}
```

### 4.7 Activity 返回结果

#### 新方式：Activity Result API（推荐）

从 Android 10 开始，推荐使用 Activity Result API 替代 `startActivityForResult()`。

**MainActivity.kt** - 启动并等待结果:

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var resultText: TextView
    
    // 1. 注册 ActivityResultLauncher
    private val resultLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        // 3. 处理返回结果
        if (result.resultCode == RESULT_OK) {
            val data = result.data
            val username = data?.getStringExtra("username") ?: "Unknown"
            val age = data?.getIntExtra("age", 0) ?: 0
            
            Toast.makeText(
                this,
                "返回数据: $username, $age 岁",
                Toast.LENGTH_SHORT
            ).show()
            
            // 更新 UI
            resultText.text = "用户: $username, 年龄: $age"
        } else {
            Toast.makeText(this, "操作已取消", Toast.LENGTH_SHORT).show()
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        resultText = findViewById(R.id.resultText)
        
        findViewById<Button>(R.id.btnLaunch).setOnClickListener {
            // 2. 启动 Activity
            val intent = Intent(this, InputActivity::class.java)
            resultLauncher.launch(intent)
        }
    }
}
```

**InputActivity.kt** - 返回结果:

```kotlin
class InputActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_input)
        
        val nameEdit = findViewById<EditText>(R.id.nameEdit)
        val ageEdit = findViewById<EditText>(R.id.ageEdit)
        val btnConfirm = findViewById<Button>(R.id.btnConfirm)
        val btnCancel = findViewById<Button>(R.id.btnCancel)
        
        // 确认按钮
        btnConfirm.setOnClickListener {
            val name = nameEdit.text.toString()
            val age = ageEdit.text.toString().toIntOrNull() ?: 0
            
            // 准备返回数据
            val resultIntent = Intent()
            resultIntent.putExtra("username", name)
            resultIntent.putExtra("age", age)
            
            // 设置结果并关闭
            setResult(RESULT_OK, resultIntent)
            finish()
        }
        
        // 取消按钮
        btnCancel.setOnClickListener {
            setResult(RESULT_CANCELED)
            finish()
        }
    }
}
```

---

## 📝 第五部分：完整实例 - 三页面导航应用

### 5.1 应用结构

```
三页面导航应用
├── MainActivity (主页)
│   ├─> ProfileActivity (个人信息)
│   └─> SettingsActivity (设置，返回结果)
```

### 5.2 MainActivity.kt

```kotlin
package com.example.navigationdemo

import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    
    private lateinit var welcomeText: TextView
    private lateinit var settingsResultText: TextView
    private var username = "游客"
    
    // 注册结果接收器
    private val settingsLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == RESULT_OK) {
            val theme = result.data?.getStringExtra("theme") ?: "默认"
            settingsResultText.text = "当前主题: $theme"
            Toast.makeText(this, "主题已更改为: $theme", Toast.LENGTH_SHORT).show()
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        welcomeText = findViewById(R.id.welcomeText)
        settingsResultText = findViewById(R.id.settingsResultText)
        
        updateWelcome()
        
        // 跳转到个人信息页
        findViewById<Button>(R.id.btnProfile).setOnClickListener {
            val intent = Intent(this, ProfileActivity::class.java)
            intent.putExtra("username", username)
            intent.putExtra("level", 5)
            intent.putExtra("points", 1280)
            startActivity(intent)
        }
        
        // 跳转到设置页（等待返回结果）
        findViewById<Button>(R.id.btnSettings).setOnClickListener {
            val intent = Intent(this, SettingsActivity::class.java)
            settingsLauncher.launch(intent)
        }
    }
    
    override fun onResume() {
        super.onResume()
        updateWelcome()
    }
    
    private fun updateWelcome() {
        welcomeText.text = "欢迎, $username!"
    }
}
```

**activity_main.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp"
    android:gravity="center">

    <TextView
        android:id="@+id/welcomeText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="欢迎!"
        android:textSize="32sp"
        android:textStyle="bold"
        android:textColor="#028090"
        android:layout_marginBottom="16dp" />

    <TextView
        android:id="@+id/settingsResultText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="当前主题: 默认"
        android:textSize="16sp"
        android:layout_marginBottom="48dp" />

    <Button
        android:id="@+id/btnProfile"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="个人信息"
        android:textSize="18sp"
        android:padding="16dp"
        android:layout_marginBottom="16dp" />

    <Button
        android:id="@+id/btnSettings"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="设置"
        android:textSize="18sp"
        android:padding="16dp" />

</LinearLayout>
```

### 5.3 ProfileActivity.kt

```kotlin
package com.example.navigationdemo

import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity

class ProfileActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_profile)
        
        // 接收数据
        val username = intent.getStringExtra("username") ?: "未知用户"
        val level = intent.getIntExtra("level", 1)
        val points = intent.getIntExtra("points", 0)
        
        // 显示数据
        findViewById<TextView>(R.id.usernameText).text = "用户名: $username"
        findViewById<TextView>(R.id.levelText).text = "等级: Lv.$level"
        findViewById<TextView>(R.id.pointsText).text = "积分: $points"
        
        // 返回按钮
        findViewById<Button>(R.id.btnBack).setOnClickListener {
            finish()
        }
    }
}
```

**activity_profile.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="个人信息"
        android:textSize="28sp"
        android:textStyle="bold"
        android:textColor="#028090"
        android:layout_marginBottom="32dp" />

    <TextView
        android:id="@+id/usernameText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="用户名: "
        android:textSize="18sp"
        android:padding="12dp"
        android:background="#F5F5F5"
        android:layout_marginBottom="16dp" />

    <TextView
        android:id="@+id/levelText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="等级: "
        android:textSize="18sp"
        android:padding="12dp"
        android:background="#F5F5F5"
        android:layout_marginBottom="16dp" />

    <TextView
        android:id="@+id/pointsText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="积分: "
        android:textSize="18sp"
        android:padding="12dp"
        android:background="#F5F5F5"
        android:layout_marginBottom="32dp" />

    <Button
        android:id="@+id/btnBack"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="返回"
        android:textSize="18sp" />

</LinearLayout>
```

### 5.4 SettingsActivity.kt

```kotlin
package com.example.navigationdemo

import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.RadioGroup
import androidx.appcompat.app.AppCompatActivity

class SettingsActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_settings)
        
        val themeGroup = findViewById<RadioGroup>(R.id.themeGroup)
        val btnSave = findViewById<Button>(R.id.btnSave)
        val btnCancel = findViewById<Button>(R.id.btnCancel)
        
        // 保存按钮
        btnSave.setOnClickListener {
            // 获取选中的主题
            val theme = when (themeGroup.checkedRadioButtonId) {
                R.id.radioLight -> "浅色主题"
                R.id.radioDark -> "深色主题"
                R.id.radioAuto -> "跟随系统"
                else -> "默认主题"
            }
            
            // 返回结果
            val resultIntent = Intent()
            resultIntent.putExtra("theme", theme)
            setResult(RESULT_OK, resultIntent)
            finish()
        }
        
        // 取消按钮
        btnCancel.setOnClickListener {
            setResult(RESULT_CANCELED)
            finish()
        }
    }
}
```

**activity_settings.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="设置"
        android:textSize="28sp"
        android:textStyle="bold"
        android:textColor="#028090"
        android:layout_marginBottom="32dp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="主题设置"
        android:textSize="18sp"
        android:textStyle="bold"
        android:layout_marginBottom="16dp" />

    <RadioGroup
        android:id="@+id/themeGroup"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="32dp">

        <RadioButton
            android:id="@+id/radioLight"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="浅色主题"
            android:textSize="16sp"
            android:checked="true"
            android:padding="8dp" />

        <RadioButton
            android:id="@+id/radioDark"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="深色主题"
            android:textSize="16sp"
            android:padding="8dp" />

        <RadioButton
            android:id="@+id/radioAuto"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="跟随系统"
            android:textSize="16sp"
            android:padding="8dp" />

    </RadioGroup>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <Button
            android:id="@+id/btnCancel"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="取消"
            android:layout_marginEnd="8dp"
            style="@style/Widget.Material3.Button.OutlinedButton" />

        <Button
            android:id="@+id/btnSave"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="保存"
            android:layout_marginStart="8dp" />

    </LinearLayout>

</LinearLayout>
```

### 5.5 AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.navigationdemo">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.NavigationDemo">
        
        <!-- 主 Activity -->
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <!-- 个人信息页 -->
        <activity
            android:name=".ProfileActivity"
            android:exported="false"
            android:label="个人信息" />
        
        <!-- 设置页 -->
        <activity
            android:name=".SettingsActivity"
            android:exported="false"
            android:label="设置" />
        
    </application>

</manifest>
```

---

## 💡 第六部分：最佳实践

### 6.1 生命周期管理

#### 1. 合理使用生命周期方法

```kotlin
class GoodActivity : AppCompatActivity() {
    
    // ✅ onCreate: 一次性初始化
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        initViews()
        setupListeners()
    }
    
    // ✅ onResume: 开始前台操作
    override fun onResume() {
        super.onResume()
        startLocationUpdates()
        resumeVideo()
        refreshData()
    }
    
    // ✅ onPause: 快速暂停（不要耗时）
    override fun onPause() {
        super.onPause()
        pauseVideo()  // 快速
        saveDraft()   // 快速保存
        // ❌ 避免: saveToDatabase() - 耗时操作
    }
    
    // ✅ onStop: 释放资源（可以耗时）
    override fun onStop() {
        super.onStop()
        stopLocationUpdates()
        saveToDatabase()  // 可以做耗时操作
        releaseCamera()
    }
    
    // ✅ onDestroy: 最终清理
    override fun onDestroy() {
        super.onDestroy()
        cancelAllTasks()
        closeDatabase()
    }
}
```

#### 2. 处理配置变化

**方式1: 保存和恢复状态**

```kotlin
class MyActivity : AppCompatActivity() {
    
    private var counter = 0
    private var userInput = ""
    
    // 保存状态
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("counter", counter)
        outState.putString("user_input", editText.text.toString())
    }
    
    // 恢复状态
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        savedInstanceState?.let {
            counter = it.getInt("counter", 0)
            userInput = it.getString("user_input", "")
            editText.setText(userInput)
        }
    }
}
```

**方式2: 使用 ViewModel（推荐）**

```kotlin
// ViewModel 在配置变化时不会被销毁
class MyViewModel : ViewModel() {
    var counter = 0
    var userInput = ""
}

class MyActivity : AppCompatActivity() {
    private val viewModel: MyViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 数据会自动保留
        textView.text = viewModel.counter.toString()
        editText.setText(viewModel.userInput)
    }
}
```

### 6.2 Intent 使用最佳实践

#### 1. 使用常量和工厂方法

```kotlin
class DetailActivity : AppCompatActivity() {
    
    companion object {
        private const val EXTRA_USER_ID = "user_id"
        private const val EXTRA_USERNAME = "username"
        
        // ✅ 推荐：提供工厂方法
        fun newIntent(context: Context, userId: Int, username: String): Intent {
            return Intent(context, DetailActivity::class.java).apply {
                putExtra(EXTRA_USER_ID, userId)
                putExtra(EXTRA_USERNAME, username)
            }
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_detail)
        
        val userId = intent.getIntExtra(EXTRA_USER_ID, -1)
        val username = intent.getStringExtra(EXTRA_USERNAME) ?: ""
    }
}

// 使用
val intent = DetailActivity.newIntent(this, 123, "Alice")
startActivity(intent)
```

#### 2. 检查空值

```kotlin
// ❌ 不好
val username = intent.getStringExtra("username")
textView.text = username  // 可能崩溃（NPE）

// ✅ 好
val username = intent.getStringExtra("username") ?: "默认用户"
textView.text = username

// ✅ 或使用 let
intent.getStringExtra("username")?.let { username ->
    textView.text = username
}
```

#### 3. 避免传递大量数据

```kotlin
// ❌ 不好：传递大对象
val largeList = List(10000) { User(it, "User$it") }
intent.putParcelableArrayListExtra("users", ArrayList(largeList))

// ✅ 好：传递 ID，在目标 Activity 中加载
intent.putIntArrayExtra("user_ids", userIds)

// 在 DetailActivity 中
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val userIds = intent.getIntArrayExtra("user_ids")
    // 从数据库或网络加载用户数据
    loadUsers(userIds)
}
```

---

## 📝 课后作业

### 作业：创建三页面导航应用

#### 要求

创建一个包含三个 Activity 的应用，实现以下功能：

**第1页 (MainActivity - 主页)**:

- 显示欢迎信息
- 两个按钮分别跳转到第2页和第3页
- 能够接收并显示从第3页返回的数据

**第2页 (ProfileActivity - 个人资料)**:

- 接收从第1页传来的用户数据（姓名、年龄、城市）
- 显示所有接收到的数据
- 提供返回按钮

**第3页 (SettingsActivity - 设置)**:

- 包含输入框或选择器（如主题选择、语言选择）
- 保存按钮：返回选择的设置给第1页
- 取消按钮：不返回数据直接退出

#### 功能要求

1. **数据传递**:
    
    - MainActivity → ProfileActivity: 传递用户信息（姓名、年龄、城市）
    - SettingsActivity → MainActivity: 返回设置信息
2. **生命周期监控**:
    
    - 在每个 Activity 的所有生命周期方法中打印日志
    - 格式：`Log.d("ActivityName", "method_name called")`
3. **UI 设计**:
    
    - 布局美观、元素对齐
    - 使用 ConstraintLayout 或 LinearLayout
    - 合理使用颜色和间距

#### 提交内容

1. **源代码** (ZIP 压缩包):
    
    - 完整的 Android 项目
    - 包含所有 Activity 和布局文件
2. **运行截图** (至少5张):
    
    - MainActivity 初始界面
    - ProfileActivity 显示数据
    - SettingsActivity 设置界面
    - 数据返回后的 MainActivity
    - Logcat 生命周期日志截图
3. **说明文档** (README.md):
    
    - 应用功能描述
    - 页面跳转流程图
    - 数据传递说明
    - 遇到的问题和解决方案

#### 评分标准

|项目|分值|说明|
|---|---|---|
|功能完整性|40%|所有功能正常工作|
|生命周期日志|20%|正确打印所有生命周期方法|
|数据传递|20%|正确传递和接收数据|
|UI 设计|10%|界面美观、布局合理|
|代码质量|10%|代码规范、有注释|

#### 加分项（可选）

- ✨ 使用 ViewBinding
- ✨ 使用自定义主题和样式
- ✨ 添加页面切换动画
- ✨ 使用 Parcelable 传递自定义对象
- ✨ 使用 ViewModel 管理数据
- ✨ 添加表单验证

#### 提交方式

**文件命名**: `姓名_学号_Week4_导航应用.zip`

**截止时间**: 第5周周一上课前

---

## 🎯 课后思考题

1. **为什么 Activity 需要生命周期？如果没有生命周期会有什么问题？**
    
2. **`onPause()` 和 `onStop()` 有什么区别？在什么场景下只会调用 `onPause()` 而不调用 `onStop()`？**
    
3. **屏幕旋转时 Activity 会被重建，如何保存和恢复数据？有哪些方法？各有什么优缺点？**
    
4. **为什么推荐使用 Parcelable 而不是 Serializable 传递对象？**
    
5. **如何避免在 `onPause()` 中执行耗时操作？如果必须要保存大量数据应该怎么做？**
    

---

## 📚 扩展阅读

### 官方文档

- [Activity 生命周期官方文档](https://developer.android.com/guide/components/activities/activity-lifecycle)
- [Intent 和 Intent Filter](https://developer.android.com/guide/components/intents-filters)
- [Parcelable 和 Bundle](https://developer.android.com/guide/components/activities/parcelables-and-bundles)
- [Activity Result API](https://developer.android.com/training/basics/intents/result)

### 推荐文章

- [理解 Activity 生命周期](https://medium.com/androiddevelopers/the-android-lifecycle-cheat-sheet-part-i-single-activities-e49fd3d202ab)
- [Handling Lifecycles with Lifecycle-Aware Components](https://developer.android.com/topic/libraries/architecture/lifecycle)

### 视频教程

- [Android Developers - Activity Lifecycle](https://www.youtube.com/watch?v=RiFui-i-s-o)
- [Understanding Android Intents](https://www.youtube.com/watch?v=g1sAJIWCnJg)

---

## 🎓 下周预告

**第5周：RecyclerView 与列表展示**

- RecyclerView 基础概念
- Adapter 和 ViewHolder 模式
- LayoutManager（线性、网格、瀑布流）
- 点击事件处理
- 实践：创建商品列表应用

**预习建议**:

- 复习本周的 Activity 和 Intent 知识
- 了解列表和适配器的概念
- 思考：如何高效显示大量数据？

---

**课程资料更新时间**: 2026年2月  
**讲师**: [教师姓名]  
**联系方式**: [邮箱]

_继续努力！Activity 和 Intent 是 Android 开发的核心基础！🚀_