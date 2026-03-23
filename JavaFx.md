# 概述

JavaFX 是 Java 官方推出的一套 **用于构建富客户端应用程序**（Rich Client Applications）的 GUI 框架，主要用于开发桌面应用程序。它是Swing的后继者，旨在提供更强大、现代、美观的用户界面构建方式。

JavaFX在JDK8时推出，与`JDK`捆绑在一起。从JDK11起，`JavaFX`不再和 JDK 捆绑，变成了一个独立的开源项目，改名为`OpenJFX`。

- JDK11后，除了通过项目管理工具导入`JavaFX`，还可以下载其`SDK`导入(<a href="https://gluonhq.com/products/javafx/">下载链接</a>)。

  

## 相关依赖

```xml
 <dependency>
      <groupId>org.openjfx</groupId>
      <artifactId>javafx-controls</artifactId>
      <version>17.0.6</version>
    </dependency>

    <dependency>
      <groupId>org.openjfx</groupId>
      <artifactId>javafx-fxml</artifactId>
      <version>17.0.6</version>
    </dependency>

	<dependency>     
        <groupId>org.openjfx</groupId>
        <artifactId>javafx-maven-plugin</artifactId>
        <version>0.0.8</version>
	</dependency>
```



- 特点

  - MVC设计模式

    > Model-View-Controller的设计模式，将数据，视图与逻辑分离。

  - CSS样式支持

    > 可以像Web一样通过CSS修改控件样式。

  - 使用FXML

    > FXML是一种基于XML的声明式标记语言，用于描述JavaFX应用程序的用户界面，这使JavaFX的UI与逻辑分离。

  - 动画支持

    > 内置丰富的动画和过渡效果

  - 硬件加速渲染

    > 利用 GPU 提高渲染效率

  - 可嵌入Swing

    > 可以将 JavaFX 嵌入已有 Swing 应用中或将Swing嵌入JavaFX应用中。

  - 提供WebView组件，实现Web页面嵌入

- 组成

  > 整个JavaFX框架由五个包组成。

  |      模块名       |             描述             |
  | :---------------: | :--------------------------: |
  | `javafx-controls` | 所有 UI 控件，如按钮、表格等 |
  |   `javafx-fxml`   |        支持 FXML 加载        |
  | `javafx-graphics` |    场景图形、节点、动画等    |
  |  `javafx-media`   |        音视频播放支持        |
  |   `javafx-web`    |   内嵌 WebView 浏览器组件    |

- 使用

  ```java
  public class Demo extends Application {
      public static void main(String[] args) {
      //两种方法均可以
          //Application.launch(args);
          launch(args);
      }
      @Override
      public void start(Stage primaryStage) throws Exception {
          primaryStage.show();
      }
  }
  
  ```

# UI控件

## MenuBar

> 菜单栏，通常放在窗口顶部

- 使用

  ```java
   MenuBar menuBar = new MenuBar();
  Menu fileMenu = new Menu("文件");
  MenuItem openItem = new MenuItem("打开");
  MenuItem exitItem = new MenuItem("退出");
  fileMenu.getItems().addAll(openItem,exitItem);
  Menu editMenu = new Menu("编辑");
  editMenu.getItems().add(new MenuItem("复制"));
  editMenu.getItems().add(new MenuItem("粘贴"));
  menuBar.getMenus().addAll(fileMenu,editMenu);
  ```

## ContextMenu

> 上下文菜单，一般和其他控件绑定，在右键时弹出

- 使用

  ```java
  TextField textField = new TextField();
  
  ContextMenu contextMenu = new ContextMenu();
  MenuItem copyItem = new MenuItem("复制");
  MenuItem pasteItem = new MenuItem("粘贴");
  contextMenu.getItems().addAll(copyItem, pasteItem);
  
  // 设置右键菜单
  textField.setContextMenu(contextMenu);
  //也可以绑定在控件事件中
  button.setOnContextMenuRequested(event -> {
      contextMenu.show(button, event.getScreenX(), event.getScreenY());
  });
  
  ```

## FileChooser

> 文件对话框，用于选择文件或保存文件

- 使用

  ```java
  FileChooser openChooser = new FileChooser();
  File file = openChooser.showOpenDialog(stage);
  ```

  

- Label
- Button
- Radio Button
- Toggle Button
- Checkbox
- ChoiceBox
- Text Field
- Password Field
- Scroll Bar
- Scroll Pane

## List View

> 列表视图控件，用来显示一列可选项的控件。

- API

  |                                                            |                        |      |
  | :--------------------------------------------------------: | :--------------------: | ---- |
  |         public final ObservableList<T> getItems()          | 获得列表控件的数据模型 |      |
  | public final MultipleSelectionModel<T> getSelectionModel() |      获取选择模型      |      |
  |                                                            |                        |      |

  
  
  ## TableView

  > 表格控件，可以显示结构化的数据列表，支持排序，编辑，样式设置等高级功能。

