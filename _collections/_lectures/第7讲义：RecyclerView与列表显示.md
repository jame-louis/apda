---
title: 第7讲义：RecyclerView与列表显示
date: 2026-02-05
permalink: /lectures/lecture07
show: true
---


**课程**: Android 移动应用开发入门  
**周次**: 第7周  
**主题**: RecyclerView与列表显示  
**学时**: 3课时（理论2课时 + 实验1课时）

---

## 📋 本节课学习目标

完成本节课后，学生应该能够：

1. ✅ 理解 RecyclerView 的工作原理和优势
2. ✅ 掌握 Adapter 模式在列表显示中的作用
3. ✅ 实现 ViewHolder 设计模式提升性能
4. ✅ 创建自定义列表项布局
5. ✅ 处理列表项的点击事件
6. ✅ 独立开发一个联系人列表应用
7. ✅ 完成新闻列表应用界面开发

---

## 📱 第一部分：为什么需要 RecyclerView？

### 1.1 回顾 ListView 的问题

上周我们用 `ArrayAdapter` + `ListView` 显示笔记列表：

```kotlin
// Week 6 代码回顾
val displayItems = listOf("笔记1", "笔记2", "笔记3", ...)
val adapter = ArrayAdapter(this, android.R.layout.simple_list_item_1, displayItems)
listView.adapter = adapter
```

**ListView 的局限性**:

```
❌ 只能垂直滚动
❌ 自定义布局复杂
❌ 性能优化需要手动处理 ViewHolder
❌ 没有内置动画效果
❌ 不支持网格、瀑布流等布局
```

### 1.2 RecyclerView 的优势

**RecyclerView** 是 Google 在 Android 5.0 引入的**新一代列表组件**，完全替代 ListView。

```
✅ 支持多种布局（垂直、横向、网格、瀑布流）
✅ 强制使用 ViewHolder 模式（性能优秀）
✅ 内置动画效果
✅ 灵活的自定义能力
✅ 解耦设计（LayoutManager、Adapter、ViewHolder 各司其职）
```

**对比图**:

```
ListView:
┌─────────────┐
│  Item 1     │  ← 滚动时每次都 findViewById
│  Item 2     │  ← 滚动时每次都 findViewById
│  Item 3     │  ← 滚动时每次都 findViewById
└─────────────┘

RecyclerView + ViewHolder:
┌─────────────┐
│  Item 1     │  ← ViewHolder 缓存，复用控件
│  Item 2     │  ← ViewHolder 缓存，复用控件
│  Item 3     │  ← ViewHolder 缓存，复用控件
└─────────────┘
```

### 1.3 RecyclerView 的三大核心组件

```
┌──────────────────────────────────────┐
│          RecyclerView               │
├──────────────────────────────────────┤
│                                      │
│  ┌────────────────────────────────┐ │
│  │  LayoutManager                 │ │  ← 决定如何排列（垂直/横向/网格）
│  └────────────────────────────────┘ │
│                                      │
│  ┌────────────────────────────────┐ │
│  │  Adapter                       │ │  ← 数据 → 视图的桥梁
│  │  ├─ onCreateViewHolder()       │ │
│  │  ├─ onBindViewHolder()         │ │
│  │  └─ getItemCount()             │ │
│  └────────────────────────────────┘ │
│                                      │
│  ┌────────────────────────────────┐ │
│  │  ViewHolder                    │ │  ← 缓存列表项的控件引用
│  │  (缓存 findViewById 结果)      │ │
│  └────────────────────────────────┘ │
│                                      │
└──────────────────────────────────────┘
```

---

## 🔧 第二部分：RecyclerView 基础使用

### 2.1 添加依赖

RecyclerView 不是 Android SDK 自带的，需要在 `build.gradle` 中添加依赖：

```gradle
dependencies {
    implementation 'androidx.recyclerview:recyclerview:1.3.2'
}
```

### 2.2 在布局中添加 RecyclerView

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

### 2.3 创建列表项布局

每个列表项的布局是独立的 XML 文件。

**item_contact.xml** — 联系人列表项:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="16dp"
    android:background="?attr/selectableItemBackground">

    <!-- 头像 -->
    <TextView
        android:id="@+id/avatarText"
        android:layout_width="48dp"
        android:layout_height="48dp"
        android:gravity="center"
        android:text="A"
        android:textSize="20sp"
        android:textStyle="bold"
        android:textColor="@android:color/white"
        android:background="@drawable/circle_bg" />

    <!-- 信息区域 -->
    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:orientation="vertical"
        android:layout_marginStart="16dp"
        android:layout_gravity="center_vertical">

        <TextView
            android:id="@+id/nameText"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="张三"
            android:textSize="18sp"
            android:textStyle="bold"
            android:textColor="#212121" />

        <TextView
            android:id="@+id/phoneText"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="138-0000-0000"
            android:textSize="14sp"
            android:textColor="#757575"
            android:layout_marginTop="4dp" />

    </LinearLayout>

</LinearLayout>
```

**drawable/circle_bg.xml** — 圆形背景:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="#028090" />
</shape>
```

### 2.4 创建数据类

```kotlin
data class Contact(
    val name: String,
    val phone: String
)
```

### 2.5 创建 ViewHolder

**ViewHolder 的作用**: 缓存列表项中的控件引用，避免重复 `findViewById`。

```kotlin
class ContactViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    val avatarText: TextView = itemView.findViewById(R.id.avatarText)
    val nameText: TextView = itemView.findViewById(R.id.nameText)
    val phoneText: TextView = itemView.findViewById(R.id.phoneText)
}
```

### 2.6 创建 Adapter

**Adapter 的三大核心方法**:

