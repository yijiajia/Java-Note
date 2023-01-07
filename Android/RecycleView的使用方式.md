## RecycleView的使用方式

> RecycleView是ListView的优化方式，不仅弥补了ListView只能纵向滚动的缺点，同时提供了多种布局管理来实现多样的布局方式

在app目录下的build.gradle文件中添加recycleview的依赖
```groovy
dependencies {
    implementation 'androidx.core:core-ktx:1.7.0'
    implementation 'androidx.appcompat:appcompat:1.4.1'
    implementation 'com.google.android.material:material:1.5.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.3'
    implementation 'androidx.recyclerview:recyclerview:1.2.1'
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
}
```

RecycleView 提供了**LinearLayoutManager**、**GridLayoutManager**、**StaggeredGridLayoutManager** 三种布局管理。

使用方式一般分为以下几步：

1. 创建布局文件；
2. 创建RecycleView的子项控件；
3. 创建RecycleView的适配器，继承自RecyclerView.Adapter，重写以下三个方法
   * 创建ViewHolder的方法：`public abstract VH onCreateViewHolder(@NonNull ViewGroup parent, int viewType);`
   * 绑定数据的方法：`public abstract void onBindViewHolder(@NonNull VH holder, int position);`
   * 获取项目条数的方法：`public abstract int getItemCount();`

4. 在Activity中配置适配器，并指定layoutManager为以上三种布局管理的一种。





### LinearLayoutManager的使用方式

> 线性布局

#### 1. 创建Activity，生成layout布局和Activity

* activity_recycle_view.xml

> 使用RecyclerView控件填充布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".layout.RecycleViewActivity">
    
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycleView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```
* RecycleViewActivity

```kotlin
class RecycleViewActivity : AppCompatActivity() {

    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_recycle_view)
    }
}
```



#### 2. 创建RecycleView的子项控件

* fruit_item.xml

> 定义了一个图片和文案，垂直居中

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/fruitImage"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="10dp"/>

    <TextView
        android:id="@+id/fruitName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="10dp"/>

</LinearLayout>
```

#### 3. 创建RecycleView的适配器

> 继承自RecycleView.Adapter，并将泛型指定为ViewHolder其内部类。同时指定子项控件为R.layout.fruit_item，并在onBindViewHolder填充数据

```kotlin
class FruitRecycleAdapter(val fruitList : List<Fruit>) :
    RecyclerView.Adapter<FruitRecycleAdapter.ViewHolder>() {

    inner class ViewHolder(view : View) : RecyclerView.ViewHolder(view) {
        val fruitImage : ImageView = view.findViewById(R.id.fruitImage)
        val fruitName : TextView = view.findViewById(R.id.fruitName)
    }

    /**
     * 加载自定义的布局，创建ViewHolder实例
     */
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.fruit_item,parent,false)
        return ViewHolder(view)
    }

    /**
     * 对 RecycleView子项的数据进行赋值，会在每个子项被滚动到屏幕内的时候执行
     * 通过position来定位数据，
     */
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val fruit = fruitList[position]
        holder.fruitImage.setImageResource(fruit.imageId)
        holder.fruitName.text = fruit.name
    }

    // 返回Item的数目
    override fun getItemCount() = fruitList.size
}
```

#### 4. 在Activity中配置适配器

> 指定layoutManager为LinearLayoutManager，并配置适配器为前面创建的适配器