- 使用

  ```java
  //需要指定泛型，泛型类型即为需要结构化显示数据的类型
  TableView<Person> tableView = new TableView<>();
  TableColumn<Person,String> nameCol = new TableColumn<>("姓名");
  //nmae与age要与getter方法中的getName与getAge相匹配
  nameCol.setCellValueFactory(new PropertyValueFactory<>("name"));
  TableColumn<Person,Integer> ageCol = new TableColumn<>("年龄");
  ageCol.setCellValueFactory(new PropertyValueFactory<>("age"));
  tableView.getColumns().addAll(nameCol,ageCol);
  ObservableList<Person> data = FXCollections.observableArrayList(
      new Person("张三", 20),
      new Person("李四", 25),
      new Person("王五", 30)
  );
  table.setItems(data);
  ```

- Property类型

  > TalbeView强烈依赖JavaBeans的Property类型，用于绑定与更新

  ```java
  public class Person {
      private final SimpleStringProperty name;
      private final SimpleIntegerProperty age;
  
      public Person(SimpleStringProperty name, SimpleIntegerProperty age) {
          this.name = name;
          this.age = age;
      }
  }
  ```

  > 需要在TalbeView中显示的数据的类型需要为Property类型

- 常用API

## Tree View

> 通过一个树形结构来展示数据

- 使用

  ```java
  TreeItem<String> rootItem = new TreeItem<>("根节点");
  TreeItem<String> item1 = new TreeItem<>("子节点1");
  TreeItem<String> item2 = new TreeItem<>("子节点2");
  rootItem.getChildren().addAll(item1,item2);
  rootItem.setExpanded(true);
  TreeView<String> root = new TreeView<>(rootItem);
  ```


Tree Table View

Combo Box

Separator

Slider

Progress Bar

Progress Indicator

Hyperlink

Tooltip

HTML Editor

Titled Pane

AccordionMenu

Color Picker

Date Picker

Pagination Control

File Chooser

## Canvas

> 一个可以进行低级图形绘制的控件，可以使用一支"画笔"在画布上作画。

- 使用

  ```java
  Canvas canvas = new Canvas(400, 300); // 宽400，高300
  GraphicsContext gc = canvas.getGraphicsContext2D();
  
  gc.setFill(Color.LIGHTBLUE);
  gc.fillRect(50, 50, 200, 100); // 填充矩形
  
  gc.setStroke(Color.DARKBLUE);
  gc.strokeLine(0, 0, 400, 300); // 绘制对角线
  
  gc.setFont(Font.font("Arial", 24));
  gc.setFill(Color.BLACK);
  gc.fillText("Hello Canvas!", 100, 200); // 画文字
  
  Image image = new Image("file:src/main/resources/img/1.jpg");
  gc.drawImage(image, 100, 100);
  ```

  > 注意:
  >
  > - GraphicsContext对象相当于一根画笔。


## ImageView

> 专门用于显示Image图像的控件。

|           方法           |         说明         |
| :----------------------: | :------------------: |
|  `setFitWidth(double)`   |       设置宽度       |
|  `setFitHeight(double)`  |       设置高度       |
| `setPreserveRatio(true)` |      保持宽高比      |
|   `setRotate(double)`    |       旋转图像       |
|   `setOpacity(double)`   |  设置透明度（0~1）   |
|     `setClip(Node)`      | 设置剪裁区域（遮罩） |

> 注意:
>
> - ImageView不但可以作为独立的控件显示图像，还可以作为其他控件(如Button)的图标显示。

## Accordinn

> 折叠面板

- 使用

  ```java
  TitledPane pane1 = new TitledPane("面板一", new Label("这是内容一"));
  TitledPane pane2 = new TitledPane("面板二", new Label("这是内容二"));
  TitledPane pane3 = new TitledPane("面板三", new Label("这是内容三"));
  
  Accordion accordion = new Accordion();
  accordion.getPanes().addAll(pane1, pane2, pane3);
  accordion.setExpandedPane(pane1); // 默认展开第一个
  
  ```

- 常用API

  | 特性                                 | 描述                                    |
  | ------------------------------------ | --------------------------------------- |
  | `.getTabs()`                         | 获取标签页集合（`ObservableList<Tab>`） |
  | `.setContent(...)`                   | 设置当前 `Tab` 页签对应的展示内容       |
  | `.setClosable(true)`                 | 标签页是否可以被关闭（默认为 `true`）   |
  | `.getSelectionModel().select(index)` | 手动切换当前选中标签页                  |


## Alert

> 提示框。

# Stage

> Stage是JavaFX中的顶层容器，每一个stage都是一个窗口。在Application的start()方法中，会自动提供一个窗口作为主窗口。

- stage对象貌似只能在start方法中创建，否则会爆出关于反射的错误

- 常用API

  - `setIconified(boolean)`

    > 设置窗口最小化，iconify意为将窗口缩成一个图标，沿用自早期操作系统的命名

  - show()

    > 显示窗口

  - close()

    > 与show()方法相对，但是窗口并未被完全关闭，只是不显示

  - setWidth

  - setHeight

  - setResizeable
  
  - heightProperty().addListener
  
    > 为高度属性添加监听器，监听高度的变化
  
  - setScene()
  
  - setFullScreen()
  
  - setOpacity()
  
  - setX()
  
  - setY()
  
  - setAlwaysOnTop()
  
  - initStyle()
  
  - initModality
  
  - initOwner