```kotlin
class ContactAdapter(private val contacts: List<Contact>) : 
    RecyclerView.Adapter<ContactViewHolder>() {

    // ① 创建 ViewHolder（只在需要新视图时调用）
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ContactViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_contact, parent, false)
        return ContactViewHolder(view)
    }

    // ② 绑定数据到 ViewHolder（每次滚动都会调用）
    override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
        val contact = contacts[position]
        holder.nameText.text = contact.name
        holder.phoneText.text = contact.phone
        holder.avatarText.text = contact.name.first().toString()  // 首字母
    }

    // ③ 返回数据总数
    override fun getItemCount(): Int = contacts.size
}
```

**方法调用时机解析**:

```
App 启动，显示列表:
→ getItemCount() 被调用 → 返回 100（假设有100条数据）
→ RecyclerView 判断屏幕能显示 10 条
→ onCreateViewHolder() 被调用 10 次（创建 10 个 ViewHolder）
→ onBindViewHolder() 被调用 10 次（填充数据）

用户向下滚动:
→ 第 11 条进入屏幕
→ onCreateViewHolder() 不调用（复用第 1 条的 ViewHolder）
→ onBindViewHolder(position=10) 被调用（填充第 11 条数据）
```

### 2.7 在 Activity 中使用

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // ── 准备数据 ──
        val contacts = listOf(
            Contact("张三", "138-0000-0001"),
            Contact("李四", "138-0000-0002"),
            Contact("王五", "138-0000-0003"),
            Contact("赵六", "138-0000-0004"),
            Contact("钱七", "138-0000-0005")
        )

        // ── 设置 RecyclerView ──
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerView)
        
        // 设置布局管理器（垂直列表）
        recyclerView.layoutManager = LinearLayoutManager(this)
        
        // 设置适配器
        recyclerView.adapter = ContactAdapter(contacts)
    }
}
```

**完整流程总结**:

```
1. 定义数据类 Contact
2. 创建列表项布局 item_contact.xml
3. 创建 ViewHolder（缓存控件引用）
4. 创建 Adapter（实现三大方法）
5. 在 Activity 中:
   ① 准备数据 List<Contact>
   ② 设置 LayoutManager
   ③ 设置 Adapter
```

---

## 🎨 第三部分：Adapter 模式深入理解

### 3.1 什么是 Adapter 模式？

**Adapter（适配器）** 是一种设计模式，作用是**将数据转换为视图可以显示的格式**。

**生活类比**:

```
┌─────────────┐       ┌──────────┐       ┌─────────────┐
│  数据源     │ ───▶  │ Adapter  │ ───▶  │ RecyclerView│
│ List<Data>  │       │  (适配器) │       │   (视图)    │
└─────────────┘       └──────────┘       └─────────────┘

就像:
┌─────────────┐       ┌──────────┐       ┌─────────────┐
│  110V 电源  │ ───▶  │ 电源适配器│ ───▶  │  220V 设备  │
└─────────────┘       └──────────┘       └─────────────┘
```

### 3.2 为什么需要 Adapter？

```
没有 Adapter 的世界:
RecyclerView 需要自己处理:
- 如何从数据源获取数据？
- 如何创建列表项视图？
- 如何填充数据到视图？
→ RecyclerView 会变得无比复杂

有了 Adapter:
RecyclerView: "我只负责滚动和显示"
Adapter: "我负责把数据转换成视图"
→ 职责清晰，易于维护
```

### 3.3 Adapter 的三大核心方法详解

#### onCreateViewHolder()

**作用**: 创建新的 ViewHolder（只在需要时调用，不是每次滚动都调用）

```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ContactViewHolder {
    // parent: RecyclerView 本身
    // viewType: 列表项类型（用于多种类型列表项）
    
    val view = LayoutInflater.from(parent.context)
        .inflate(R.layout.item_contact, parent, false)
    return ContactViewHolder(view)
}
```

**何时调用**:

```
首次显示: 屏幕能显示 10 条 → 调用 10 次
滚动时: 有缓存就复用，没缓存才调用
```

#### onBindViewHolder()

**作用**: 将数据绑定到 ViewHolder（每次滚动都可能调用）

```kotlin
override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
    // holder: 要填充的 ViewHolder
    // position: 数据在 List 中的位置（从 0 开始）
    
    val contact = contacts[position]
    holder.nameText.text = contact.name
    holder.phoneText.text = contact.phone
    holder.avatarText.text = contact.name.first().toString()
}
```

**何时调用**:

```
每次有列表项进入屏幕时都会调用
滚动频繁时，这个方法会被大量调用
```

#### getItemCount()

**作用**: 告诉 RecyclerView 一共有多少条数据

```kotlin
override fun getItemCount(): Int = contacts.size
```

### 3.4 ViewHolder 的重要性

**没有 ViewHolder 的代码（ListView 时代）**:

```kotlin
// ❌ 每次滚动都重复 findViewById（性能差）
override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
    val view = convertView ?: layoutInflater.inflate(R.layout.item, parent, false)
    
    // 每次都要 findViewById，太慢！
    val nameText = view.findViewById<TextView>(R.id.nameText)
    val phoneText = view.findViewById<TextView>(R.id.phoneText)
    
    nameText.text = contacts[position].name
    phoneText.text = contacts[position].phone
    
    return view
}
```

**有 ViewHolder 的代码（RecyclerView）**:

```kotlin
// ✅ ViewHolder 缓存了控件引用
class ContactViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    val nameText: TextView = itemView.findViewById(R.id.nameText)    // 只调用一次
    val phoneText: TextView = itemView.findViewById(R.id.phoneText)  // 只调用一次
}