```
class RecycleViewActivity : AppCompatActivity() {

    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_recycle_view)

        initFruits()

        val layoutManager = LinearLayoutManager(this)
        val recyclerView : RecyclerView = findViewById(R.id.recycleView)

        recyclerView.layoutManager = layoutManager
        val adapter = FruitRecycleAdapter(fruitList)
        recyclerView.adapter = adapter
    }

    private fun initFruits() {
        repeat(2) {
            fruitList.add(Fruit("Apple", R.drawable.apple_pic))
            fruitList.add(Fruit("Banana", R.drawable.banana_pic))
            fruitList.add(Fruit("Orange", R.drawable.orange_pic))
            fruitList.add(Fruit("Watermelon", R.drawable.watermelon_pic))
            fruitList.add(Fruit("Pear", R.drawable.pear_pic))
            fruitList.add(Fruit("Grape", R.drawable.grape_pic))
            fruitList.add(Fruit("Pineapple", R.drawable.pineapple_pic))
            fruitList.add(Fruit("Strawberry", R.drawable.strawberry_pic))
            fruitList.add(Fruit("Cherry", R.drawable.cherry_pic))
            fruitList.add(Fruit("Mango", R.drawable.mango_pic))
        }
    }
}
```

#### 5. 运行截图

和ListView中自定义控件的运行效果一致。

[![zjesGn.png](https://s1.ax1x.com/2022/12/23/zjesGn.png)](https://imgse.com/i/zjesGn)



#### 6.  LinearLayoutManager 横向滚动使用

> 此外LinearLayoutManager还可以实现横向滚动，指定方向即可

1. 创建Activity，生成layout布局和Activity。

   > 与前面的一样

2. 创建RecycleView的子项控件

   > 这里与上面的不同的是，指定了图片和文案的方向为水平居中

* fruit_item_vertical.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="80dp"
    android:layout_height="match_parent">

    <!--垂直排列 -->
    <ImageView
        android:id="@+id/fruitImage_v"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_gravity="center_horizontal"
        android:layout_marginLeft="10dp"/>

    <TextView
        android:id="@+id/fruitName_v"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginLeft="10dp"/>

</LinearLayout>
```

3. 创建RecycleView的适配器

> 与前面的一样，只不过这里需要指定fruit_item_vertical为要渲染的布局

4. 在RecycleView中配置适配器，并指定方向为水平

```kotlin
class RecycleViewVerticalActivity : AppCompatActivity() {

    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_recycle_view_vertical)

        initFruits()

        val layoutManager = LinearLayoutManager(this)
        layoutManager.orientation = LinearLayoutManager.HORIZONTAL

        val recycleViewVerticalActivity : RecyclerView = findViewById(R.id.recycleView_v)
        recycleViewVerticalActivity.layoutManager = layoutManager

        val adapter = FruitRecycleVerticalAdapter(fruitList)
        recycleViewVerticalActivity.adapter = adapter
    }

    private fun initFruits() {
        repeat(2) {
            fruitList.add(Fruit("Apple", R.drawable.apple_pic))
            fruitList.add(Fruit("Banana", R.drawable.banana_pic))
            fruitList.add(Fruit("Orange", R.drawable.orange_pic))
            fruitList.add(Fruit("Watermelon", R.drawable.watermelon_pic))
            fruitList.add(Fruit("Pear", R.drawable.pear_pic))
            fruitList.add(Fruit("Grape", R.drawable.grape_pic))
            fruitList.add(Fruit("Pineapple", R.drawable.pineapple_pic))
            fruitList.add(Fruit("Strawberry", R.drawable.strawberry_pic))
            fruitList.add(Fruit("Cherry", R.drawable.cherry_pic))
            fruitList.add(Fruit("Mango", R.drawable.mango_pic))
        }
    }
}
```



5. 运行截图（可水平滑动）

[![zjnPh9.png](https://s1.ax1x.com/2022/12/23/zjnPh9.png)](https://imgse.com/i/zjnPh9)





### GridLayoutManager的使用方式

> 网格布局



#### 1. 创建Activity，生成layout布局和Activity

* activity_grid.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".layout.GridActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycleView_grid"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

* GridActivity 文件

```kotlin
class GridActivity : AppCompatActivity() {

    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_grid)
    }
}
```



#### 2. 创建RecycleView的子项控件

* fruit_item_grid.xml，水平居中

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_margin="5dp"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/fruitImage_grid"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"/>

    <TextView
        android:id="@+id/fruitName_grid"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"/>

</LinearLayout>
```



