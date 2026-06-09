# 魔术方法

## `__call__()`

> 实现call方法后，对象可以当作函数一样调用。当调用对象时，会调用对象的call方法

## 运算符

### 三元运算符

```python
#如果boolen为True,则表达式值为x，否则为y
x if boolen else y
```

# 常用`Python`包

## `python-dotenv`

是一个 **用来在 Python 中管理环境变量的工具**，可以把环境变量写在文件里，然后在程序启动时自动加载到环境里。它解决了直接在系统环境里设置变量的不方便问题。

<h3>原理</h3>

在`.env`文件中以`key=value`的形式配置环境变量，可以使用 `python-dotenv`读取 `.env` 文件，把里面的键值对加载到系统环境变量中，后续可以通过`os`相关API获取并使用。

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