# Scence

> Scence意为场景，是所有需要显示UI元素的容器，需要将Scence放在Stage上显示，一个`Stage`可以切换多个`Scene`，但同一时刻只能显示一个。每一个Scence都具有一个scence graphic，这是一个树状数据结构，用于管理控件

## Node

> Node 是 JavaFX UI 的最基本构建单位。所有控件和容器都继承自Node。这些Node构成了一颗树，被称为Scene Graph(场景图)。

## Scence Graphic

> 场景图，场景中需要显示多个控件，附加到场景中的控件，布局等的所有对象会形成一个树状结构，称为场景图。每一个控件与布局都是一个节点，分为分支节点与叶节点。他们都继承了javafx.scence.Node类



# Pane(布局)

> JavaFX使用布局容器来组织和排列界面中的控件

## 常用布局

|   布局容器   |        作用        |        示例控件         |
| :----------: | :----------------: | :---------------------: |
|    `Pane`    |      自由布局      |      手动设定坐标       |
|    `HBox`    |      横向排列      |   按顺序从左到右放置    |
|    `VBox`    |      纵向排列      |   按顺序从上到下放置    |
| `BorderPane` | 上下左右中五个区域 |    类似网页五区布局     |
|  `GridPane`  |      网格布局      |    类似表格（行列）     |
| `StackPane`  |      堆叠布局      |   所有子元素重叠显示    |
|  `FlowPane`  |      流式布局      | 类似 HTML 的 float 效果 |

### Hbox/VBox(水平/垂直盒式布局)

> HBox会水平排列其子节点，VBox中的子节点会竖直排列

- API

  |               |                                  |      |
  | ------------- | -------------------------------- | ---- |
  | setPadding    | 设置盒子内间距                   |      |
  | setAlignment  | 设置各个节点相对父节点的对齐方式 |      |
  | getChildren() |                                  |      |

  | 枚举值             | 含义       |
  | ------------------ | ---------- |
  | `Pos.TOP_LEFT`     | 顶部左对齐 |
  | `Pos.CENTER`       | 居中对齐   |
  | `Pos.BOTTOM_RIGHT` | 底部右对齐 |

### BorderPane(边界布局)

> 边界布局将界面分为五个区域:`Top`（顶部）`Bottom`（底部）`Left`（左侧）`Right`（右侧）`Center`（中间）

- API

  |                                                         |      |      |
  | ------------------------------------------------------- | ---- | ---- |
  | setTop                                                  |      |      |
  | setBottom                                               |      |      |
  | setLeft                                                 |      |      |
  | setRight                                                |      |      |
  | setCenter                                               |      |      |
  | BorderPane.setMargin(root.getCenter(), new Insets(30)); |      |      |

  > 注意:
>
  > - 每个区域只能放一个**节点**（Node），可以通过嵌套节点放置多个节点

### GridPane(网格布局)

> 是一种表格布局容器，可将控件按照**行（row）和列（column）**精确对齐。

- API

  |                                               |                        |      |
  | --------------------------------------------- | ---------------------- | ---- |
  | grid.add(node, columnIndex, rowIndex)         |                        |      |
  |                                               |                        |      |
  | GridPane.setHalignment(loginBtn, HPos.RIGHT); | 控件在单元格中对齐方式 |      |

### StackPane(层叠布局)

> 所有子节点会被叠放在一起，默认居中，后添加的控件会压住前面的控件，但是不会完全覆盖

- API

  |                        |      |      |
  | ---------------------- | ---- | ---- |
  | StackPane.setAlignment |      |      |
  | StackPane.setMargin    |      |      |
  |                        |      |      |


### TabPane

> 标签页布局，专门用于放置多个标签页

- 使用

  ```java
  TabPane tabPane = new TabPane();
  
  Tab tab1 = new Tab("首页");
  tab1.setContent(new Label("欢迎来到首页"));
  
  Tab tab2 = new Tab("设置");
  tab2.setContent(new Label("这里是设置界面"));
  
  tabPane.getTabs().addAll(tab1, tab2);
  
  ```

- 常用API

  | 特性                                 | 描述                                    |
  | ------------------------------------ | --------------------------------------- |
  | `.getTabs()`                         | 获取标签页集合（`ObservableList<Tab>`） |
  | `.setContent(...)`                   | 设置当前 `Tab` 页签对应的展示内容       |
  | `.setClosable(true)`                 | 标签页是否可以被关闭（默认为 `true`）   |
  | `.getSelectionModel().select(index)` | 手动切换当前选中标签页                  |

### AnchorPane



# 事件处理

> JavaFX中的事件处理机制是基于观察者模式设计，基于观察者模式设计了多个事件对象。