override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
    // 直接使用缓存的控件，不需要 findViewById
    holder.nameText.text = contacts[position].name
    holder.phoneText.text = contacts[position].phone
}
```

**性能对比**:

```
假设列表有 100 条数据，用户滚动浏览所有:

没有 ViewHolder:
→ findViewById 被调用 200 次（nameText + phoneText × 100）
→ 滚动卡顿

有 ViewHolder:
→ findViewById 只在创建时调用 20 次（假设屏幕显示 10 条）
→ 滚动流畅
```

---

## 🔄 第四部分：LayoutManager — 控制布局方式

### 4.1 三种常用 LayoutManager

|LayoutManager|效果|使用场景|
|---|---|---|
|`LinearLayoutManager`|垂直或横向列表|联系人列表、新闻列表|
|`GridLayoutManager`|网格布局|相册、应用列表|
|`StaggeredGridLayoutManager`|瀑布流|图片墙、Pinterest 风格|

### 4.2 LinearLayoutManager — 垂直/横向列表

```kotlin
// 垂直列表（默认）
recyclerView.layoutManager = LinearLayoutManager(this)

// 横向列表
recyclerView.layoutManager = LinearLayoutManager(
    this, 
    LinearLayoutManager.HORIZONTAL,  // 横向
    false                             // 是否反转
)
```

### 4.3 GridLayoutManager — 网格布局

```kotlin
// 2 列网格
recyclerView.layoutManager = GridLayoutManager(this, 2)

// 3 列网格，横向滚动
recyclerView.layoutManager = GridLayoutManager(
    this, 
    3,                                  // 列数
    GridLayoutManager.HORIZONTAL,       // 横向
    false                               // 是否反转
)
```

### 4.4 StaggeredGridLayoutManager — 瀑布流

```kotlin
// 2 列瀑布流
recyclerView.layoutManager = StaggeredGridLayoutManager(
    2,                                          // 列数
    StaggeredGridLayoutManager.VERTICAL         // 方向
)
```

### 4.5 切换布局演示

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: ContactAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        recyclerView = findViewById(R.id.recyclerView)
        adapter = ContactAdapter(getContacts())
        
        // 默认垂直列表
        recyclerView.layoutManager = LinearLayoutManager(this)
        recyclerView.adapter = adapter

        // 切换按钮
        findViewById<Button>(R.id.btnSwitchToGrid).setOnClickListener {
            recyclerView.layoutManager = GridLayoutManager(this, 2)
        }

        findViewById<Button>(R.id.btnSwitchToList).setOnClickListener {
            recyclerView.layoutManager = LinearLayoutManager(this)
        }
    }
}
```

---

## 🖱️ 第五部分：处理列表项点击事件

RecyclerView **没有内置的 `setOnItemClickListener`**，需要我们自己实现。

### 5.1 方法一：在 Adapter 中处理（推荐）

```kotlin
class ContactAdapter(
    private val contacts: List<Contact>,
    private val onItemClick: (Contact) -> Unit  // 点击回调
) : RecyclerView.Adapter<ContactViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ContactViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_contact, parent, false)
        return ContactViewHolder(view)
    }

    override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
        val contact = contacts[position]
        holder.nameText.text = contact.name
        holder.phoneText.text = contact.phone
        holder.avatarText.text = contact.name.first().toString()

        // ── 设置点击事件 ──
        holder.itemView.setOnClickListener {
            onItemClick(contact)  // 触发回调
        }
    }

    override fun getItemCount(): Int = contacts.size
}
```

**Activity 中使用**:

```kotlin
val adapter = ContactAdapter(contacts) { contact ->
    // 点击事件处理
    Toast.makeText(this, "点击了 ${contact.name}", Toast.LENGTH_SHORT).show()
    
    // 跳转到详情页
    val intent = Intent(this, DetailActivity::class.java)
    intent.putExtra("name", contact.name)
    intent.putExtra("phone", contact.phone)
    startActivity(intent)
}

recyclerView.adapter = adapter
```

### 5.2 方法二：在 ViewHolder 中处理

```kotlin
class ContactViewHolder(
    itemView: View,
    private val onItemClick: (Int) -> Unit  // position 回调
) : RecyclerView.ViewHolder(itemView) {

    val nameText: TextView = itemView.findViewById(R.id.nameText)
    val phoneText: TextView = itemView.findViewById(R.id.phoneText)
    val avatarText: TextView = itemView.findViewById(R.id.avatarText)

    init {
        itemView.setOnClickListener {
            onItemClick(adapterPosition)  // 传递位置
        }
    }
}
```

### 5.3 长按事件

```kotlin
override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
    val contact = contacts[position]
    // ... 数据绑定 ...

    // 长按事件
    holder.itemView.setOnLongClickListener {
        AlertDialog.Builder(holder.itemView.context)
            .setTitle("删除联系人")
            .setMessage("确定要删除 ${contact.name}？")
            .setPositiveButton("删除") { dialog, _ ->
                // 删除逻辑
                dialog.dismiss()
            }
            .setNegativeButton("取消") { dialog, _ -> dialog.dismiss() }
            .show()
        true  // 返回 true 表示事件已消费
    }
}
```

### 5.4 点击列表项内的特定控件

```kotlin
override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
    val contact = contacts[position]
    holder.nameText.text = contact.name
    holder.phoneText.text = contact.phone

    // 点击整个列表项
    holder.itemView.setOnClickListener {
        Toast.makeText(it.context, "点击了整个项", Toast.LENGTH_SHORT).show()
    }

    // 点击电话号码区域 → 拨打电话
    holder.phoneText.setOnClickListener {
        val intent = Intent(Intent.ACTION_DIAL, Uri.parse("tel:${contact.phone}"))
        it.context.startActivity(intent)
    }

    // 点击头像 → 显示大图
    holder.avatarText.setOnClickListener {
        Toast.makeText(it.context, "点击了头像", Toast.LENGTH_SHORT).show()
    }
}
```