#### 3. 创建RecycleView的适配器

> 继承自RecyclerView.Adapter，重写其方法

```kotlin
class GridAdapter(val fruitList : List<Fruit>) : RecyclerView.Adapter<GridAdapter.ViewHolder>()  {

    inner class ViewHolder(view : View) : RecyclerView.ViewHolder(view) {
        val fruitImage : ImageView = view.findViewById(R.id.fruitImage_grid)
        val fruitName : TextView = view.findViewById(R.id.fruitName_grid)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.fruit_item_grid,parent,false)

        val viewHolder = ViewHolder(view)
        // 注册点击事件
        // 针对最外层子项注册点击事件
        viewHolder.itemView.setOnClickListener {
//            val position = viewHolder.adapterPosition
            val position = viewHolder.bindingAdapterPosition
            val fruit = fruitList[position]
            Toast.makeText(parent.context, "you clicked View ${fruit.name}", Toast.LENGTH_SHORT).show()
        }

        // 针对image注册点击事件
        viewHolder.fruitImage.setOnClickListener {
            val position = viewHolder.adapterPosition
            val fruit = fruitList[position]
            Toast.makeText(parent.context, "you clicked Image ${fruit.name}", Toast.LENGTH_SHORT).show()
        }

        return viewHolder
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val fruit = fruitList[position]
        holder.fruitImage.setImageResource(fruit.imageId)
        holder.fruitName.text = fruit.name
    }

    override fun getItemCount() = fruitList.size

}
```

#### 4. 在Activity中配置适配器

* GridActivity

> 指定3列

```kotlin
class GridActivity : AppCompatActivity() {

    private val fruitList = ArrayList<Fruit>()


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_grid)

        initFruits()

        val layoutManager = GridLayoutManager(this,3)
        val recyclerGridActivity : RecyclerView = findViewById(R.id.recycleView_grid)
        recyclerGridActivity.layoutManager = layoutManager

        val adapter = GridAdapter(fruitList)
        recyclerGridActivity.adapter = adapter

    }

    private fun initFruits() {
        repeat(5) {
            fruitList.add(Fruit("Apple",R.drawable.apple_pic))
            fruitList.add(Fruit("Banana",R.drawable.banana_pic))
            fruitList.add(Fruit("Orange",R.drawable.orange_pic))
            fruitList.add(Fruit("Watermelon",R.drawable.watermelon_pic))
            fruitList.add(Fruit("Pear",R.drawable.pear_pic))
            fruitList.add(Fruit("Grape",R.drawable.grape_pic))
            fruitList.add(Fruit("Pineapple",R.drawable.pineapple_pic))
            fruitList.add(Fruit("Strawberry",R.drawable.strawberry_pic))
            fruitList.add(Fruit("Cherry",R.drawable.cherry_pic))
            fruitList.add(Fruit("Mango",R.drawable.mango_pic))
        }
    }
}
```

