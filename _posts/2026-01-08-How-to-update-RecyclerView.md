---
layout: post
title: RecyclerView 的正确更新方式
date: 2026-01-08 17:00:00 +0800
categories: android
tags: RecyclerView
published: true
---

* content
{:toc}

## 前言

RecyclerView 一直是 Android 中最常用的控件，但如何正确优雅的更新列表，感觉很多人没有真正弄懂过，在项目里常常能见到各种稀奇古怪的更新方式。下面是一种通用优雅的列表更新方式。

## xml

让我们从一个简单列表开始，假设要显示如下 ItemView 的列表：

![item view 预览]({{ '/styles/images/list/item_view.jpg' | prepend: site.baseurl  }})  

左边依次是用户头像、用户名字，右边是一个勾选框，对应的布局如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="48dp"
    android:paddingHorizontal="16dp">

    <com.google.android.material.imageview.ShapeableImageView
        android:id="@+id/avatar"
        android:layout_width="32dp"
        android:layout_height="32dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:shapeAppearance="@style/CircleStyle"
        tools:ignore="ContentDescription"
        tools:src="@drawable/ic_default_avatar" />

    <TextView
        android:id="@+id/name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginHorizontal="8dp"
        android:ellipsize="end"
        android:gravity="center_vertical"
        android:lines="1"
        android:maxLines="1"
        android:textColor="@android:color/black"
        app:layout_constrainedWidth="true"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@id/select"
        app:layout_constraintHorizontal_bias="0"
        app:layout_constraintStart_toEndOf="@id/avatar"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="name" />

    <ImageView
        android:id="@+id/select"
        android:layout_width="24dp"
        android:layout_height="24dp"
        android:padding="4dp"
        android:src="@drawable/checkbox_selector"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:ignore="ContentDescription"
        tools:src="@drawable/ic_check" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

需要注意的是，勾选框不要使用 CheckBox，而是 ImageView + selector drawable，数据驱动 UI，UI 层仅做渲染，不存储状态。

## Model

对应的数据模型：

```kotlin
data class ItemModel(
    val uid: Long,
    val name: String,
    val avatar: String,
    val selected: Boolean = false,  // 本地状态，记录 item 是否被选中
)
```

数据模型使用 data class，且属性都使用 val 不可变，更新属性的时候使用 `itemModel.copy(xx = xxx)` 创建新对象。

这里比较有争议的大概就是本地状态 selected 了，和其他来于服务端接口的属性不同，这个属性只是 UI 层的临时状态。一种方式是新建一个 UI 层级的 state 包裹真实的数据 model 以及本地状态，
但是这样一来会增加很多繁琐的 boilerplate code，个人觉得意义不大，有点为了封装而封装，当然本地状态很多的复杂情况另当别论，UI 状态复杂、可复用性高时，仍建议引入 UIState 层。

## ViewModel

ViewModel 里做两件事儿：mock 列表数据以及反选

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch

class ListViewModel : ViewModel() {

    private val _items = MutableStateFlow<List<ItemModel>>(emptyList())
    val items = _items.asStateFlow()

    fun loadData() {
        viewModelScope.launch {
            delay(500)
            _items.value = mock()
        }
    }

    /**
     * @param model - 反选的 item 对象
     * @param singleSelectionMode - 是否是单选模式
     */
    fun onSelectChanged(model: ItemModel, singleSelectionMode: Boolean = false) {
        val newSelected = !model.selected
        _items.update { list ->
            list.map {
                when {
                    it.uid == model.uid -> it.copy(selected = newSelected)
                    singleSelectionMode && newSelected && it.selected -> it.copy(selected = false)
                    else -> it
                }
            }
        }
    }

    private fun mock(): List<ItemModel> {
        return List(30) { i ->
            ItemModel(
                uid = i.toLong(),
                name = "name_$i",
                avatar = "https://p2.music.126.net/3DMKf25Gs_86P3m3TYD7Pg==/109951169063396093.jpg",
                selected = false
            )
        }
    }
}
```

## Activity

```kotlin
import android.os.Bundle
import android.view.View
import androidx.activity.viewModels
import androidx.lifecycle.lifecycleScope
import androidx.recyclerview.widget.DividerItemDecoration
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.fiture.ui.base.BaseActivity
import kotlinx.coroutines.flow.collectLatest
import kotlinx.coroutines.launch

