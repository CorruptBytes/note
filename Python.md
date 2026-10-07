# 魔术方法

## `__call__()`

> 实现call方法后，对象可以当作函数一样调用。当调用对象时，会调用对象的call方法

## 运算符

### 三元运算符

```python
#如果boolen为True,则表达式值为x，否则为y
x if boolen else y
```

# 装饰器

装饰器是 Python 中一种非常重要的特性，它允许在**不修改原函数或原类代码的前提下，动态地为其添加功能**。

<h3>基本语法</h3>

装饰器本质是一个函数，它接收目标函数/类作为参数，并返回增强后的函数/类。其核心就是利用闭包，把原函数或类包装成一个功能更强的新函数或新类，并让函数名/类名指向这个新函数/新类。

```python
# 定义装饰器函数，其最终返回一个函数
def log(func):

    def wrapper():
        print("开始执行")

        func()

        print("执行结束")

    return wrapper

# 调用装饰器获取增强后的函数
目标 = 装饰器(目标)
```

<h3>装饰器语法糖</h3>

```python
@装饰器
def hello():
    print("Hello")


```

<h3>带参数装饰器</h3>

当装饰器自身需要参数时，需要额外增加一层（或多层）函数来接收这些参数，每一层返回下一层函数，最终最内层接收被装饰对象（函数或类），并返回增强后的对象。

```
@A(1)(2)
def f():
    pass

# 等价于
f = A(1)(2)(f)
```

## 内置装饰器

<h3><code>@dataclass</code></h3>

用于装饰类，自动生成类的`__init__`,`__repr__`,`__eq__`等方法。

```python
@dataclass
class User:
    name: str
    age: int
```

# 常用标准库

## `typing`

### `TypedDict`

`TypedDict` 是` Python 3.8` 引入的一个类型标注工具，用来声明一个**字典的结构**。通过将该结构标注给字典变量，让静态类型检查工具进行安全检查。

<h4>使用示例</h4>

```python
# 通过类字段和类型注解声明字典结构
class User(TypedDict):
    name: str
    age: int

# 创建时仍然是字典的创建方式
user: User = {
    "name": "张三",
    "age": 20,
}
```

- `TypedDict` **只用于静态类型检查，不会在运行时验证数据，且运行时依然只是普通 `dict`**。

<h4>特殊用法</h4>

**可选字段**

默认情况下，所有字段都是必需的。可以使用 `NotRequired` 声明可选字段：

```
class User(TypedDict):
    name: str
    age: int
    email: NotRequired[str]
```

也可以通过 `total=False` 让**所有字段都变成可选字段**：

```
class User(TypedDict, total=False):
    name: str
    age: int
```

### `Annotated`

`Annotated`是一个类型标注工具，可以为**类型附加额外的元数据**，元数据能够被框架或工具读取并使用。

<h3>使用示例</h3>

```python
# 基本结构为Annotated[实际类型, 元数据1, 元数据2, ...]
age: Annotated[int, "年龄必须大于 0"]
```



# 泛型

泛型将类型当作参数传递，用于编写与具体类型无关、可复用的代码，并可以配合类型检查器提供更好的类型提示。

<h3>基本语法</h3>

使用泛型时，需要通过 `TypeVar` 定义类型变量：

```python
from typing import TypeVar
T = TypeVar("T")
```

- 构造器参数为该类型变量对象内部记录的名字，通常遵循变量名和类型变量的名字保持一致。

之后就可以在方法/类中使用该类型变量

```python
#泛型函数
def max_value(a: T, b: T) -> T:
    return a if a > b else b

#泛型类
class Box(Generic[T]):
	pass

#后续使用该类时，通过[]指定具体类型
box1 = Box[int]()
```



# 常用`Python`包

## `python-dotenv`

是一个 **用来在 Python 中管理环境变量的工具**，可以把环境变量写在文件里，然后在程序启动时自动加载到环境里。它解决了直接在系统环境里设置变量的不方便问题。

<h3>原理</h3>

在`.env`文件中以`key=value`的形式配置环境变量，可以使用 `python-dotenv`读取 `.env` 文件，把里面的键值对加载到系统环境变量中，后续可以通过`os`相关API获取并使用。

- `.env`文件支持`#`单行注释

<h3>使用示例</h3>

```python
from dotenv import load_dotenv
import os

# 指定 .env 文件路径（默认会找当前目录下 .env）
load_dotenv()

# 现在就可以像读取普通环境变量一样
debug = os.getenv("DEBUG")
db_url = os.getenv("DATABASE_URL")

print("DEBUG:", debug)
print("DATABASE_URL:", db_url)
```

## `FastAPI`

一个用于构建 **Python Web 服务** 的现代 Web 框架。

## `Streamlit`

`Streamlit` 是一个用于 **快速开发数据应用和 AI 应用 Web 界面**的开源框架。

## `Pydantic`

`Pydantic` 是 `Python` 中用于**数据校验、类型转换和数据建模**的库。

# `UV`

`uv`是 Astral 使用 Rust 编写的 Python 工具链，它将以下功能整合到了一个工具中：

- Python 版本管理：类似 `pyenv`
- 虚拟环境：替代 `python -m venv`
- 依赖安装：替代 `pip`
- 项目依赖管理：类似 Poetry
- 依赖锁定：生成 `uv.lock`
- CLI 工具运行：类似 `pipx`

## 安装

Windows 可以使用 Winget 安装：

```
winget install --id=astral-sh.uv -e
```

也可以使用官方安装脚本：

```
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

对于 `Linux/Macos`，可以使用官方脚本安装：

```
curl -LsSf https://astral.sh/uv/install.sh | sh
```

安装后，通过以下命令检查是否安装成功：

```
uv --version
```

如果通过官方脚本安装，可以通过以下命令更新

```
uv self update
```

## 常用命令

### `Python`版本管理

相当于`pyenv`，用于`Python`的安装和版本管理

```
uv python <sub_command>
```

<h4><code>uv python list</code></h4>

查看可用的 `Python` 版本

<h4><code>uv python install/uninstall</code></h4>

安装/删除指定 `python` 版本

```
uv python install 3.12
```

常用参数有：

- `--install-dir <path>`|`-i`：指定安装目录，也可以通过设置`UV_PYTHON_INSTALL_DIR`环境变量更改安装目录。

<h4><code>uv python dir</code></h4>

查看当前 `Python` 安装路径，也就是`UV_PYTHON_INSTALL_DIR`环境变量的值。

```
uv python dir
```

常用参数有：

- `--bin`：查看`python.exe`的存放路径，也就是`UV_PYTHON_BIN_DIR`的值

### 虚拟环境

```
uv venv
```

# `pip`

## 常用命令

<h3><code>pip install</code></h3>

安装依赖

```
pip install <package_name>
```

常用参数有：

- `-U`|`--upgrage`：将包升级到符合要求的最新版本，如果未安装，则直接安装。