#### 5. 运行截图

 [![zjnIjx.png](https://s1.ax1x.com/2022/12/23/zjnIjx.png)](https://imgse.com/i/zjnIjx)





### StaggeredGridLayoutManager的使用方式

> 瀑布流布局是网格布局的升级版，可以让布局的子项高度不一致

#### 1. 创建Activity，生成layout布局和Activity

* activity_staggered_grid.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".layout.StaggeredGridActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycleView_stg"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

* StaggeredGridActivity.kt

```kotlin
class StaggeredGridActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_staggered_grid)
    }
}
```

#### 2. 创建RecycleView的子项控件

* fruit_item_stagger.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_margin="5dp"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    
    <ImageView
        android:id="@+id/fruitImage_stg"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"/>
    
    <TextView
        android:id="@+id/fruitName_stg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="left"
        android:layout_marginTop="10dp"/>

</LinearLayout>
```

#### 3. 创建RecycleView的适配器

* FruitRecycleStgApapter.kt

```kotlin
class FruitRecycleStgAdapter(val fruitList : List<Fruit>) :
    RecyclerView.Adapter<FruitRecycleStgAdapter.ViewHolder>() {

    inner class ViewHolder(view : View) : RecyclerView.ViewHolder(view) {
        val fruitImage : ImageView = view.findViewById(R.id.fruitImage_stg)
        val fruitName : TextView = view.findViewById(R.id.fruitName_stg)
    }

    /**
     * 加载自定义的布局，创建ViewHolder实例
     */
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.fruit_item_stagger,parent,false)

        val viewHolder = ViewHolder(view)
        // 注册点击事件
        // 针对最外层子项注册点击事件
        viewHolder.itemView.setOnClickListener {
//            val position = viewHolder.adapterPosition
            val position = viewHolder.bindingAdapterPosition
            val fruit = fruitList[position]
            Toast.makeText(parent.context, "you clicked View ${fruit.name}",Toast.LENGTH_SHORT).show()
        }

        // 针对image注册点击事件
        viewHolder.fruitImage.setOnClickListener {
            val position = viewHolder.adapterPosition
            val fruit = fruitList[position]
            Toast.makeText(parent.context, "you clicked Image ${fruit.name}",Toast.LENGTH_SHORT).show()
        }

        return viewHolder
    }

    /**
     * 对 RecycleView子项的数据进行赋值，会在每个子项被滚动到屏幕内的时候执行
     * 通过position来定位数据，
     */
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val fruit = fruitList[position]
        holder.fruitImage.setImageResource(fruit.imageId)
        holder.fruitName.text = fruit.name
    }

    override fun getItemCount() = fruitList.size
}
```

#### 4. 在Activity中配置适配器

> 指定StaggeredGridLayoutManager布局管理，设置排3列，纵向排列

```kotlin
class StaggeredGridActivity : AppCompatActivity() {

    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_staggered_grid)

        initFruits()

        /**
         * 设置排3列，纵向排列
         */
        val layoutManager = StaggeredGridLayoutManager(3,StaggeredGridLayoutManager.VERTICAL)
        val recyclerViewStg : RecyclerView = findViewById(R.id.recycleView_stg)
        recyclerViewStg.layoutManager = layoutManager

        val adapter = FruitRecycleStgAdapter(fruitList)
        recyclerViewStg.adapter = adapter

    }


    private fun initFruits() {
        repeat(4) {
            fruitList.add(Fruit(getRandomLenStr("Apple"), R.drawable.apple_pic))
            fruitList.add(Fruit(getRandomLenStr("Banana"), R.drawable.banana_pic))
            fruitList.add(Fruit(getRandomLenStr("Orange"), R.drawable.orange_pic))
            fruitList.add(Fruit(getRandomLenStr("Watermelon"), R.drawable.watermelon_pic))
            fruitList.add(Fruit(getRandomLenStr("Pear"), R.drawable.pear_pic))
            fruitList.add(Fruit(getRandomLenStr("Grape"), R.drawable.grape_pic))
            fruitList.add(Fruit(getRandomLenStr("Pineapple"), R.drawable.pineapple_pic))
            fruitList.add(Fruit(getRandomLenStr("Strawberry"), R.drawable.strawberry_pic))
            fruitList.add(Fruit(getRandomLenStr("Cherry"), R.drawable.cherry_pic))
            fruitList.add(Fruit(getRandomLenStr("Mango"), R.drawable.mango_pic))
        }
    }

    private fun getRandomLenStr(str : String) : String {
        val n = (1..20).random()
        val builder = StringBuilder()
        repeat(n) {
            builder.append(str)
        }
        return builder.toString()
    }
}
```

#### 5. 运行截图

> 随机生成文案内容，长度各不一致，可以看出各行的高度都是不一样的

[![zjw0XT.png](https://s1.ax1x.com/2022/12/23/zjw0XT.png)](https://imgse.com/i/zjw0XT)