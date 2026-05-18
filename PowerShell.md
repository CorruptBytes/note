# 概述

> 微软于2006年开发的命令解释器，为了解决传统Windows命令行(`cmd`，命令提示符)的局限性。最初名为`Monad`,构建于`.NET Framework`之上。

**`PowerShell`与`cmd`对比**

| 对比点       |    cmd     | PowerShell |
| ------------ | :--------: | :--------: |
| 出现时间     |    更早    |    更晚    |
| 设计思想     | 字符串命令 |  面向对象  |
| 管道         |  传字符串  |   传对象   |
| 脚本能力     |     弱     |    很强    |
| 现代系统管理 |     ❌      |     ✅      |

## 可执行命令

PowerShell中的可执行命令分为四类：

- **外部命令：**包括系统上任何可用命令，它们是PowerShell外部独立的可执行文件。

| 类型       | 示例               | 本质                |
| ---------- | ------------------ | ------------------- |
| **Cmdlet** | `Get-Process`      | **.NET 类实例方法** |
| 函数       | `cd`（实际是函数） | PowerShell 代码     |
| 别名       | `ls`               | Cmdlet 的快捷名     |
| 外部程序   | `ping` / `git`     | `.exe` / ELF        |

### `cmdlet`

**PowerShell命令：**称为`cmdlet`,它不是独立的可执行文件，而是位于`PowerShell`模块的内部命令，可按需加载。

- 可以用任何编译的 .NET 语言或 PowerShell 脚本语言本身来编写 cmdlet
- PowerShell中以**“动词-名词”**的格式命名`cmdlet`，动词为执行的操作，名词为被操作对象。如`Get-Command`获取已注册的所有 cmdlet

`PowerShell`官方**限定了`cmdlet`命名可用的动词集合**，避免乱命名

| 动词     | 含义        | 示例                |
| -------- | ----------- | ------------------- |
| `Get`    | 获取        | `Get-Process`       |
| `Set`    | 设置        | `Set-Location`      |
| `New`    | 创建        | `New-Item`          |
| `Remove` | 删除        | `Remove-Item`       |
| `Invoke` | 执行 / 调用 | `Invoke-WebRequest` |

- 通过`Get-Verb`查看**所有合法动词**

## 安装PowerShell

通常Windows会自带旧版本(5.1版本及以前)的PowerShell(`Windows PowerShell`)。可以手动安装新一代的PowerShell(`PowerShell Core`)

```bash
#搜索最新版本的 PowerShell
winget search --id Microsoft.PowerShell

#选择所需的PowerShell版本，填入其Id会自动下载其 MSI 包
winget install --id Microsoft.PowerShell --source winget
```

## 配置文件

配置文件是特殊的脚本，在`PowerShell`启动时自动执行，用于根据需求定制环境。 

配置文件可以针对当前用户和特定主机、当前用户的所有主机、所有用户针对特定主机，或所有用户在所有主机等几种不同的范围生效，以决定哪些自定义配置应广泛应用，或仅适用于特定上下文。

配置文件的配置内容包括：

- 为常用命令设置别名
- 定义自定义函数以自动化重复任务
- 在启动时自动导入所需模块。
- 命令行提示符外观
- 环境变量

# 命令

**查看Power Shell信息**

```
get-host
```

# 模块

# 小知识

## Shell

命令解释器。

**功能**

- 解析用户输入的文本命令，调用内核或其他程序执行，最后返回执行结果

Shell可执行的命令可分为两类：

- **外部命令：**是一个独立于Shell的程序，本质是磁盘上的**可执行文件**,由 Shell fork + exec 调用。
  - Shell 会在 `PATH` 中查找可执行文件
  - 找到后通过 `fork()` 创建子进程，再 `exec()` 加载程序
- **Shell内建命令：**不是独立程序，由 Shell 自己实现的命令。
  - 直接在 Shell 进程内执行，不需要 fork 新进程。

## `CLI/GUI`

命令行接口(Command Line Interface),是一种**文本形式的人机交互**方式，用户以文本输入命令，系统以文本输出结果。

- 命令行环境**通常**由命令行接口（CLI）与命令解释器（Shell）共同组成。在现代操作系统中，它们通常是两个独立的程序，CLI接收文本命令并通过文本输出命令结果，Shell 解析并执行输入的命令，并将执行结果返回给 CLI 

与命令行相对的是图形化用户接口(GUI，Graphical User Interface),通过**窗口、按钮、菜单、鼠标**与系统交互

## 命令行提示符

由Shell输出到CLI显示的一段文本，用于提示用户当前可以输入命令，并反映当前的执行上下文信息

```bash
#cmd.exe的命令行提示符
C:\Users\Alice>
#bash的命令行提示符
alice@host:~/code$
```

## `PowerShell Core`

**PowerShell Core** 指的是从 PowerShell 6 开始的跨平台 PowerShell 系列，它们基于 **.NET Core/.NET** 构建，**开源且跨平台** ，支持 Windows、Linux 和 macOS。

通常把Windows自带的老版本PowerShell称为 **Windows PowerShell**,它包括 **PowerShell 5.1 及之前的版本**，这一系列仅支Windows。

从PowerShell 6起，微软推出新一代Power shell，称为**PowerShell Core**，基于 `.NET Core/.NET` 构建 ，实现了跨平台支持,它是PowerShell现在的主线发展版本。

- `Windows PowerShell`的可执行文件为`powershell.exe`,`PowerShell Core`的可执行文件为`pwsh.exe`。
- 自` PowerShell 7` 发布并确立为主线版本后，微软直接称`PowerShell 6`及其之后的版本为`PowerShell`,将`PowerShell 5.1`及之前的版本称为`Windows PowerShell`。

## 终端（Terminal）

终端是 CLI 的一种常见实现载体，是一个提供文本输入与输出的独立程序。通过键盘输入文本与屏幕回显文本，让用户与计算机中的程序（通常是 Shell）进行交互。

- 常说的命令行通常指终端 + Shell 的组合

Windows中默认提供了两种终端：

- **`Windows conhost`:**`Windows`控制台主机，旧的终端
- **`Windows Terminal`:**`Windows`终端，是微软新推出的现代终端模拟器,默认使用。