class ListActivity : BaseActivity() {

    private val viewModel: ListViewModel by viewModels<ListViewModel>()

    private val itemAdapter: ItemAdapter = ItemAdapter { event ->
        when (event) {
            is ItemEvent.SelectChanged -> viewModel.onSelectChanged(event.model)
        }
    }

    private val recyclerView: RecyclerView by lazy {
        RecyclerView(this).apply {
            overScrollMode = View.OVER_SCROLL_NEVER
            layoutManager = LinearLayoutManager(this@ListActivity)
            adapter = itemAdapter
            val divider = DividerItemDecoration(context, LinearLayoutManager.VERTICAL)
            addItemDecoration(divider)
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(recyclerView)
        observe()
        load()
    }

    private fun observe() {
        lifecycleScope.launch {
            viewModel.items.collectLatest {
                loading.dismiss()
                itemAdapter.submitList(it)
            }
        }
    }

    private fun load() {
        loading.show()
        viewModel.loadData()
    }
}
```

Activity 只是继承了我自己封装的一个 BaseActivity，用到了里边的 loading 相关逻辑，其他与 AppCompatActivity 无异。

## Adapter

**ListAdapter**

```kotlin
import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.ListAdapter
import com.fiture.ui.databinding.ItemListBinding

class ItemAdapter(
    private val onEvent: (ItemEvent) -> Unit = {}
) : ListAdapter<ItemModel, ItemViewHolder>(ItemDiffCallback) {

    override fun onCreateViewHolder(
        parent: ViewGroup,
        viewType: Int
    ): ItemViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        val binding = ItemListBinding.inflate(inflater, parent, false)
        return ItemViewHolder(binding) { position ->
            onEvent(ItemEvent.SelectChanged(getItem(position)))
        }
    }

    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) {
        holder.bind(getItem(position), position)
    }

    override fun onBindViewHolder(
        holder: ItemViewHolder,
        position: Int,
        payloads: MutableList<Any>
    ) {
        if (payloads.isEmpty()) {
            onBindViewHolder(holder, position)
        } else {
            payloads.forEach { payload ->
                when (payload) {
                    is Payload.Select -> holder.bindSelected(payload.value)
                }
            }
        }
    }
}

sealed interface ItemEvent {
    data class SelectChanged(val model: ItemModel) : ItemEvent
}
```

**AsyncListDiffer**

如果 adapter 有继承其他类而无法继承 ListAdapter，可以改成如下方式（ListAdapter 内部实现其实也就是封装了一个 AsyncListDiffer）：

```kotlin
import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.AsyncListDiffer
import androidx.recyclerview.widget.RecyclerView
import com.fiture.ui.databinding.ItemListBinding

class ItemAdapter(
    private val onEvent: (ItemEvent) -> Unit = {}
) : RecyclerView.Adapter<ItemViewHolder>() {

    private val differ = AsyncListDiffer(this, ItemDiffCallback)

    override fun onCreateViewHolder(
        parent: ViewGroup,
        viewType: Int
    ): ItemViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        val binding = ItemListBinding.inflate(inflater, parent, false)
        return ItemViewHolder(binding) { position ->
            onEvent(ItemEvent.SelectChanged(getItem(position)))
        }
    }

    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) {
        holder.bind(getItem(position), position)
    }

    override fun onBindViewHolder(
        holder: ItemViewHolder,
        position: Int,
        payloads: MutableList<Any>
    ) {
        if (payloads.isEmpty()) {
            onBindViewHolder(holder, position)
        } else {
            payloads.forEach { payload ->
                when (payload) {
                    is Payload.Select -> holder.bindSelected(payload.value)
                }
            }
        }
    }

    override fun getItemCount(): Int = differ.currentList.size

    fun getItem(position: Int): ItemModel = differ.currentList[position]

    fun submitList(list: List<ItemModel>?) {
        differ.submitList(list)
    }
}