---

## 🧪 第六部分：实验

### 6.1 实验一：创建联系人列表

**目标**: 创建一个联系人列表应用，支持显示联系人信息，点击跳转到详情页。

#### 数据类

```kotlin
data class Contact(
    val id: Int,
    val name: String,
    val phone: String,
    val email: String = "",
    val avatarColor: Int = 0xFF028090.toInt()  // 头像背景色
)
```

#### 列表项布局 (item_contact.xml)

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="2dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="16dp"
        android:background="?attr/selectableItemBackground">

        <!-- 圆形头像 -->
        <TextView
            android:id="@+id/avatarText"
            android:layout_width="56dp"
            android:layout_height="56dp"
            android:gravity="center"
            android:text="A"
            android:textSize="24sp"
            android:textStyle="bold"
            android:textColor="@android:color/white"
            android:background="@drawable/circle_bg" />

        <!-- 信息区域 -->
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical"
            android:layout_marginStart="16dp"
            android:layout_gravity="center_vertical">

            <TextView
                android:id="@+id/nameText"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="张三"
                android:textSize="18sp"
                android:textStyle="bold"
                android:textColor="#212121" />

            <TextView
                android:id="@+id/phoneText"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="138-0000-0000"
                android:textSize="14sp"
                android:textColor="#757575"
                android:layout_marginTop="4dp" />

            <TextView
                android:id="@+id/emailText"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="zhangsan@example.com"
                android:textSize="13sp"
                android:textColor="#9E9E9E"
                android:layout_marginTop="2dp" />

        </LinearLayout>

        <!-- 右箭头 -->
        <ImageView
            android:layout_width="24dp"
            android:layout_height="24dp"
            android:src="@android:drawable/ic_menu_more"
            android:tint="#BDBDBD"
            android:layout_gravity="center_vertical" />

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

#### ViewHolder

```kotlin
class ContactViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    val avatarText: TextView = itemView.findViewById(R.id.avatarText)
    val nameText: TextView = itemView.findViewById(R.id.nameText)
    val phoneText: TextView = itemView.findViewById(R.id.phoneText)
    val emailText: TextView = itemView.findViewById(R.id.emailText)
}
```

#### Adapter

```kotlin
class ContactAdapter(
    private val contacts: List<Contact>,
    private val onItemClick: (Contact) -> Unit
) : RecyclerView.Adapter<ContactViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ContactViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_contact, parent, false)
        return ContactViewHolder(view)
    }

    override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
        val contact = contacts[position]
        
        // 填充数据
        holder.nameText.text = contact.name
        holder.phoneText.text = contact.phone
        holder.emailText.text = contact.email
        holder.avatarText.text = contact.name.first().toString()
        
        // 设置头像背景色
        val drawable = holder.avatarText.background as? GradientDrawable
        drawable?.setColor(contact.avatarColor)

        // 点击事件
        holder.itemView.setOnClickListener {
            onItemClick(contact)
        }
    }

    override fun getItemCount(): Int = contacts.size
}
```

#### MainActivity

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val recyclerView = findViewById<RecyclerView>(R.id.recyclerView)

        // 准备测试数据
        val contacts = listOf(
            Contact(1, "张三", "138-0000-0001", "zhangsan@example.com", 0xFF028090.toInt()),
            Contact(2, "李四", "138-0000-0002", "lisi@example.com", 0xFF4CAF50.toInt()),
            Contact(3, "王五", "138-0000-0003", "wangwu@example.com", 0xFFFF9800.toInt()),
            Contact(4, "赵六", "138-0000-0004", "zhaoliu@example.com", 0xFFE91E63.toInt()),
            Contact(5, "钱七", "138-0000-0005", "qianqi@example.com", 0xFF9C27B0.toInt())
        )

        // 设置 RecyclerView
        recyclerView.layoutManager = LinearLayoutManager(this)
        recyclerView.adapter = ContactAdapter(contacts) { contact ->
            // 点击跳转详情页
            val intent = Intent(this, ContactDetailActivity::class.java)
            intent.putExtra("contact_id", contact.id)
            intent.putExtra("contact_name", contact.name)
            intent.putExtra("contact_phone", contact.phone)
            intent.putExtra("contact_email", contact.email)
            startActivity(intent)
        }
    }
}
```

#### ContactDetailActivity — 详情页

```kotlin
class ContactDetailActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_contact_detail)

        // 接收数据
        val name = intent.getStringExtra("contact_name") ?: ""
        val phone = intent.getStringExtra("contact_phone") ?: ""
        val email = intent.getStringExtra("contact_email") ?: ""

        // 显示数据
        findViewById<TextView>(R.id.detailName).text = name
        findViewById<TextView>(R.id.detailPhone).text = phone
        findViewById<TextView>(R.id.detailEmail).text = email

        // 返回按钮
        findViewById<Button>(R.id.btnBack).setOnClickListener {
            finish()
        }

        // 拨打电话按钮
        findViewById<Button>(R.id.btnCall).setOnClickListener {
            val intent = Intent(Intent.ACTION_DIAL, Uri.parse("tel:$phone"))
            startActivity(intent)
        }
    }
}
```

#### activity_contact_detail.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="联系人详情"
        android:textSize="24sp"
        android:textStyle="bold"
        android:textColor="#028090"
        android:layout_marginBottom="32dp" />

    <!-- 姓名 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="姓名"
        android:textSize="14sp"
        android:textColor="#757575"
        android:layout_marginBottom="4dp" />

    <TextView
        android:id="@+id/detailName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text=""
        android:textSize="18sp"
        android:textColor="#212121"
        android:layout_marginBottom="24dp" />

    <!-- 电话 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="电话"
        android:textSize="14sp"
        android:textColor="#757575"
        android:layout_marginBottom="4dp" />

    <TextView
        android:id="@+id/detailPhone"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text=""
        android:textSize="18sp"
        android:textColor="#212121"
        android:layout_marginBottom="24dp" />

    <!-- 邮箱 -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="邮箱"
        android:textSize="14sp"
        android:textColor="#757575"
        android:layout_marginBottom="4dp" />

    <TextView
        android:id="@+id/detailEmail"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text=""
        android:textSize="18sp"
        android:textColor="#212121"
        android:layout_marginBottom="32dp" />

    <!-- 按钮区域 -->
    <Button
        android:id="@+id/btnCall"
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:text="📞 拨打电话"
        android:textSize="16sp" />

    <Button
        android:id="@+id/btnBack"
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:text="返回"
        android:textSize="16sp"
        android:layout_marginTop="8dp"
        style="@style/Widget.Material3.Button.OutlinedButton" />

</LinearLayout>
```