|     类型      |          说明          |
| :-----------: | :--------------------: |
| `ActionEvent` |  按钮点击、菜单选择等  |
| `MouseEvent`  | 鼠标进入、点击、拖动等 |
|  `KeyEvent`   |    键盘按下、释放等    |
| `WindowEvent` |    窗口关闭、打开等    |

​		JavaFX通过控件的SetOnXXX()方法为控件绑定事件处理函数，事件处理函数在触发时会传入一个事件对象。

|   控件    |        方法         |        描述         |
| :-------: | :-----------------: | :-----------------: |
|  Button   |    `setOnAction`    |    按钮点击事件     |
| TextField |  `setOnKeyPressed`  |   按下键盘时触发    |
| CheckBox  |    `setOnAction`    | 勾选/取消勾选时触发 |
|   Scene   | `setOnMouseClicked` | 鼠标点击场景时触发  |

> 注意:
>
> - 函数参数需要实现一个EventHandler接口，这个接口中的方法会自动传入一个事件对象参数。

# 属性

> JavaFX通过`属性(Property)`机制简化UI控件之间的数据同步，数据同步的核心手段是绑定，分为双向绑定与单向绑定。

## 属性绑定

### 单向绑定

> 通过属性的bind()方法进行属性的单向绑定

```java
//简单绑定
label.textProperty().bind(textField.textProperty());
```

### 双向绑定

> 通过属性的bindBidirectional()进行属性的双向绑定。

```java
//双向绑定
textField1.textProperty().bindBidirectional(textField2.textProperty());
```

### 自定义绑定表达式

> 利用 `Bindings` 类中的静态方法，或继承 `Binding<T>` 接口，自定义多个属性之间的动态关系。

#### 使用Bingdings中的静态方法

> JavaFX 的 `Bindings` 类是一个工具类，它提供了大量的**静态方法**，用来创建各种**绑定表达式对象（Binding）**，可以自动监听属性变化并动态计算结果

- 数值计算绑定

  ```java
  SimpleIntegerProperty a = new SimpleIntegerProperty(5);
  SimpleIntegerProperty b = new SimpleIntegerProperty(3);
  
  // 绑定 c 为 a + b 的结果
  NumberBinding c = Bindings.add(a, b);
  
  System.out.println(c.getValue()); // 输出 8
  a.set(10);
  System.out.println(c.getValue()); // 输出 13，自动更新
  
  ```

- 字符串拼接绑定

  ```java
  SimpleStringProperty name = new SimpleStringProperty("Alice");
  Label label = new Label();
  
  label.textProperty().bind(
      Bindings.concat("你好，", name, "！")
  );
  
  ```

- 条件表达式绑定

  ```java
  SimpleIntegerProperty score = new SimpleIntegerProperty(75);
  
  StringBinding result = Bindings.when(score.greaterThanOrEqualTo(60))
      .then("及格")
      .otherwise("不及格");
  
  System.out.println(result.get()); // 输出：及格
  
  ```

- 常用API

  | 方法                                                         | 描述                       | 示例                                    |
  | ------------------------------------------------------------ | -------------------------- | --------------------------------------- |
  | `Bindings.add()`                                             | 加法绑定                   | `Bindings.add(a, b)`                    |
  | `Bindings.subtract()`                                        | 减法绑定                   | `Bindings.subtract(a, b)`               |
  | `Bindings.multiply()`                                        | 乘法绑定                   | `Bindings.multiply(a, b)`               |
  | `Bindings.divide()`                                          | 除法绑定                   | `Bindings.divide(a, b)`                 |
  | `Bindings.min()/max()`                                       | 求最小/最大值              | `Bindings.min(a, b)`                    |
  | `Bindings.concat()`                                          | 字符串拼接                 | `Bindings.concat("姓名: ", name)`       |
  | `Bindings.format()`                                          | 字符串格式化               | `Bindings.format("%.2f", price)`        |
  | `Bindings.when(...).then(...).otherwise(...)`                | 三元条件表达式             | 判断通过就返回 then，否则返回 otherwise |
  | `Bindings.createXXXBinding(Callable<XXX> func,Observable... dependencies)` | 创建一个表达式，逻辑自定义 |                                         |

- 表达式类型

  | 返回类型           | 描述                    |
  | ------------------ | ----------------------- |
  | `BooleanBinding`   | 布尔表达式              |
  | `StringBinding`    | 字符串表达式            |
  | `NumberBinding`    | 数值表达式（整型/浮点） |
  | `DoubleBinding`    | Double 绑定表达式       |
  | `ObjectBinding<T>` | 任意类型表达式          |

#### 自定义Binding子类

> 继承 `Binding<T>` 或 `XXXBinding` 类，实现自己的表达式逻辑。

```java
DoubleProperty width = new SimpleDoubleProperty(100);
DoubleProperty height = new SimpleDoubleProperty(200);

DoubleBinding area = new DoubleBinding() {
    {
        super.bind(width, height);
    }

    @Override
    protected double computeValue() {
        return width.get() * height.get();
    }
};

```

## 属性类型