sealed interface ItemEvent {
    data class SelectChanged(val model: ItemModel) : ItemEvent
}
```

## ViewHolder

```kotlin
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import com.fiture.ui.databinding.ItemListBinding

class ItemViewHolder(
    val binding: ItemListBinding,
    val onSelectClick: (position: Int) -> Unit = {},
) : RecyclerView.ViewHolder(binding.root) {

    fun bind(model: ItemModel, position: Int) {
        Glide.with(binding.avatar.context).load(model.avatar).into(binding.avatar)
        binding.name.text = model.name
        bindSelected(model.selected)
        binding.select.setOnClickListener {
            val pos = bindingAdapterPosition
            if (pos != RecyclerView.NO_POSITION) {
                onSelectClick(pos)
            }
        }
    }

    fun bindSelected(selected: Boolean) {
        binding.select.isSelected = selected
    }
}
```

需要注意的是 ViewHolder bind 方法里的 model 仅代表 bind 那一时刻的数据状态，后续通过 payload 更新 UI 之后 model 的状态依然不会改变，所以点击事件之类的不能使用 model 里的状态。
当然，这里我们通过回调把点击事件丢出去给 Adapter 处理避免了这个问题。如果执意要在 ViewHolder 里边处理点击事件具体的业务逻辑，应该使用
`(bindingAdapter as ItemAdapter).getItem(bindingAdapterPosition)` 这种方式获取 model。如果 Adapter 是继承自 ListAdapter， `getItem` 方法是 protected，需要自行在 adapter 里再定义一个方法将其暴露出来使用。

## Diff

```kotlin
import androidx.recyclerview.widget.DiffUtil

object ItemDiffCallback : DiffUtil.ItemCallback<ItemModel>() {

    override fun areItemsTheSame(old: ItemModel, new: ItemModel): Boolean = old.uid == new.uid

    override fun areContentsTheSame(old: ItemModel, new: ItemModel): Boolean = old == new

    override fun getChangePayload(old: ItemModel, new: ItemModel): Any? {
        return if (old.selected != new.selected) { Payload.Select(new.selected) } else null
    }
}