---

### 6.2 实验二：添加分隔线

RecyclerView 默认没有分隔线，需要手动添加。

#### 方法一：使用 DividerItemDecoration

```kotlin
val recyclerView = findViewById<RecyclerView>(R.id.recyclerView)
recyclerView.layoutManager = LinearLayoutManager(this)

// 添加默认分隔线
val divider = DividerItemDecoration(this, DividerItemDecoration.VERTICAL)
recyclerView.addItemDecoration(divider)

recyclerView.adapter = ContactAdapter(contacts) { ... }
```

#### 方法二：自定义分隔线

**drawable/divider_line.xml**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="#E0E0E0" />
    <size android:height="1dp" />
</shape>
```

**使用**:

```kotlin
val divider = DividerItemDecoration(this, DividerItemDecoration.VERTICAL)
divider.setDrawable(ContextCompat.getDrawable(this, R.drawable.divider_line)!!)
recyclerView.addItemDecoration(divider)
```

---

## 📝 第七部分：课后作业 — 新闻列表应用界面

### 7.1 应用概述

开发一个新闻列表应用，显示新闻标题、摘要、图片、时间，点击可查看详情。

#### 功能要求

**必需功能**:

- ✅ 新闻列表：显示标题、摘要、配图、发布时间
- ✅ 点击列表项：跳转到新闻详情页
- ✅ 详情页：显示完整标题、正文、图片、时间
- ✅ 列表项使用 CardView 美化
- ✅ 至少准备 10 条测试数据

**加分功能**（可选）:

- ✨ 支持下拉刷新
- ✨ 新闻分类（科技、体育、娱乐等）
- ✨ 搜索功能
- ✨ 收藏功能（用 SharedPreferences 保存）
- ✨ 网格布局切换

### 7.2 数据设计

```kotlin
data class News(
    val id: Int,
    val title: String,
    val summary: String,
    val content: String,
    val imageUrl: String,     // 本地测试可用 drawable 资源
    val publishTime: String,
    val category: String = "综合"
)
```

### 7.3 项目结构

```
app/src/main/
├── java/com.example.newsapp/
│   ├── MainActivity.kt
│   ├── NewsDetailActivity.kt
│   ├── NewsAdapter.kt
│   ├── NewsViewHolder.kt
│   └── News.kt
├── res/layout/
│   ├── activity_main.xml
│   ├── activity_news_detail.xml
│   └── item_news.xml
└── res/drawable/
    └── (新闻配图)
```

### 7.4 完整参考代码

#### News.kt

```kotlin
package com.example.newsapp

data class News(
    val id: Int,
    val title: String,
    val summary: String,
    val content: String,
    val imageResId: Int,      // 图片资源 ID
    val publishTime: String,
    val category: String = "综合"
)
```

#### item_news.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="12dp"
        android:background="?attr/selectableItemBackground">

        <!-- 新闻图片 -->
        <ImageView
            android:id="@+id/newsImage"
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:scaleType="centerCrop"
            android:src="@drawable/news_placeholder"
            android:contentDescription="新闻配图" />

        <!-- 右侧信息区 -->
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical"
            android:layout_marginStart="12dp"
            android:layout_gravity="center_vertical">

            <!-- 标题 -->
            <TextView
                android:id="@+id/newsTitle"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="新闻标题"
                android:textSize="16sp"
                android:textStyle="bold"
                android:textColor="#212121"
                android:maxLines="2"
                android:ellipsize="end" />

            <!-- 摘要 -->
            <TextView
                android:id="@+id/newsSummary"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="新闻摘要内容…"
                android:textSize="14sp"
                android:textColor="#757575"
                android:maxLines="2"
                android:ellipsize="end"
                android:layout_marginTop="6dp" />

            <!-- 底部信息栏 -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:layout_marginTop="8dp">

                <TextView
                    android:id="@+id/newsCategory"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="科技"
                    android:textSize="12sp"
                    android:textColor="@android:color/white"
                    android:background="#028090"
                    android:paddingHorizontal="8dp"
                    android:paddingVertical="2dp" />

                <View
                    android:layout_width="0dp"
                    android:layout_height="1dp"
                    android:layout_weight="1" />

                <TextView
                    android:id="@+id/newsTime"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="2小时前"
                    android:textSize="12sp"
                    android:textColor="#9E9E9E" />

            </LinearLayout>

        </LinearLayout>

    </LinearLayout>

</androidx.cardview.widget.CardView>
```

#### NewsViewHolder.kt