> JavaFX提供了多个不同类型的Property，用于不同类型数据的绑定

| 类型   | 类名              |
| ------ | ----------------- |
| 整数   | `IntegerProperty` |
| 浮点数 | `DoubleProperty`  |
| 布尔值 | `BooleanProperty` |
| 字符串 | `StringProperty`  |

- API

  | 方法 |      |      |
  | :--: | ---- | ---- |
  |      |      |      |
  |      |      |      |
  |      |      |      |

## 监听器(Listener)

> JavaFX中，监听器用于监听数据变化，允许在数据变化时触发回调逻辑，实现更为灵活的行为控制，JavaFX提供了两种监听器接口:

|          接口          |             功能描述             |
| :--------------------: | :------------------------------: |
|  `ChangeListener<T>`   | 监听属性值的变化（旧值 → 新值）  |
| `InvalidationListener` | 监听属性失效（不关注具体值变化） |

- 使用

  ```java
  TextField textField = new TextField();
  
  textField.textProperty().addListener((observable, oldValue, newValue) -> {
      System.out.println("文本从 " + oldValue + " 变为 " + newValue);
  });
  ```

  > 注意:
  >
  > - 监听器是用于监听属性值的，因此使用属性的addListener方法为属性添加监听器。
  > - InvalidationListener常用在不关心新旧值的场景
  > - 监听器接口中只有一个方法，这个方法的三个参数分别是

  - `observable`：被监听的属性
  - `oldValue`：旧值
  - `newValue`：新值



## 属性解绑

> 如果不想让属性继续保持绑定关系，可以将关系解绑。

```
target.textProperty().unbind(); // 取消单向绑定
textField1.textProperty().unbindBidirectional(textField2.textProperty()); // 取消双向绑定
```



# 生命周期

> JavaFX每个窗口对象都具有三个声明周期函数:,init(),start(Stage stage),stop()，其中start()方法是最重要的方法，Application中将其定义为抽象方法，需要使用者实现。

- start

  > 具有一个参数，由虚拟机传入，这是当前的界面对象

# UI线程

> 所有GUI程序中最起码存在两个线程，一个是作为入口的主线程，另一个是专门控制UI界面的UI线程

# Platform

> 一个工具类

- Platform.exit()

# CSS支持

JavaFX具有内置的样式系统，完全支持类似CSS的语法。JavaFX提供了两种方式为控件提供样式:CSS文件与setStyle()方法

- Java文件内设置CSS样式

  ```java
  Button btn = new Button("登录");
  btn.setStyle("-fx-font-size: 16px; -fx-background-color: lightblue;");
  ```

- 独立CSS文件提供样式

  > 类似css语法，通过选择器提供样式

  - CSS文件
  
    ```
    .button {
        -fx-font-size: 16px;
        -fx-background-color: #5cb85c;
        -fx-text-fill: white;
        -fx-padding: 10px 20px;
        -fx-background-radius: 8;
    }
    
    .button:hover {
        -fx-background-color: #4cae4c;
  }
    ```

  - 加载css文件
  
    ```java
    scene.getStylesheets().add(
        getClass().getResource("/css/style.css").toExternalForm()
    );
    ```

## CSS属性

- 通用属性
  - -fx-font-size
  - -fx-background-color,
  - -fx-border-color
- -fx-text-fill
  
- Label
  
    - -fx-font-weight
    
    - -fx-text-fill
## 选择器

通过控件方法可以为控件设置ID或class，从而在css文件中通过css选择器精准控制其样式

```
button.setId("main-btn");
button.getStyleClass().add("rounded-btn");
```

- JavaFX的每一个控件都具有一个默认的类名即Java类名的小写形式，这个类为控件提供JavaFX默认样式


# 图形API

> JavaFX 提供了一套完整的图形绘制功能，核心是 `javafx.scene.shape` 包中的 **Shape API**，可以用它绘制各种图形（如：线段、圆形、矩形、多边形、贝塞尔曲线等）。

- 常用图形

  - `Line`：线段
  - `Rectangle`：矩形
  - `Circle`：圆形
  - `Ellipse`：椭圆
  - `Polygon`：多边形
  - `Polyline`：折线
  - `Arc`：圆弧
  - `Path`：路径（组合线段、曲线）

- 绘制图形

  1. 创建需要绘制的图形对象

     ```java
     Line line = new Line(50, 50, 200, 200);
     Rectangle rect = new Rectangle(100, 100); // 宽 100，高 100
     rect.setX(50);
     rect.setY(50);
     rect.setFill(Color.LIGHTBLUE);
     rect.setStroke(Color.BLACK);  // 边框颜色
     ```

  2. 将图形添加到布局中

     ```java
     Group root = new Group();
     root.getChildren().addAll(line, rect);
     
     Scene scene = new Scene(root, 400, 400);
     primaryStage.setScene(scene);
     primaryStage.show();
     ```

## Path

> Path是最为灵活的图形，可以看作多个线段的组合