sealed interface Payload {
    data class Select(val value: Boolean) : Payload
}
```

使用 AsyncListDiffer 的方式更新，只需要实现 DiffUtil.ItemCallback 即可，AsyncListDiffer 内部会创建 DiffUtil.Callback 间接调用 DiffUtil.ItemCallback。

## Diff vs adapter.notifyXXX

上面就是一个完整版的简单列表更新（复杂列表比如多类型 item 等自行做相应细节改动即可），使用 diff + payload 只对变化的 item 及控件进行最小化更新，相较于传统的 adapter.notifyXXX 来说节省性能、避免抖动。当然 adapter.notifyXXX 也有相应的 payload 方法，但是使用上没有 diff 这么灵活方便（比如点击页面某个按钮对列表做全选）

## Myers Diff Algorithm

上面的 Adapter 示例无论是继承 ListAdapter 还是手动创建 AsyncListDiffer，其本质都是使用 AsyncListDiffer，`AsyncListDiffer.submitList` 后内部使用 `DiffUtil.calculateDiff` 异步计算出 DiffResult，然后再调用 `DiffResult.dispatchUpdatesTo` 应用差分结果。

`DiffUtil.calculateDiff` 可以通过参数控制是否需要 detectMoves，AsyncListDiffer 默认 detectMoves 为 true。事实上，如果不需要侦测移动（绝大部分场景都不需要），差分算法运行效率更高，如果列表数据量足够大、变动够多，想要更高效率的差分计算，可以自行创建 `DiffUtil.Callback`，配合 `DiffUtil.calculateDiff(Callback cb, boolean detectMoves)` 传入 detectMoves false。不要忘了自行处理异步计算，毕竟数据量足够大的时候差分算法可能会比较耗时。99.99% 的情况下你都应该使用 AsyncListDiffer 的方式，它已经处理好了异步线程切换（后台算 diff，主线程 dispatch）和防止并发错乱（generation/version 校验）。

最后，聊一聊 DiffUtil 差分算法的底层实现。关键点在于 `DiffUtil.calculateDiff` 和 `DiffResult.dispatchUpdatesTo` 这两个方法，前者计算出差分结果，后者应用差分结果。
初看 DiffUtil 实现，如果对其使用算法不了解，会对 Diagonal、Snake 等关键词一脸懵逼，DiffUtil 用的算法本体是 Myers Diff Algorithm，作者 Eugene W. Myers。

算法复杂度：
* 最优情况 O(N)
* 一般情况 O((N + M)D)
* 最坏情况 O(N²)（极端差异）

这是目前：
* 最短编辑路径（Shortest Edit Script）
* 最少插入 / 删除操作
* Git、Unix diff、IDE diff 的核心算法

先看[论文](https://neil.fraser.name/writing/diff/myers.pdf){:target="_blank"}吧，算法细节以后有空再补^_^。

## Java

上面的代码针对 kotlin，如果项目混编有 java 代码应该如何写 ItemModel 呢？

Kotlin data class 到底帮你做了什么，先把“魔法”拆穿：

```kotlin
data class ItemModel(
    val uid: Long,
    val name: String,
    val avatar: String,
    val selected: Boolean
)
```

等价于 Java 里自动帮你生成了：
1.	equals() / hashCode()（字段级）
2.	toString()
3.	copy(...)（创建新对象）
4.	所有字段 final
5.	解构（Java 不关心）

RecyclerView + Diff 真正依赖的只有 1 + 3 + 不可变性。
Kotlin 的 data class 并不是 RecyclerView + Diff 的前提，
它只是把 Java 里本该显式写出的“不可变模型 + copy + equals”自动化了。
在 Java 中，只要遵守不可变 + 新对象更新的原则，Diff 的效果是完全等价的。

```java
import com.fiture.ui.base.Keepable;
import java.util.Objects;

public final class ItemModel implements Keepable {

    private final long uid;
    private final String name;
    private final String avatar;
    private final boolean selected;

    public ItemModel(long uid, String name, String avatar, boolean selected) {
        this.uid = uid;
        this.name = name;
        this.avatar = avatar;
        this.selected = selected;
    }

    public long getUid() { return uid; }
    public String getName() { return name; }
    public String getAvatar() { return avatar; }
    public boolean isSelected() { return selected; }

    public ItemModel copy(boolean selected) {
        return new ItemModel(
                uid,
                name,
                avatar,
                selected
        );
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ItemModel that)) return false;
        return uid == that.uid &&
                selected == that.selected &&
                Objects.equals(name, that.name) &&
                Objects.equals(avatar, that.avatar);
    }

    @Override
    public int hashCode() {
        return Objects.hash(uid, name, avatar, selected);
    }
}
```

如果使用 java 17 及以上，java 代码还可以使用 record class：

```java
import com.fiture.ui.base.Keepable;
import java.util.Objects;

public record ItemModel(long uid, String name, String avatar, boolean selected) implements Keepable {

    public ItemModel copy(boolean selected) {
        return new ItemModel(
                uid,
                name,
                avatar,
                selected
        );
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ItemModel that)) return false;
        return uid == that.uid &&
                selected == that.selected &&
                Objects.equals(name, that.name) &&
                Objects.equals(avatar, that.avatar);
    }
}
```

record 天然具备：
* ✅ final 字段（不可变）
* ✅ 自动生成 equals / hashCode / toString
* ✅ 构造函数
* ❌ 没有 copy()（但可手写）

在 Java 项目中，record class 是最接近 Kotlin data class 的等价物。
它天然不可变、自动生成 equals/hashCode，非常适合与 DiffUtil 配合使用。
唯一需要补充的是语义化的 copy 方法，用于生成新的状态对象。

<!-- https://zhuanlan.zhihu.com/p/34640058 -->
<!-- https://cloud.tencent.com/developer/article/2367282 -->
<!-- https://git-scm.com/docs/diff-options -->