```kotlin
package com.example.newsapp

import android.view.View
import android.widget.ImageView
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class NewsViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    val newsImage: ImageView = itemView.findViewById(R.id.newsImage)
    val newsTitle: TextView = itemView.findViewById(R.id.newsTitle)
    val newsSummary: TextView = itemView.findViewById(R.id.newsSummary)
    val newsCategory: TextView = itemView.findViewById(R.id.newsCategory)
    val newsTime: TextView = itemView.findViewById(R.id.newsTime)
}
```

#### NewsAdapter.kt

```kotlin
package com.example.newsapp

import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView

class NewsAdapter(
    private val newsList: List<News>,
    private val onItemClick: (News) -> Unit
) : RecyclerView.Adapter<NewsViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): NewsViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_news, parent, false)
        return NewsViewHolder(view)
    }

    override fun onBindViewHolder(holder: NewsViewHolder, position: Int) {
        val news = newsList[position]
        
        holder.newsTitle.text = news.title
        holder.newsSummary.text = news.summary
        holder.newsCategory.text = news.category
        holder.newsTime.text = news.publishTime
        holder.newsImage.setImageResource(news.imageResId)

        // 点击事件
        holder.itemView.setOnClickListener {
            onItemClick(news)
        }
    }

    override fun getItemCount(): Int = newsList.size
}
```

#### MainActivity.kt

```kotlin
package com.example.newsapp

import android.content.Intent
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val recyclerView = findViewById<RecyclerView>(R.id.recyclerView)

        // 测试数据
        val newsList = listOf(
            News(1, "AI技术迎来重大突破", "研究团队宣布在自然语言处理领域取得突破性进展", 
                "详细内容...", R.drawable.news1, "2小时前", "科技"),
            News(2, "新能源汽车销量创新高", "第一季度新能源汽车销量同比增长40%",
                "详细内容...", R.drawable.news2, "3小时前", "汽车"),
            News(3, "航天器成功着陆火星", "探测器成功着陆，开始科学考察任务",
                "详细内容...", R.drawable.news3, "5小时前", "航天"),
            News(4, "经济复苏势头良好", "多项经济指标显示复苏态势明显",
                "详细内容...", R.drawable.news4, "6小时前", "财经"),
            News(5, "教育改革方案发布", "新的教育改革方案今日正式发布",
                "详细内容...", R.drawable.news5, "8小时前", "教育"),
            News(6, "气候变化应对会议召开", "全球气候变化应对峰会在京召开",
                "详细内容...", R.drawable.news6, "10小时前", "环境"),
            News(7, "医疗技术新进展", "新型治疗方法临床试验取得成功",
                "详细内容...", R.drawable.news7, "12小时前", "医疗"),
            News(8, "体育赛事精彩纷呈", "多项体育赛事同时举行，精彩纷呈",
                "详细内容...", R.drawable.news8, "1天前", "体育"),
            News(9, "文化活动丰富多彩", "各地举办丰富多彩的文化活动",
                "详细内容...", R.drawable.news9, "1天前", "文化"),
            News(10, "旅游业逐步回暖", "旅游市场呈现回暖趋势，游客数量增加",
                "详细内容...", R.drawable.news10, "2天前", "旅游")
        )

        recyclerView.layoutManager = LinearLayoutManager(this)
        recyclerView.adapter = NewsAdapter(newsList) { news ->
            // 跳转详情页
            val intent = Intent(this, NewsDetailActivity::class.java)
            intent.putExtra("news_id", news.id)
            intent.putExtra("news_title", news.title)
            intent.putExtra("news_content", news.content)
            intent.putExtra("news_image", news.imageResId)
            intent.putExtra("news_time", news.publishTime)
            intent.putExtra("news_category", news.category)
            startActivity(intent)
        }
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

    <!-- 顶部栏 -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:orientation="horizontal"
        android:gravity="center_vertical"
        android:background="#028090"
        android:paddingHorizontal="16dp">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="📰 新闻"
            android:textSize="22sp"
            android:textStyle="bold"
            android:textColor="@android:color/white" />

    </LinearLayout>

    <!-- 新闻列表 -->
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#F5F5F5" />

</LinearLayout>
```

#### NewsDetailActivity.kt

```kotlin
package com.example.newsapp

import android.os.Bundle
import android.widget.Button
import android.widget.ImageView
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity

class NewsDetailActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_news_detail)

        val title = intent.getStringExtra("news_title") ?: ""
        val content = intent.getStringExtra("news_content") ?: ""
        val imageResId = intent.getIntExtra("news_image", 0)
        val time = intent.getStringExtra("news_time") ?: ""
        val category = intent.getStringExtra("news_category") ?: ""

        findViewById<TextView>(R.id.detailTitle).text = title
        findViewById<TextView>(R.id.detailContent).text = content
        findViewById<TextView>(R.id.detailTime).text = "$category  •  $time"
        if (imageResId != 0) {
            findViewById<ImageView>(R.id.detailImage).setImageResource(imageResId)
        }

        findViewById<Button>(R.id.btnBack).setOnClickListener {
            finish()
        }
    }
}
```