```java
Path path = new Path();

MoveTo moveTo = new MoveTo(50, 50);
LineTo line1 = new LineTo(150, 50);
LineTo line2 = new LineTo(150, 150);
ClosePath closePath = new ClosePath();

path.getElements().addAll(moveTo, line1, line2, closePath);
path.setStroke(Color.DARKGREEN);
path.setFill(Color.LIGHTGREEN);
```

## 颜色控制

> JavaFX使用javafx.scence.paint.Color类来表示颜色

- 颜色的定义方式

  - 通过Color提供的静态变量

    > Color提供了一系列静态变量表示常用颜色

    ```
    Color.RED, Color.LIGHTBLUE, Color.YELLOW
    ```

  - 自定义RGB值与透明度

    ```
    Color.rgb(255, 100, 100)
    Color.rgb(0, 0, 0, 0.5)
    ```


## Image

> 表示一张图片资源，可以是本地文件、网络图片、classpath 资源等。

```java
Image image = new Image("file:img/4.jpg");// 绝对/相对路径
Image image1 = new Image("https://example.com/img.png"); // 网络地址
Image image2 = new Image(getClass().getResource("/img/4.jpg").toString());
```

- 对图片进行像素级处理

  ```java
  Image image = new Image("file:src/main/resources/img/1.jpg");
  WritableImage writableImage = new WritableImage((int) image.getWidth(), (int) image.getHeight());
  PixelReader reader = image.getPixelReader();
  PixelWriter writer = writableImage.getPixelWriter();
  
  for (int y = 0; y < image.getHeight(); y++) {
      for (int x = 0; x < image.getWidth(); x++) {
          Color color = reader.getColor(x, y);
          Color gray = color.grayscale(); // 转为灰色
          writer.setColor(x, y, gray);
      }
  }
  gc.drawImage(writableImage, 0, 0);
  ```

## Font

> 用于设置控件（如 Label、Text、Button）上的字体样式，包括字体名、大小、样式等。

- 使用

  ```
  Font font1 = new Font("Arial", 16); // 常规方式
  Font font2 = Font.font("Verdana", FontWeight.BOLD, 18);
  Label label = new Label("你好 JavaFX");
  label.setFont(Font.font("宋体", 24));
  ```

  > 注意:
  >
  > - 也可以通过css的方式设置字体

  ```
  .label-title {
      -fx-font-family: "微软雅黑";
      -fx-font-size: 18px;
      -fx-font-weight: bold;
  }
  ```

- 加载自定义字体

  > 将 `.ttf` 字体文件放到 `resources/fonts/` 目录下，在程序中加载：

  ```java
  Font font = Font.loadFont(getClass().getResourceAsStream("/fonts/custom.ttf"), 20);
  text.setFont(font);
  ```


# FXML

> FX Markup Language，是一种基于XML的语言，用于描述JavaFX应用的界面结构。

- 优点

  1. 界面布局与业务逻辑解耦
  2. 更易维护和设计
  3. 更适合配合设计工具（如 Scene Builder)
  4. 支持事件绑定

