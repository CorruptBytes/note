# 概述

> Swing是后来Java提供的更加高级的GUI库

# AWT

> AWT全称`Abstract Window Toolkit`，抽象窗口工具集，是JDK1.0发布时提供的GUI库，提供了GUI编程的基本功能。AWT并未提供GUI组件的实现(AWT名字中抽象的体现)，而是将组件的创建和动作委托给操作系统。使用AWT编写图形界面时，程序仅指定界面组件的位置和行为，JVM通过调用操作系统本地的图形界面系统来创建和平台风格一致的组件。Java希望采用这种方式实现"Write once, Run Anywhere"的目标。

## AWT继承体系

![img](file://C:/Users/80468/Desktop/fsdownload/java%20GUI%E7%95%8C%E9%9D%A2%E7%BC%96%E7%A8%8B%E8%B5%84%E6%96%99/%E6%96%87%E6%A1%A3/images/AWT%E7%BB%84%E4%BB%B6%E7%BB%A7%E6%89%BF%E4%BD%93%E7%B3%BB.png?lastModify=1747997063)

> 注意:
>
> - Container 是一种特殊的 Component，它代表一种容器，可以盛装普通的 Component。
> - LayoutManager负责管理组件的布局方式，每一种容器都具有默认的布局管理器管理其子组件的布局

## Component

> Component是所有窗口组件的父类，提供了通用的方法设置组件的大小，位置，可见性

- 通用API

|                    方法签名                    |          方法功能          |
| :--------------------------------------------: | :------------------------: |
|           setLocation(int x, int y)            |      设置组件的位置。      |
|         setSize(int width, int height)         |      设置组件的大小。      |
| setBounds(int x, int y, int width, int height) | 同时设置组件的位置、大小。 |
|             setVisible(Boolean b):             |    设置该组件的可见性。    |

> 注意:
>
> - 默认GUI窗口的左上角为原点，向右为x轴正方向，向下为y轴正方向。AWT基于这个坐标系设置组件的位置
> - setVisible要在所有组件均布局完成后调用，否则会出现组件无法显示的问题

- 常用子类组件

  |    组件名     |                             功能                             |
  | :-----------: | :----------------------------------------------------------: |
  |    Button     |                            Button                            |
  |    Canvas     |                        用于绘图的画布                        |
  |   Checkbox    |             复选框组件（也可当做单选框组件使用）             |
  | CheckboxGroup | 用于将多个Checkbox 组件组合成一组， 一组 Checkbox 组件将只有一个可以 被选中 ， 即全部变成单选框组件 |
  |    Choice     |                          下拉选择框                          |
  |     Frame     |            窗口 ， 在 GUI 程序里通过该类创建窗口             |
  |     Label     |                  标签类，用于放置提示性文本                  |
  |     List      |                 JU表框组件，可以添加多项条目                 |
  |     Panel     |          不能单独存在基本容器类，必须放到其他容器中          |
  |   Scrollbar   | 滑动条组件。如果需要用户输入位于某个范围的值 ， 就可以使用滑动条组件 ，比如调 色板中设置 RGB 的三个值所用的滑动条。当创建一个滑动条时，必须指定它的方向、初始值、 滑块的大小、最小值和最大值。 |
  |  ScrollPane   |                 带水平及垂直滚动条的容器组件                 |
  |   TextArea    |                          多行文本域                          |
  |   TextField   |                          单行文本框                          |

### Container容器

> Container 是一种特殊的 Component，它代表一种容器，可以盛装普通的 Component。

![img](file://C:/Users/80468/Desktop/fsdownload/java%20GUI%E7%95%8C%E9%9D%A2%E7%BC%96%E7%A8%8B%E8%B5%84%E6%96%99/%E6%96%87%E6%A1%A3/images/Container%E7%BB%A7%E6%89%BF%E4%BD%93%E7%B3%BB.png?lastModify=1747997063)

> 注意:
>
> - Winow是可以独立存在的顶级窗口,默认使用BorderLayout管理其内部组件布局;
> - Panel可以容纳其他组件，但不能独立存在，它必须内嵌其他容器中使用，默认使用FlowLayout管理其内部组件布局；
> - ScrollPane 是 一个带滚动条的容器，它也不能独立存在，默认使用 BorderLayout 管理其内部组件布局；

- API

  |                 方法签名                 |                           方法功能                           |
  | :--------------------------------------: | :----------------------------------------------------------: |
  |      Component add(Component comp)       | 向容器中添加其他组件 (该组件既可以是普通组件，也可以 是容器) ， 并返回被添加的组件 。 |
  | Component getComponentAt(int x, int y):  |                     返回指定点的组件 。                      |
  |         int getComponentCount():         |                  返回该容器内组件的数量 。                   |
  |       Component[] getComponents():       |                  返回该容器内的所有组件 。                   |
  | public void setLayout(LayoutManager mgr) |                     设置容器的布局管理器                     |

#### Window

- API

  |        方法        |                      作用                      |
  | :----------------: | :--------------------------------------------: |
  | public void pack() | 使窗口根据子组件的布局与大小自动调整为最佳大小 |
  |                    |                                                |
  |                    |                                                |

##### Frame

> Frame是属于Window的一种窗口

##### Dialog对话框

> Window的子类，是可以独立存在的顶级窗口，但是一般会使其依赖于一个父窗口。Dialog分为非模式对话框(non-modal)和模式对话框(modal),模式对话框总是位于父窗口之上，且被关闭之前，父窗口无法获得焦点。

|                     方法名称                     |                           方法功能                           |
| :----------------------------------------------: | :----------------------------------------------------------: |
| Dialog(Frame owner, String title, boolean modal) | 创建一个对话框对象：<br/>owner:当前对话框的父窗口<br/>title:当前对话框的标题<br/>modal：当前对话框是否是模式对话框，true/false |

- 文件对话框FileDialog

  > 用于打开或保存文件，无法指定模态或非模态，依赖于当前的运行平台。

  |                     方法名称                     |                           方法功能                           |
  | :----------------------------------------------: | :----------------------------------------------------------: |
  | FileDialog(Frame parent, String title, int mode) | 创建一个文件对话框：<br/>parent:指定父窗口<br/>title:对话框标题<br/>mode:文件对话框类型，如果指定为FileDialog.load，用于打开文件，如果指定为FileDialog.SAVE,用于保存文件 |
  |              String getDirectory()               |                获取被打开或保存文件的绝对路径                |
  |                 String getFile()                 |                 获取被打开或保存文件的文件名                 |

#### Panel

> Panel不能独立存在，必须为其指定父容器

#### ScrollPane

> ScrollPane同样无法独立存在，必须为其指定父容器

## LayoutManager

> 通过布局管理器，可以使组件根据不同的平台来自动调整大小，从而实现良好的跨平台特性。Java提供了五种布局管理器的实现

![img](file://C:/Users/80468/Desktop/fsdownload/java%20GUI%E7%95%8C%E9%9D%A2%E7%BC%96%E7%A8%8B%E8%B5%84%E6%96%99/%E6%96%87%E6%A1%A3/images/%E5%B8%B8%E7%94%A8%E5%B8%83%E5%B1%80%E7%AE%A1%E7%90%86%E5%99%A8.png?lastModify=1747997063)

### FlowLayout

> 流式布局，组件像流水一样向某方向流动(排列)，遇到障碍(边界)就折回，从头开始排列。默认情况下，从左向右排列所有组件，遇到边界就折回从下一行重新开始。

- API

  |                构造方法                 |                           方法功能                           |
  | :-------------------------------------: | :----------------------------------------------------------: |
  |              FlowLayout()               | 使用默认 的对齐方式及默认的垂直间距、水平间距创建 FlowLayout 布局管理器。 |
  |          FlowLayout(int align)          | 使用指定的对齐方式及默认的垂直间距、水平间距创建 FlowLayout 布局管理器。 |
  | FlowLayout(int align,int hgap,int vgap) | 使用指定的对齐方式及指定的垂直问距、水平间距创建FlowLayout 布局管理器。 |

  > 注意:
  >
  > - align为排列方式，通过Flowout类的静态常量定义：FlowLayout.LEFT(从左向右),FlowLayout.CENTER(从中间向两边),Flowout.RIGHT（从右向左)

### BorderLayout

> 将容器分为East，south，west，north，center五个区域，子组件可以被放置在这五个区域的任意一个

![img](file://C:/Users/80468/Desktop/fsdownload/java%20GUI%E7%95%8C%E9%9D%A2%E7%BC%96%E7%A8%8B%E8%B5%84%E6%96%99/%E6%96%87%E6%A1%A3/images/BorderLayout.png?lastModify=1747997063)

> 注意:
>
> - 改变容器大小时，NORTH，SOUTH，CENTER区域会水平调整，EAST，WEST和CENTER会垂直调整。
> - 向容器中添加组件时，要指定添加到哪个区域中，否则默认添加到中间区域。通过add方法中的constraints参数指定添加区域，参数为BorderLayout中的静态常量
> - 向同一个区域添加多个组件时，后放入的组件会覆盖先放入的组件。
> - 如果某个区域没有放置组件，则该区域会被CENTER区域占据

- API

  | 构造方法                         | 方法功能                                                     |
  | -------------------------------- | ------------------------------------------------------------ |
  | BorderLayout()                   | 使用默认的水平间距、垂直 间距创建 BorderLayout 布局管理器 。 |
  | BorderLayout(int hgap,int vgap): | 使用指定的水平间距、垂直间距创建 BorderLayout 布局管理器。   |

### GridLayout

> 网格布局会将容器分隔成纵横线分隔的网格，每个网格占据区域大小相同。默认从左向右，从上到下依次向每个网格中添加组件，每个组件将自动占满整个网格区域

|                    构造方法                     |                           方法功能                           |
| :---------------------------------------------: | :----------------------------------------------------------: |
|         GridLayout(int rows,in t cols)          | 采用指定的行数 、列数，以及默认的横向间距、纵向间距将容器 分割成多个网格 |
| GridLayout(int rows,int cols,int hgap,int vgap) | 采用指定 的行数、列 数 ，以及指定的横向间距 、 纵向间距将容器分割成多个网格。 |

### GridBagLayout

> 是Java中最强大也是最复杂的布局管理器，在网格包布局中，父容器仍然被分为多个网格，但是网格可以设置不同大小，并且一个组件可以占据多个网格。

![img](file://C:/Users/80468/Desktop/fsdownload/java%20GUI%E7%95%8C%E9%9D%A2%E7%BC%96%E7%A8%8B%E8%B5%84%E6%96%99/%E6%96%87%E6%A1%A3/images/GridBagLayout.png?lastModify=1747997063)

> 注意:
>
> - GridBagLayout使用时复杂繁琐，一般不会使用它，而是使用Swing中更强大的布局管理

### CardLayout

> CardLayout以时间而非空间管理组件，将加入容器的所有组件看作一叠卡片，每次只会显示最上面的组件。

| 方法名称                          | 方法功能                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| CardLayout()                      | 创建默认的 CardLayout 布局管理器。                           |
| CardLayout(int hgap,int vgap)     | 通过指定卡片与容器左右边界的间距 C hgap) 、上下边界 Cvgap) 的间距来创建 CardLayout 布局管理器. |
| first(Container target)           | 显示target 容器中的第一张卡片.                               |
| last(Container target)            | 显示target 容器中的最后一张卡片.                             |
| previous(Container target)        | 显示target 容器中的前一张卡片.                               |
| next(Container target)            | 显示target 容器中的后一张卡片.                               |
| show(Container taget,String name) | 显 示 target 容器中指定名字的卡片.                           |

## 菜单组件

> 菜单组件分为MenuBar(菜单条)，MenuItem(菜单项)两类，菜单条是菜单项的容器，同时Menu菜单项也可以作为其他菜单项的容器。

![img](file://G:/java%20GUI%E7%95%8C%E9%9D%A2%E7%BC%96%E7%A8%8B%E8%B5%84%E6%96%99/%E6%96%87%E6%A1%A3/images/%E8%8F%9C%E5%8D%95%E9%A1%B9%E7%BB%84%E4%BB%B6%E7%BB%A7%E6%89%BF%E4%BD%93%E7%B3%BB.png?lastModify=1748052235)

|   菜单组件名称   |                             功能                             |
| :--------------: | :----------------------------------------------------------: |
|     MenuBar      |                   菜单条 ， 菜单的容器 。                    |
|       Menu       | 菜单组件 ， 菜单项的容器 。 它也是Menultem的子类 ，所以可作为菜单项使用 |
|    PopupMenu     |                 上下文菜单组件(右键菜单组件)                 |
|     Menultem     |                        菜单项组件 。                         |
| CheckboxMenuItem |                       复选框菜单项组件                       |

- 使用

  1.准备菜单项组件，这些组件可以是MenuItem及其子类对象

  2.准备菜单组件Menu或者PopupMenu(右击弹出子菜单)，把第一步中准备好的菜单项组件添加进来；

  3.准备菜单条组件MenuBar，把第二步中准备好的菜单组件Menu添加进来；

  4.把第三步中准备好的菜单条组件通过setMenuBar()方法添加到窗口对象中显示。

> 注意:
>
> - .如果要在某个菜单的菜单项之间添加分割线，那么只需要调用Menu的add（new MenuItem(-)）即可。
> - 如果要给某个菜单项关联快捷键功能，那么只需要在创建菜单项对象时设置即可，例如给菜单项关联 ctrl+shif+/ 快捷键，只需要：new MenuItem("菜单项名字",new MenuShortcut(KeyEvent.VK_Q,true);

## 事件处理

> 当在组件上发生某些操作时，会自动触发一段代码。在GUI事件处理机制时，包含四部分:

- **事件源(Event Source)**：操作发生的场所，通常指某个组件，例如按钮、窗口等；
- **事件（Event）**：在事件源上发生的操作可以叫做事件，GUI会把事件都封装到一个Event对象中，如果需要知道该事件的详细信息，就可以通过Event对象来获取。
- **事件监听器(Event Listener)**:当在某个事件源上发生了某个事件，事件监听器就可以对这个事件进行处理。
- **注册监听**：把某个事件监听器(A)通过某个事件(B)绑定到某个事件源(C)上，当在事件源C上发生了事件B之后，那么事件监听器A的代码就会自动执行。

![img](file://G:/java%20GUI%E7%95%8C%E9%9D%A2%E7%BC%96%E7%A8%8B%E8%B5%84%E6%96%99/%E6%96%87%E6%A1%A3/images/%E4%BA%8B%E4%BB%B6%E5%A4%84%E7%90%86%E6%9C%BA%E5%88%B6.png?lastModify=1748052235)

- 使用
  1. 自定义类，实现xxxListener接口，重写抽象方法
  2. 创建事件监听器对象
  3. 调用组件的addxxxListener()方法为组件注册监听

### 事件

> 事件即在组件上发生的动作，AWT 的事件类都是 AWTEvent 类的子类 ， AWTEvent是 EventObject 的子类。

​		**AWT把事件分为两大类:**

1. 低级事件：这类事件是基于某个特定动作的事件。比如进入、点击、拖放等动作的鼠标事件，再比如得到焦点和失去焦点等焦点事件。

   |      事件      |                           触发时机                           |
   | :------------: | :----------------------------------------------------------: |
   | ComponentEvent | 组件事件 ， 当 组件尺寸发生变化、位置发生移动、显示/隐藏状态发生改变时触发该事件。 |
   | ContainerEvent |  容器事件 ， 当容器里发生添加组件、删除组件时触发该事件 。   |
   |  WindowEvent   | 窗口事件， 当窗 口状态发生改变 ( 如打开、关闭、最大化、最 小化)时触发该事件 。 |
   |   FocusEvent   |     焦点事件 ， 当组件得到焦点或失去焦点 时触发该事件 。     |
   |    KeyEvent    |      键盘事件 ， 当按键被按下、松开、单击时触发该事件。      |
   |   MouseEvent   | 鼠标事件，当进行单击、按下、松开、移动鼠标等动作 时触发该事件。 |
   |   PaintEvent   | 组件绘制事件 ， 该事件是一个特殊的事件类型 ， 当 GUI 组件调 用 update/paint 方法 来呈现自身时触发该事件，该事件并非专用于事件处理模型 。 |

2. 高级事件:这类事件并不会基于某个特定动作，而是根据功能含义定义的事件。

   | 事件           | 触发时机                                                     |
   | -------------- | ------------------------------------------------------------ |
   | ActionEvent    | 动作事件 ，当按钮、菜单项被单击，在 TextField 中按 Enter 键时触发 |
   | AjustmentEvent | 调节事件，在滑动条上移动滑块以调节数值时触发该事件。         |
   | ltemEvent      | 选项事件，当用户选中某项， 或取消选中某项时触发该事件 。     |
   | TextEvent      | 文本事件， 当文本框、文本域里的文本发生改变时触发该事件。    |

### 常用事件监听器

> AWT 提供了大量的事件监听器接口用于实现不同类型的事件监听器，监听不同类型的事件。当事件发生后，会自动调用事件监听器中的处理方法处理事件。

|    事件类别     |         描述信息         |    监听器接口名     |
| :-------------: | :----------------------: | :-----------------: |
|   ActionEvent   |         激活组件         |   ActionListener    |
|    ItemEvent    |      选择了某些项目      |    ItemListener     |
|   MouseEvent    |         鼠标移动         | MouseMotionListener |
|   MouseEvent    |        鼠标点击等        |    MouseListener    |
|    KeyEvent     |         键盘输入         |     KeyListener     |
|   FocusEvent    |    组件收到或失去焦点    |    FocusListener    |
| AdjustmentEvent |    移动了滚动条等组件    | AdjustmentListener  |
| ComponentEvent  |  对象移动缩放显示隐藏等  |  ComponentListener  |
|   WindowEvent   |    窗口收到窗口级事件    |   WindowListener    |
| ContainerEvent  |   容器中增加删除了组件   |  ContainerListener  |
|    TextEvent    | 文本字段或文本区发生改变 |    TextListener     |

- 适配器

  > Java为每一种事件监听器都提供了对应的适配器xxxAdapter,它实现了监听器接口，并且对于事件监听器接口的方法都进行了空实现，用户可以根据自己的需求重写对应的方法，而不必重写接口中的所有方法。

## 绘图

> 组件的外观正是通过绘图功能绘制。在AWT中，提供绘图功能的是Graphics对象，在组件中，提供了三个方法完成组件图形的绘制与刷新

```java
paint(Graphics g)://绘制组件的外观；

update(Graphics g)://内部调用paint方法，刷新组件外观；

repaint()://调用update方法，刷新组件外观；
```

![img](file://G:/java%20GUI%E7%95%8C%E9%9D%A2%E7%BC%96%E7%A8%8B%E8%B5%84%E6%96%99/%E6%96%87%E6%A1%A3/images/%E7%BB%84%E4%BB%B6%E7%BB%98%E5%88%B6%E5%9B%BE%E5%BD%A2%E6%B5%81%E7%A8%8B.png?lastModify=1748052235)



> 注意:
>
> - 在程序中，如果需要自己画图，一般调用reapint()方法重新绘图

### Grahpics对象

> 是AWT用于画图的画笔，是绘图的核心。画图之前，需要调用setColor(),setFont()方法设置画笔的属性

- API

  |      方法名称      |        方法功能        |
  | :----------------: | :--------------------: |
  | setColor(Color c)  |        设置颜色        |
  | setFont(Font font) |        设置字体        |
  |     drawLine()     |        绘制直线        |
  |     drawRect()     |        绘制矩形        |
  |  drawRoundRect()   |      绘制圆角矩形      |
  |     drawOval()     |       绘制椭圆形       |
  |   drawPolygon()    |       绘制多边形       |
  |     drawArc()      |        绘制圆弧        |
  |   drawPolyline()   |        绘制折线        |
  |     fillRect()     |      填充矩形区域      |
  |  fillRoundRect()   |    填充圆角矩形区域    |
  |     fillOval()     |      填充椭圆区域      |
  |   fillPolygon()    |     填充多边形区域     |
  |     fillArc()      | 填充圆弧对应的扇形区域 |
  |    drawImage()     |        绘制位图        |

### 绘图的步骤

> 在AWT中，提供Canvas类充当画布，Grapics类充当画笔，使用画笔在画布上作画。

1. 自定义类，继承Canvas类，重写paint(Graphics g)方法完成画图；

2. 在paint方法内部，真正开始画图之前调用Graphics对象的setColor()、setFont()等方法设置画笔的颜色、字体等属性；

3. 调用Graphics画笔的drawXxx()方法开始画图。


### 绘制位图

> 仅仅绘制简单的几何图形，程序的美化效果很单一。因此，AWT提供了在组件绘制位图的方法:drawImage(Image image)。

- 使用
  1. 创建Image的子类对象BufferedImage(int width,int height,int ImageType),创建时需要指定位图的宽高及类型属性；此时相当于在内存中生成了一张图片；
  2. 调用BufferedImage对象的getGraphics()方法获取画笔，此时就可以往内存中的这张图片上绘图了，绘图的方法和之前学习的一模一样；
  3. 调用组件的drawImage()方法，一次性的内存中的图片BufferedImage绘制到特定的组件上。
- 位图的优点
  1. 使用位图来绘制组件，相当于实现了图的缓冲区，此时绘图时没有直接把图形绘制到组件上，而是先绘制到内存中的BufferedImage上，等全部绘制完毕，再一次性的图像显示到组件上即可，这样用户的体验会好一些。

### ImageIO

> 在AWT中，可以通过ImageIO类读取图片文件并显示。

|                           方法名称                           |         方法功能         |
| :----------------------------------------------------------: | :----------------------: |
|            static BufferedImage read(File input)             |   读取本地磁盘图片文件   |
|         static BufferedImage read(InputStream input)         |   读取本地磁盘图片文件   |
| static boolean write(RenderedImage im, String formatName, File output) | 往本地磁盘中输出图片文件 |

# Swing

> Swing是由纯Java实现，不再依赖于本地平台的GUI，可以在所有平台上保持相同的外观。独立于本地平台的Swing组件被称为轻量级组件，不依赖于本地平台的AWT组件被称为重量级组件。

- Swing优势

  1. Swing 组件不再依赖于本地平台的 GUI，无须采用各种平台的 GUI 交集 ，因此 Swing 提供了大量图形界面组件 ， 远远超出了 AWT 所提供的图形界面组件集。
  2. Swing 组件不再依赖于本地平台 GUI ，因此不会产生与平台 相关的 bug 。
  3.  Swing 组件在各种平台上运行时可以保证具有相同的图形界面外观。

- Swing劣势

  Swing的组件完全由自己实现，相比调用本地平台GUI实现组件，速度会慢一些。 

- Swing特点

  1. 采用MVC(Model-View-Controller)设计模式

        	1. 

     	模型(Model): 用于维护组件的各种状态；
     	
     	视图(View): 是组件的可视化表现；
     		
     	控制器(Controller):用于控制对于各种事件、组件做出响应 。
     		
     		当模型发生改变时，它会通知所有依赖它的视图，视图会根据模型数据来更新自己。Swing使用UI代理来包装视图和控制器， 还有一个模型对象来维护该组件的状态。例如，按钮JButton有一个维护其状态信息的模型ButtonModel对象 。 Swing组件的模型是自动设置的，因此一般都使用JButton，而无须关心ButtonModel对象。
     	
     	2. Swing在不同的平台上表现一致，并且有能力提供本地平台不支持的显示外观 。由于 Swing采用 MVC 模式来维护各组件，所以 当组件的外观被改变时，对组件的状态信息(由模型维护)没有任何影响 。因 此，Swing可以使用插拔式外观感觉 (Pluggable Look And Feel, PLAF)来控制组件外观，使得 Swing图形界面在同一个平台上运行时能拥有不同的外观，用户可以选择自己喜欢的外观 。相比之下，在 AWT 图形界面中，由于控制组件外观的对等类与具体平台相关 ，因此 AWT 组件总是具有与本地平台相同的外观 。    


![](D:\笔记\图片\Swing组件继承体系.png)

- Swing组件分类
  - 根据功能
    1. 顶层容器: JFrame、JApplet、JDialog 和 JWindow 。
    2. 中间容器: JPanel 、 JScrollPane 、 JSplitPane 、 JToolBar 等 。
    3. 特殊容器:在用户界面上具有特殊作用的中间容器，如 JIntemalFrame 、 JRootPane 、 JLayeredPane和 JDestopPane 等 。
    4. 基本组件 : 实现人机交互的组件，如 JButton、 JComboBox 、 JList、 JMenu、 JSlider 等 
    5. 不可编辑信息的显示组件:向用户显示不可编辑信息的组件，如JLabel 、 JProgressBar 和 JToolTip等。
    6. 可编辑信息的显示组件:向用户显示能被编辑的格式化信息的组件，如 JTable 、 JTextArea 和JTextField 等 。
    7. 特殊对话框组件:可以直接产生特殊对话框的组件 ， 如 JColorChooser 和 JFileChooser 等。

- AWT组件的Swing实现

  > Swing中有许多由Java实现的与AWT中功能相同的组件，但是Swing组件功能更强大。

  1. 可以为 Swing 组件设置提示信息。使用 setToolTipText()方法，为组件设置对用户有帮助的提示信息 。
  2. 很多 Swing 组件如按钮、标签、菜单项等，除使用文字外，还可以使用图标修饰自己。为了允许在 Swing 组件中使用图标， Swing为Icon 接口提供了 一个实现类: Imagelcon ，该实现类代表一个图像图标。
  3. 支持插拔式的外观风格。每个 JComponent 对象都有一个相应的 ComponentUI 对象，为它完成所有的绘画、事件处理、决定尺寸大小等工作。 ComponentUI 对象依赖当前使用的 PLAF ， 使用 UIManager.setLookAndFeel()方法可以改变图形界面的外观风格 。
  4. 支持设置边框。Swing 组件可以设置一个或多个边框。 Swing 中提供了各式各样的边框供用户边 用，也能建立组合边框或自己设计边框。 一种空白边框可以用于增大组件，同时协助布局管理器对容器中的组件进行合理的布局。   

- UI类

  > 每个Swing组件都有一个对应的UI类代理其UI实现，如JButton对应ButtonUI。UI代理类通常是一个抽象基类 ， 不同的 PLAF 会有不同的UI代理实现类 。 Swing 类库中包含了几套UI代理,分别放在不同的包下， 每套UI代理都几乎包含了所有 Swing组件的 ComponentUI实现，每套这样的实现都被称为一种PLAF 实现 。以 JButton 为例，其 UI 代理的继承层次下图：

  ![](D:\笔记\图片\ComponentUI.png)

  ```java
  //容器：
  JFrame jf = new JFrame();
  
  try {
  
      //设置外观风格
      UIManager.setLookAndFeel("com.sun.java.swing.plaf.windows.WindowsLookAndFeel");
  
      //刷新jf容器及其内部组件的外观
      SwingUtilities.updateComponentTreeUI(jf);
  } catch (Exception e) {
      e.printStackTrace();
  }
  ```

## Menu

- 为MenuItem添加分割线

  调用Menu对象的addSeparator()方法

## Container

### Box

> Swing中提供的新的容器，默认使用BoxLayout布局管理器，大多数情况下，使用Box容器容纳多个GUI组件，然后把Box容器作为组件添加到其他容器中，实现整体的窗口布局。

|             方法名称             |              方法功能              |
| :------------------------------: | :--------------------------------: |
| static Box createHorizontalBox() | 创建一个水平排列组件的 Box 容器 。 |
|  static Box createVerticalBox()  | 创建一个垂直排列组件的 Box 容器 。 |

- Box间隔组件

  > 默认情况下，Box内的多个组件是没有间隔的，如果想要添加间隔，可以在子组件之间添加间隔组件，该组件紧起分隔作用。

  |                     方法名称                      |                           方法功能                           |
  | :-----------------------------------------------: | :----------------------------------------------------------: |
  |      static Component createHorizontalGlue()      |       创建一条水平 Glue (可在两个方向上同时拉伸的间距)       |
  |       static Component createVerticalGlue()       |      创建一条垂直 Glue (可在两个方向上同时拉伸的间距）       |
  | static Component createHorizontalStrut(int width) | 创建一条指定宽度(宽度固定了，不能拉伸)的水平Strut (可在垂直方向上拉伸的间距) |
  | static Component createVerticalStrut(int height)  | 创建一条指定高度(高度固定了，不能拉伸)的垂直Strut (可在水平方向上拉伸的间距) |

## LayoutManager

### BoxLayout

> Swing引入的新的布局管理器，允许在垂直和水平两个方向上摆放GUI组件，

|               方法名称                |                           方法功能                           |
| :-----------------------------------: | :----------------------------------------------------------: |
| BoxLayout(Container target, int axis) | 指定创建基于 target 容器的 BoxLayout 布局管理器，该布局管理器里的组件按 axis 方向排列。其中 axis 有 BoxLayout.X_AXIS( 横向)和 BoxLayout.Y _AXIS (纵向〉两个方向。 |

## Timer

> 定时器类，可以设置定时器，为定时器绑定一个事件监听器，每隔一段时间触发事件。

|                                           |                                                              |
| :---------------------------------------: | :----------------------------------------------------------: |
| Timer(int delay, ActionListener listener) | 每间隔 delay 毫秒，系统自动触发 ActionListener 监听器里的事件处理器方法 |

## 边框

> 为组件增加边框可使其更加具有设计感，swing中提供了Border对象来代表边框

![](D:\笔记\图片\Border继承体系.png)

> 注意:
>
> - TitledBorder:它的作用并不是直接为其他组件添加边框，而是为其他边框设置标题，创建该类的对象时，需要传入一个其他的Border对象；
> -  ComoundBorder:用来组合其他两个边框，创建该类的对象时，需要传入其他两个Border对象，一个作为内边框，一个座位外边框

- 为组件设置边框
  1. 使用BorderFactory或者XxxBorder创建Border的实例对象；
  2. 调用Swing组件的setBorder（Border b）方法为组件设置边框；

## JToolBar

> 工具条组件，可以添加多个工具按钮

|                 方法名称                 |                           方法功能                           |
| :--------------------------------------: | :----------------------------------------------------------: |
| JToolBar( String name , int orientation) | 创建一个名字为name，方向为orientation的工具条对象，其orientation的是取值可以是SwingConstants.HORIZONTAL或SwingConstants.VERTICAL |
|          JButton add(Action a)           |       通过Action对象为JToolBar工具条添加对应的工具按钮       |
|      addSeparator( Dimension size )      |                向工具条中添加指定大小的分隔符                |
|        setFloatable( boolean b )         |                   设定工具条是否可以被拖动                   |
|           setMargin(Insets m)            |                  设置工具条与工具按钮的边距                  |
|         setOrientation( int o )          |                       设置工具条的方向                       |
|      setRollover(boolean rollover)       |                  设置此工具条的rollover状态                  |

> 注意:
>
> - Action为ActionListener的子类，通过它，可以同时设置按钮的动作与样式

- 调用add(Action a)时发生的事情
  1. 创建一个适用于该容器的组件(例如，在工具栏中创建一个工具按钮)；
  2. 从 Action 对象中获得对应的属性来设置该组件(例如，通过 name 来设置文本，通过 lcon 来设置图标) ；
  3. 把Action监听器注册到刚才创建的组件上；

# 小知识

## 改变虚拟机编码

添加虚拟机选项

```
-Dfile.encoding=GBK
```

## 关闭程序

调用System.exit(0)关闭Java虚拟机

```
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
```