#### activity_news_detail.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <!-- 新闻图片 -->
        <ImageView
            android:id="@+id/detailImage"
            android:layout_width="match_parent"
            android:layout_height="240dp"
            android:scaleType="centerCrop"
            android:src="@drawable/news_placeholder"
            android:contentDescription="新闻配图" />

        <!-- 内容区域 -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">

            <!-- 标题 -->
            <TextView
                android:id="@+id/detailTitle"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text=""
                android:textSize="22sp"
                android:textStyle="bold"
                android:textColor="#212121"
                android:layout_marginBottom="8dp" />

            <!-- 时间和分类 -->
            <TextView
                android:id="@+id/detailTime"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text=""
                android:textSize="13sp"
                android:textColor="#9E9E9E"
                android:layout_marginBottom="16dp" />

            <!-- 分隔线 -->
            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:background="#E0E0E0"
                android:layout_marginBottom="16dp" />

            <!-- 正文 -->
            <TextView
                android:id="@+id/detailContent"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text=""
                android:textSize="16sp"
                android:textColor="#424242"
                android:lineSpacingExtra="4dp" />

        </LinearLayout>

        <!-- 返回按钮 -->
        <Button
            android:id="@+id/btnBack"
            android:layout_width="match_parent"
            android:layout_height="48dp"
            android:text="返回"
            android:textSize="16sp"
            android:layout_margin="16dp" />

    </LinearLayout>

</ScrollView>
```

### 7.5 评分标准

|项目|分值|评分细则|
|---|---|---|
|列表显示|30%|RecyclerView 正确显示新闻列表，数据完整|
|列表项布局|20%|使用 CardView，布局美观，信息层次清晰|
|点击跳转|20%|点击列表项能正确跳转并传递数据|
|详情页显示|20%|详情页完整显示标题、图片、正文、时间|
|代码质量|10%|Adapter、ViewHolder 实现规范，命名清晰|

#### 加分项（可选，每项+5分，上限+15分）

- ✨ 实现新闻分类筛选功能
- ✨ 支持下拉刷新
- ✨ 添加收藏功能并持久化
- ✨ 支持网格布局切换
- ✨ 列表项添加点击动画效果

#### 提交要求

1. **源代码** — 完整 Android 项目（ZIP 压缩包）
2. **运行截图** — 至少4张（列表页、详情页、点击效果、特殊功能）
3. **说明文档** — README.md（功能介绍、实现思路、遇到的问题和解决方法）

**文件命名**: `姓名_学号_Week7_新闻列表.zip`  
**截止时间**: 第8周周一上课前

---

## 💡 第八部分：最佳实践与常见坑

### 8.1 RecyclerView 常见坑

```kotlin
// ─── 坑①: 忘记设置 LayoutManager ───
recyclerView.adapter = adapter   // ❌ 还没设置 LayoutManager，不显示

// ✅ 正确顺序
recyclerView.layoutManager = LinearLayoutManager(this)  // 先设置
recyclerView.adapter = adapter                          // 后设置

// ─── 坑②: 数据变化后忘记刷新 ───
val list = mutableListOf<Contact>(...)
val adapter = ContactAdapter(list)
recyclerView.adapter = adapter

// 后来添加了数据
list.add(Contact("新联系人", "123"))
// ❌ 界面不会更新！

// ✅ 需要通知 Adapter
list.add(Contact("新联系人", "123"))
adapter.notifyDataSetChanged()     // 刷新整个列表
// 或者
adapter.notifyItemInserted(list.size - 1)  // 只刷新新增项（更高效）

// ─── 坑③: 在 onBindViewHolder 中做耗时操作 ───
override fun onBindViewHolder(holder: ViewHolder, position: Int) {
    val data = dataList[position]
    
    // ❌ 加载网络图片（会卡顿）
    val bitmap = downloadImage(data.imageUrl)  // 耗时操作
    holder.imageView.setImageBitmap(bitmap)
    
    // ✅ 应该用图片加载库（如 Glide、Coil）
    Glide.with(holder.itemView.context)
        .load(data.imageUrl)
        .into(holder.imageView)
}
```

### 8.2 Adapter 性能优化技巧

```kotlin
// ─── 技巧①: 使用 ViewType 优化多类型列表 ───
override fun getItemViewType(position: Int): Int {
    return when (items[position]) {
        is HeaderItem -> TYPE_HEADER
        is ContentItem -> TYPE_CONTENT
        is FooterItem -> TYPE_FOOTER
        else -> TYPE_CONTENT
    }
}

override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
    return when (viewType) {
        TYPE_HEADER -> HeaderViewHolder(...)
        TYPE_CONTENT -> ContentViewHolder(...)
        TYPE_FOOTER -> FooterViewHolder(...)
        else -> ContentViewHolder(...)
    }
}

// ─── 技巧②: 避免在 onBindViewHolder 中创建对象 ───
// ❌ 每次滚动都创建新对象
override fun onBindViewHolder(holder: ViewHolder, position: Int) {
    holder.itemView.setOnClickListener {  // 每次都创建新 Listener
        onClick(position)
    }
}

// ✅ 在 ViewHolder 中创建一次
class MyViewHolder(itemView: View, val onClick: (Int) -> Unit) : RecyclerView.ViewHolder(itemView) {
    init {
        itemView.setOnClickListener {
            onClick(adapterPosition)
        }
    }
}

// ─── 技巧③: 使用 DiffUtil 精确刷新 ───
// 当数据变化时，DiffUtil 会计算差异，只刷新变化的项
class NewsDiffCallback(
    private val oldList: List<News>,
    private val newList: List<News>
) : DiffUtil.Callback() {
    
    override fun getOldListSize() = oldList.size
    override fun getNewListSize() = newList.size
    
    override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldList[oldItemPosition].id == newList[newItemPosition].id
    }
    
    override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldList[oldItemPosition] == newList[newItemPosition]
    }
}

// 使用
val diffResult = DiffUtil.calculateDiff(NewsDiffCallback(oldList, newList))
diffResult.dispatchUpdatesTo(adapter)
```

### 8.3 布局优化建议

```xml
<!-- ❌ 嵌套过深（性能差） -->
<LinearLayout>
    <LinearLayout>
        <LinearLayout>
            <TextView />
        </LinearLayout>
    </LinearLayout>
</LinearLayout>