- 使用

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  
  <?import javafx.scene.control.*?>
  <?import javafx.scene.layout.*?>
  
  <VBox spacing="10.0" alignment="CENTER" xmlns="http://javafx.com/javafx"
        xmlns:fx="http://javafx.com/fxml" fx:controller="com.example.MyController">
      <Label text="请输入用户名：" />
      <TextField fx:id="usernameField" />
      <Button text="提交" onAction="#handleSubmit"/>
  </VBox>
  
  ```

## FXML与Java控制器绑定

- 注入控件

  ```xml
  <TextField fx:id="usernameField" />
  ```

  ```java
  @FXML
  private TextField usernameField;
  ```

- 指定控制器类

  ```xml
  <VBox fx:controller="com.example.MyController">
  ```

  > 注意:
  >
  > - 控制器应该绑定在根节点上

- 绑定事件

  ```xml
  <Button onAction="#handleSubmit"/>
  ```

  ```java
  @FXML
  private void handleSubmit(ActionEvent event) {
      System.out.println("提交按钮被点击");
  }
  ```

- 加载FXML

  ```java
  FXMLLoader fxmlLoader = new FXMLLoader(HelloApplication.class.getResource("hello-view.fxml"));
  Scene scene = new Scene(fxmlLoader.load(), 320, 240);
  ```

> 注意:
>
> - 引入FXML后，JavaFX应用可以严格按照MVC模式设计，可以单独设置一个控制器类控制逻辑

## 控制器的功能

- 事件处理
- 控件控制与更新
- 数据与模型交互
- 初始化 UI

> 注意:
>
> - 不建议在控制器里操作Stage，如需跳转页面等操作，建议从启动类传入 `Stage` 或通过封装的导航管理类进行操作，避免耦合。
> - 不要在控制器中写数据库操作，建议调用 Service 类执行逻辑

## FXML注解

> @FXML可以应用在方法上与成员变量上，

- 方法上

  > 标明该方法会在FXML文件中调用，如果方法名为initialize，则会在页面加载时自动调用

  ```java
  @FXML
  public void initialize() {
      // 页面加载时自动执行
  }
  ```

- 变量上

  > 自动向变量注入与id与变量名相同的控件，控件id通过`fx:id`设置

  ```java
  @FXML 
  private TextField usernameField;
  ```

  ```xml
  <TextField fx:id="usernameField" layoutX="100" layoutY="120"/>
  ```


# 动画

> JavaFX提供了两类动画:①底层级动画(Timeline)，控制属性随时间变化②高级动画(TranslateTransition，FadeTransition等)，封装常用动画

## Timeline动画

> 随着时间修改节点的某些属性(如透明度，位置，颜色)。

- 使用

  ```java
  Button button = new Button("淡入淡出");
  
  Timeline timeline = new Timeline(
      new KeyFrame(Duration.ZERO, new KeyValue(button.opacityProperty(), 1.0)),
      new KeyFrame(Duration.seconds(1), new KeyValue(button.opacityProperty(), 0.0))
  );
  timeline.setAutoReverse(true);  // 自动反向播放
  timeline.setCycleCount(Timeline.INDEFINITE);  // 无限循环
  timeline.play();
  
  ```

- API

  | 类/方法              | 说明                     |
  | -------------------- | ------------------------ |
  | `Timeline`           | 动画时间线               |
  | `KeyFrame`           | 关键帧，定义时间点和变化 |
  | `KeyValue`           | 属性目标值               |
  | `Duration`           | 时间控制类               |
  | `play()` / `pause()` | 控制动画播放与暂停       |

## `Transition` 动画

> 封装了多种常见的动画效果，这些动画效果均是Transition的子类。

- 使用

  ```java
  //创建一个动画对象并指明动画时间与绑定控件
  TranslateTransition tt = new TranslateTransition(Duration.seconds(2), button);
  //设置动画的属性
  tt.setByX(200);  // 向右移动 200 像素
  tt.setCycleCount(TranslateTransition.INDEFINITE);
  tt.setAutoReverse(true);
  tt.play();
  
  ```

- 动画效果

  |        动画类         |              功能              |
  | :-------------------: | :----------------------------: |
  | `TranslateTransition` |            平移动画            |
  |   `FadeTransition`    |           透明度动画           |
  |   `ScaleTransition`   |            缩放动画            |
  |  `RotateTransition`   |            旋转动画            |
  |   `FillTransition`    | 填充颜色动画（主要用于 Shape） |
  |  `StrokeTransition`   |          边框颜色动画          |

## 组合动画

> 组合动画即将多个动画组合起来展示，分为两种:①顺序组合动画②并发组合动画

- 顺序组合动画

  > 顺序执行多个动画

  ```
  SequentialTransition st = new SequentialTransition(tt, ft, rt);
  st.play();
  ```

- 并发组合动画

  > 动画效果同时独立的展示

  ```
  ParallelTransition pt = new ParallelTransition(tt, ft, rt);
  pt.play();
  ```

## 动画控制

> 动画控制方法适用于所有动画，用于控制动画的行为

|          方法          |                 说明                 |
| :--------------------: | :----------------------------------: |
|        `play()`        |               播放动画               |
|       `pause()`        |                 暂停                 |
|        `stop()`        |              停止并重置              |
|  `setCycleCount(int)`  | 循环次数，例如 `INDEFINITE` 无限循环 |
| `setAutoReverse(true)` |          播放完自动反向播放          |

## 视觉效果

> JavaFX 提供了许多 **视觉特效（Effect）**，可直接应用于任何 `Node`，让控件实现 **发光、阴影、模糊、颜色调整** 等效果。

- 使用

  ```java
  DropShadow shadow = new DropShadow();
  shadow.setOffsetX(5);         // X轴偏移
  shadow.setOffsetY(5);         // Y轴偏移
  shadow.setColor(Color.GRAY);  // 阴影颜色
  
  button.setEffect(shadow);
  ```

- 常用效果

  |     特效类     |              描述              |
  | :------------: | :----------------------------: |
  |     `Glow`     |            发光效果            |
  |  `DropShadow`  |            投影阴影            |
  |    `Bloom`     |          亮度溢出效果          |
  |   `BoxBlur`    |          简单模糊滤镜          |
  | `GaussianBlur` |          高斯模糊滤镜          |
  |  `SepiaTone`   |      老照片滤镜（棕褐色）      |
  | `ColorAdjust`  | 色调、饱和度、亮度、对比度调整 |

- 组合效果

  > 通过effect的setInput方法组合多种视觉效果
  
  ```java
  Glow glow = new Glow(0.5);
  DropShadow shadow = new DropShadow();
  shadow.setInput(glow); // 先应用 glow，再叠加 shadow
  
  button.setEffect(shadow);
  ```
  
  > 注意:
  >
  > - setInput可以链式调用

## AnimationTimer

> AnimationTimer是每帧调用一次的动画类，本质是一个定时器，每一帧触发一次该类的handle方法，用来，JavaFX中一秒有60帧。

- 使用

  ```java
  AnimationTimer timer = new AnimationTimer() {
      double dx = 3, dy = 2;
  
      @Override
      public void handle(long now) {
          circle.setLayoutX(circle.getLayoutX() + dx);
          circle.setLayoutY(circle.getLayoutY() + dy);
      }
  };
  timer.start();
  ```

  > 注意:
  >
  > - handle方法的参数now表示从JVM启动到该方法调用时经过的纳秒数，可以用来计算每一帧的间隔，一般第0帧到第一帧的间隔会很长，会直接丢弃不用。

# 多线程

> JavaFX是单线程UI框架，所有UI更新操作都必须在JavaFX Application Thread中完成。如果直接在UI线程中进行一些阻塞操作时，如I/O操作，网络请求等，会导致界面卡顿甚至冻结，因此需要后台线程来处理耗时任务。

JavaFX提供了专门的API来支持UI与后台线程的配合:

| 类/接口                       | 用途                                                  |
| ----------------------------- | ----------------------------------------------------- |
| `Task<V>`                     | 可观测的后台任务，继承自 `FutureTask`，适合一次性任务 |
| `Service<V>`                  | 更高级的可重用任务封装，适合重复执行的任务            |
| `Platform.runLater(Runnable)` | 从后台线程切回 UI 线程，进行UI 更新、异常处理等       |

## Task

- 使用

  ```java
  Task<String> task = new Task<>() {
      @Override
      protected String call() throws Exception {
          Thread.sleep(3000); // 模拟耗时任务
          return "读取完成";
      }
  };
  
  // UI绑定
  task.setOnSucceeded(event -> {
      String result = task.getValue();
      label.setText(result); // 安全地更新UI
  });
  task.setOnFailed(e -> label.setText("读取失败"));
  
  // 启动任务
  new Thread(task).start();
  
  ```

- API

  |       方法       |              描述              |
  | :--------------: | :----------------------------: |
  |    getValue()    |          获取任务结果          |
  | updateMessage()  | 在后台线程中向 UI 线程传递数据 |
  | updateProgress() | 在后台线程中向 UI 线程传递数据 |

## Service

> `Service` 是 `Task` 的封装器，适合需要 **多次执行** 的后台任务。

- 使用

  ```java
  Service<String> queryService = new Service<>() {
      @Override
      protected Task<String> createTask() {
          return new Task<>() {
              @Override
              protected String call() throws Exception {
                  Thread.sleep(2000);
                  return "查询结果";
              }
          };
      }
  };
  
  // 成功后更新UI
  queryService.setOnSucceeded(e -> label.setText(queryService.getValue()));
  
  // 点击按钮时启动
  button.setOnAction(e -> {
      if (!queryService.isRunning()) {
          queryService.restart(); // 每次都重新启动新任务
      }
  });
  
  ```

## 属性绑定

|                           |                  |
| :-----------------------: | :--------------: |
| `task.progressProperty()` | 绑定 ProgressBar |
| `task.runningProperty()`  | 绑定按钮可用状态 |
| `task.messageProperty()`  |    绑定 Label    |



# 小知识

- StringExpression对象的contact()

  > 可以向StringExpression对象字符串的末尾拼接字符串

- Binds.contact()

  > 可以接收任意参数，拼接为StringExpression

## CellFactory

> 在 JavaFX 中，`CellFactory`（单元格工厂）用于自定义列表控件（如 `ListView`、`TableView`、`TreeView`、`ComboBox` 等）中每一项的显示方式。

- 使用

  ```java
  TableView<Person> tableView = new TableView<>();
  TableColumn<Person, String> nameCol = new TableColumn<>("姓名");
  nameCol.setCellValueFactory(new PropertyValueFactory<>("name"));
  
  // 自定义每个单元格的显示样式
  nameCol.setCellFactory(column -> new TableCell<Person, String>() {
      @Override
      protected void updateItem(String item, boolean empty) {
          super.updateItem(item, empty);
          if (empty || item == null) {
              setText(null);
              setGraphic(null);
          } else {
              Label label = new Label("👤 " + item);
              label.setStyle("-fx-font-weight: bold;");
              setGraphic(label);
          }
      }
  });
  
  ```

  > 注意:
  >
  > - 默认情况下，列表控件会调用对象的tostring()方法来显示文本，通过CellFactory，不仅可以自定义数据显示，而且可以在列表项中显示复杂控件。

- API

  | 关键点           | 说明                                                |
  | ---------------- | --------------------------------------------------- |
  | `setCellFactory` | 设置一个工厂函数，每个 Cell（单元格）都会使用它生成 |
  | `updateItem`     | 更新 Cell 内容的核心方法                            |
  | `setGraphic()`   | 用于设置自定义组件（如 VBox、HBox、ImageView 等）   |
  | 适用控件         | `ListView`、`TableView`、`ComboBox`、`TreeView` 等  |

## SDK

**软件开发工具包**(`Software Development Kit`)是由平台提供方发布的一组 **软件开发工具的集合**，用于帮助开发者 **快速、正确地开发基于该平台或服务的应用程序**。