---
layout: post
title: "Kotlin使用记录3-提取一个通用的分组Adapter"
categories: kotlin
date: 2017-03-31 12:12:00

---

这两天使用kotlin改造我们的Android项目的时候，发现有好几个模块都用到了一个类似的列表展现模式，分组展现。

如我们会议室列表页面：

![meeting room](http://muliba.u.qiniudn.com/kotlin_sealed_class_1.jpg)

还有论坛板块页面：

![bbs sections](http://muliba.u.qiniudn.com/blog/post/bbs_sections.png.jpg)

等等，类似的蛮多的。

每次这种业务，我都习惯性的写一个专门的Adapter，然后根据数据类型来创建不一样的LayoutView 。然后就发现这些Adapter其实都很类似，于是就想用kotlin重构下这些Adapter，用一个通用的Adapter替换，去掉这些重复代码。

这些数据类型都类似，有一个head对象，如上面会议室列表的**大楼**对象，论坛板块列表的**分区**。还有就是body对象，如具体的**会议室**对象，**板块**对象。于是我就定义了一个`Group`对象

```kotlin
data class Group<out T, out R>(val header: T, val children: List<R>)
```

定义一个抽象的Adapter，传入数据列表就是上面定义的`Group`数据列表，然后还有两个必需要的东西就是head和body的layout布局文件。

```kotlin
abstract class GroupRecyclerViewAdapter<T, R>(val groups:List<Group<T, R>>, val headerResourceId:Int, val childResourceId:Int) :
        RecyclerView.Adapter<CommonRecyclerViewHolder>()
```

`GroupRecyclerViewAdapter`继承自官方的`RecyclerViewAdapter`，关于里面的`CommonRecyclerViewHolder`这个类灵感来自网上一个Android大神的[Hongyang](https://github.com/hongyangAndroid/baseAdapter/blob/master/baseadapter-recyclerview/src/main/java/com/zhy/adapter/recyclerview/CommonAdapter.java)

主要代码：

```java

public CommonRecyclerViewHolder(View itemView) {
    super(itemView);
    this.convertView = itemView;
    this.views = new SparseArray<>();
    this.convertView.setTag(this);
}


public <T extends View> T getView(int viewId) {
    View view = views.get(viewId);
    if (view == null) {
        view = convertView.findViewById(viewId);
        views.put(viewId, view);
    }
    return (T) view;
}
public CommonRecyclerViewHolder setText(int viewId, String text) {
    TextView view = getView(viewId);
    view.setText(text);
    return this;
}
```

通过这个简单封装可以在Adapter中使用链式调用绑定view

```java
holder.setText(R.id.tv, "title")
                      .setImageViewResource(R.id.image, res);
```



然后说回我们的GroupRecyclerViewAdapter，因为有两种类型的layout布局，所以使用kotlin的companion对象定义两个类型的常量。

```kotlin
companion object {
        val ITEM_TYPE_HEAD = 0 //head
        val ITEM_TYPE_CHILD = 1 //body
    }
```

这样就可以重写`getItemViewType`来告诉Adapter需要用哪个layout布局创建View

```kotlin
override fun getItemViewType(position: Int): Int {
        return items[position]
    }
```

这里的items不是开始传入的groups，因为groups是分组的对象，不是RecyclerView的item，所以在重写`getItemCount`的时候做了点手脚

```kotlin
override fun getItemCount(): Int {
        var count:Int = 0
        groups.map {
            items.put(count, ITEM_TYPE_HEAD)
            count++
            it.children.map {
                items.put(count, ITEM_TYPE_CHILD)
                count++
            }
        }
        return count
    }
```

这样知道了position对应的类型，我们就可以创建ViewHolder了

```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CommonRecyclerViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        return when(viewType) {
            ITEM_TYPE_HEAD -> CommonRecyclerViewHolder(inflater.inflate(headerResourceId, parent, false))
            else -> CommonRecyclerViewHolder(inflater.inflate(childResourceId, parent, false))
        }
    }
```

我定义了两个抽象函数来处理在`onBindViewHolder`中不同的layout布局的情况

```kotlin
abstract fun onBindHeaderViewHolder(holder: CommonRecyclerViewHolder, header: T)

    abstract fun onBindChildViewHolder(holder: CommonRecyclerViewHolder, child: R)

```

这时候就有个问题，我们有groups，没有对应的某一个item的具体对象，`onBindViewHolder`怎么处理。我通过计算position在哪个Group的位置来确定当前需要给出的Item真正的对象。

```kotlin
var currentPosition = 0
groups.forEach {
    val childrenTotal = it.children.size
    when(position) {
        in currentPosition..(currentPosition+childrenTotal) -> {
            if (currentPosition == position) {
                onBindHeaderViewHolder(holder, it.header)
                return
            }else {
                val childIndex = position - currentPosition - 1
                onBindChildViewHolder(holder, it.children[childIndex])
                return
            }
        }
    }
    currentPosition += childrenTotal + 1
}
```

这里用到了`when`函数判断区间的能力，非常好用，省了很多事 :-) 

这样这个Adapter就完成了，使用起来很方便：

```kotlin
adapter = new GroupRecyclerViewAdapter<MeetingRoom.Building, MeetingRoom.Room>(roomList,
                R.layout.item_meeting_room_list_build, R.layout.item_meeting_room_list_room) {
                //implement onBindHeaderViewHolder & onBindChildViewHolder
}
```



对了，上面的论坛板块页面用到`GridLayoutManager`它每行的个数不一样，head的地方应该是放一个item，而body的地方是放3个item的，这个可以用`GridLayoutManager`的一个`setSpanSizeLookup`解决

```java
GridLayoutManager glm = new GridLayoutManager(this, 3);
        glm.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
            @Override
            public int getSpanSize(int position) {
                if (adapter.getItemViewType(position) == GroupRecyclerViewAdapter.Companion.getITEM_TYPE_HEAD()) {
                    return 3;
                }else {
                    return 1;
                }
            }
        });
```

根据position判断当前的view是head还是body，然后给出不同的元素个数就行了。