<!-- ✅ 扁平化布局（推荐用 ConstraintLayout） -->
<androidx.constraintlayout.widget.ConstraintLayout>
    <TextView 
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

### 8.4 内存泄漏预防

```kotlin
// ❌ Adapter 持有 Activity 引用
class MyAdapter(private val activity: Activity) : RecyclerView.Adapter<...>() {
    // 如果 RecyclerView 被长时间持有，Activity 无法释放 → 内存泄漏
}

// ✅ 使用回调接口
class MyAdapter(private val onItemClick: (Item) -> Unit) : RecyclerView.Adapter<...>() {
    // 只持有函数引用，不持有 Activity
}

// Activity 中
val adapter = MyAdapter { item ->
    // 处理点击
}
```

---

## 🔄 第九部分：RecyclerView 进阶技巧

### 9.1 添加列表动画

```kotlin
// RecyclerView 自带动画，只需调用通知方法即可
adapter.notifyItemInserted(position)      // 插入动画
adapter.notifyItemRemoved(position)       // 删除动画
adapter.notifyItemChanged(position)       // 变化动画
adapter.notifyItemMoved(fromPos, toPos)   // 移动动画
```

### 9.2 实现下拉刷新

```xml
<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
    android:id="@+id/swipeRefresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
```

```kotlin
val swipeRefresh = findViewById<SwipeRefreshLayout>(R.id.swipeRefresh)

swipeRefresh.setOnRefreshListener {
    // 刷新数据
    loadData()
    
    // 刷新完成后停止动画
    swipeRefresh.isRefreshing = false
}
```

### 9.3 实现滑动删除

```kotlin
val itemTouchHelper = ItemTouchHelper(object : ItemTouchHelper.SimpleCallback(
    0,  // 拖拽方向（0 表示不支持）
    ItemTouchHelper.LEFT or ItemTouchHelper.RIGHT  // 滑动方向
) {
    override fun onMove(...): Boolean = false  // 不处理拖拽

    override fun onSwiped(viewHolder: RecyclerView.ViewHolder, direction: Int) {
        val position = viewHolder.adapterPosition
        
        // 删除数据
        dataList.removeAt(position)
        adapter.notifyItemRemoved(position)
        
        Toast.makeText(this@MainActivity, "已删除", Toast.LENGTH_SHORT).show()
    }
})

itemTouchHelper.attachToRecyclerView(recyclerView)
```

### 9.4 实现加载更多

```kotlin
recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
    override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
        super.onScrolled(recyclerView, dx, dy)
        
        val layoutManager = recyclerView.layoutManager as LinearLayoutManager
        val visibleItemCount = layoutManager.childCount
        val totalItemCount = layoutManager.itemCount
        val firstVisibleItemPosition = layoutManager.findFirstVisibleItemPosition()
        
        // 判断是否滑到底部
        if (visibleItemCount + firstVisibleItemPosition >= totalItemCount 
            && firstVisibleItemPosition >= 0) {
            // 加载更多数据
            loadMoreData()
        }
    }
})
```

### 9.5 空状态处理

```kotlin
fun updateList(newData: List<Item>) {
    dataList.clear()
    dataList.addAll(newData)
    adapter.notifyDataSetChanged()
    
    // 显示/隐藏空状态视图
    if (dataList.isEmpty()) {
        findViewById<View>(R.id.emptyView).visibility = View.VISIBLE
        recyclerView.visibility = View.GONE
    } else {
        findViewById<View>(R.id.emptyView).visibility = View.GONE
        recyclerView.visibility = View.VISIBLE
    }
}
```

---

## 🎯 课后思考题

1. **RecyclerView 相比 ListView 有哪些优势？为什么 Google 推荐使用 RecyclerView？**
    
2. **ViewHolder 的作用是什么？如果不用 ViewHolder 会有什么问题？**
    
3. **Adapter 的三大核心方法分别在什么时候被调用？onCreateViewHolder 和 onBindViewHolder 的调用频率有何不同？**
    
4. **如果列表有 1000 条数据，屏幕只能显示 10 条，onCreateViewHolder 会被调用多少次？为什么？**
    
5. **你在开发新闻列表应用时遇到了什么问题？是如何解决的？（写入 README.md）**
    

---

## 📚 扩展阅读

### 官方文档

- [RecyclerView 开发指南](https://developer.android.com/develop/ui/views/layout/recyclerview)
- [创建列表和卡片](https://developer.android.com/develop/ui/views/layout/recyclerview-custom)
- [Adapter 最佳实践](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.Adapter)

### 推荐文章

- [RecyclerView Performance Best Practices](https://developer.android.com/topic/performance/rendering/optimizing-view-hierarchies)
- [DiffUtil 使用指南](https://developer.android.com/reference/androidx/recyclerview/widget/DiffUtil)
- [Material Design List Guidelines](https://material.io/components/lists)

---

## 🎓 下周预告

**第8周：Fragment 基础**

- Fragment 概念与生命周期
- Fragment 事务管理（添加、替换、移除）
- Activity 与 Fragment 之间的通信
- Fragment 参数传递（Bundle、接口回调）
- 实践：创建可重用的 Fragment，实现 Fragment 切换

**预习建议**:

- 复习本周的 RecyclerView 知识（Fragment 中也会用到列表）
- 思考：一个 Activity 能包含多个界面吗？如何实现？
- 了解"模块化"、"组件化"的概念

---

**课程资料更新时间**: 2026年2月  
**讲师**: [教师姓名]  
**联系方式**: [邮箱]

_RecyclerView 是 Android 开发的核心技能！理解 Adapter 模式和 ViewHolder 机制对后续学习至关重要。建议多练习，把新闻列表做得更完善。加油！🚀_