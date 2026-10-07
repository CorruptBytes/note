# 四大组件

## Activity

负责界面展示和用户交互，一个 Activity 通常对应一个屏幕页面。

### 生命周期

```
onCreate() → onStart() → onResume() → 运行中 → onPause() → onStop() → onDestroy()
```

## Service

在后台执行长时间运行操作。

两种形式：

- Started Service：启动后独立运行，即使启动它的组件销毁也能继续
- Bound Service：组件绑定到服务，可跨进程通信（IPC）

## BroadcastReceiver

监听和响应系统或应用发出的广播消息。

两种注册方式：

- 静态注册：在 AndroidManifest.xml 中声明，应用未启动也能接收
- 动态注册：代码中 registerReceiver()，需手动注销

## ContentProvider

在不同应用之间共享数据，统一的数据访问接口。

# UI界面

Android 编写 UI 界面的方式主要有三种：

- XML
- Jetpack Compose
- 纯代码动态创建

目前主流是 XML 布局 和 Jetpack Compose。

## XML布局

在 `res/layout/` 下写 XML 文件，通过 `setContentView()` 加载。

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello Android"
        android:textSize="24sp" />

    <Button
        android:id="@+id/btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="点击" />

</LinearLayout>
```

```
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        findViewById<Button>(R.id.btn).setOnClickListener {
            // 处理点击
        }
    }
}
```

## Jetpack Compose

目前 Google 主推的方案，声明式 UI，用 Kotlin 代码直接描述界面，无需 XML。

```
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp()
        }
    }
}

@Composable
fun MyApp() {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        Text(
            text = "Hello Android",
            fontSize = 24.sp
        )
        Button(onClick = { /* 处理点击 */ }) {
            Text("点击")
        }
    }
}
```

### Composable函数

使用 `@Composable` 注解的函数，是 Compose 的核心单元，用于描述 UI。

```
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}
```

- 函数名首字母大写
- 不能返回任何值（Unit）
- 在 Composable 函数内只能调用其他 Composable 函数

### 实时预览功能

在 Composable 函数上添加 `@Preview` 注解即可实时预览。

```
@Preview(showBackground = true, showSystemUi = true)
@Composable
fun GreetingPreview() {
    MyAppTheme {
        Greeting("Compose")
    }
}
```

### Modifier

Modifier 是 Compose 中控制外观和行为的通用机制。

```
Text(
    text = "Hello",
    modifier = Modifier
        .padding(16.dp)              // 内边距
        .fillMaxWidth()              // 填满宽度
        .height(50.dp)               // 固定高度
        .background(Color.Red)       // 背景色
        .clickable { /* 点击事件 */ }
)
```

Modifier 的链式调用顺序会影响组件的具体表现。

```
// 先 padding 再 background：背景不包含 padding
Modifier.padding(16.dp).background(Color.Red)

// 先 background 再 padding：背景包含 padding
Modifier.background(Color.Red).padding(16.dp)
```

| Modifier                 | 作用     |
| ------------------------ | -------- |
| padding                  | 内边距   |
| size / width / height    | 尺寸     |
| fillMaxWidth/Height/Size | 填满容器 |
| background               | 背景色   |
| clip                     | 裁剪形状 |
| clickable                | 点击事件 |
| border                   | 边框     |
| offset                   | 偏移     |

# 布局

## Column

垂直排列。

```
Column(
    modifier = Modifier.fillMaxSize(),
    verticalArrangement = Arrangement.Center,
    horizontalAlignment = Alignment.CenterHorizontally
) {
    Text("第一行")
    Text("第二行")
}
```

## Row

水平排列。

```
Row(
    modifier = Modifier.fillMaxWidth(),
    horizontalArrangement = Arrangement.SpaceBetween,
    verticalAlignment = Alignment.CenterVertically
) {
    Text("左侧")
    Text("右侧")
}
```

## Box

层叠布局。

```
Box(
    modifier = Modifier.fillMaxSize(),
    contentAlignment = Alignment.Center
) {
    Image(painter = ..., contentDescription = null)
    Text("叠加文字")
}
```