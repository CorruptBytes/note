# 概述

# 命令

Linux 发行版中的命令来自不同开源项目提供的用户空间工具，因此参数风格、行为习惯存在差异。

## 文件相关

<h3><code>mv</code></h3>

移动或重命名文件/目录。

```bash
#重命名文件/目录
mv [OPTION]... SOURCE DEST

#移动文件/目录到指定目录
mv [OPTION]... SOURCE... DIRECTORY
mv [OPTION]... -t DIRECTORY SOURCE...
```

<h3><code>mkdir</code></h3>

创建目录

```bash
mkdir [OPTION]... DIRECTORY...
```

<h4>常用选项</h4>

- `-p`/`--parents`：如果需要，可以递归创建父目录

<h3><code>cd</code></h3>

切换当前工作目录

```bash
cd [-L|[-P [-e]] [-@]] [dir]
```

- 不指定`dir`时，默认切换到`HOME`环境变量指定的目录，
- `dir`参数可以使用`~`表示`HOME`环境变量指定的目录，`.`表示当前目录，`..`表示上一级目录。

<h3><code>ls</code></h3>

列出目录内容

```bash
ls [OPTION]... [FILE]...
```

- 不指定`FILE`时，默认列出当前目录内容。

<h3><code>grep</code></h3>

通过正则表达式按行过滤标准输入或文件内容，并输出匹配的行

```
grep [OPTION]... PATTERNS [FILE]...
```

- 除了对文件进行过滤，`grep`还可以通过管道符`|`接收其他命令输出并进行过滤
- 管道符 `|` 用来 **把前一个命令的标准输出（`stdout`）传给下一个命令的标准输入（`stdin`）**。

<h3><code>tar</code></h3>

`tar` 是 Linux/Unix 中常用的归档工具，用于将多个文件和目录归档（archive）或解包（extract），并可结合 gzip、bzip2 等压缩工具对归档文件进行压缩或解压。

```
tar [OPTION...] [FILE]...
```


<h3><code>sed</code></h3>

一个按行处理文本流的编辑工具，主要用于 **文本处理和批量修改**。可以对文件或标准输入进行模式匹配、替换、插入、删除等操作

```
sed [OPTION]... {script-only-if-no-other-script} [input-file]...
```

- 如果不指定`input-file`，则从标准输入读取。
- **按行处理文本**，不会一次性加载整个文件，适合大文件

<h4>脚本格式</h4>

`sed`脚本的基本格式为`[address1[,address2]]<command>`表示对匹配 `address` 的行执行 `command` 编辑命令。`address`可选，如果不指定，则对 **所有行** 执行该命令。

`address`可以通过多种方式指定：

常用`command`有：

```
s/old/new/flags
```

<h4>常用选项</h4>

- **`-i`：**直接修改文件

<h3><code>sort</code></h3>

用于对文本文件或标准输入的行进行排序

```
sort [OPTION]... [FILE]...
```

- 如果不指定文件，则从标准输入读取。
- 不指定选项时，默认按字典序(ASCII)排序，逐行排序，输出到标准输出，不修改原文件

<h4>常用选项</h4>

- **`-r`：**倒序排序
- **`-n`：**按数值大小排序，而不是字符串比较。
- **`-u`：**对排序结果去重

<h3>vim/vi</h3>

`vi`（Visual Interface）是最早的 Unix 文本编辑器之一。`vim`（Vi IMproved）是 `vi` 的增强版，功能更强大（支持语法高亮、插件、可视化模式等)

```
vim [options] [file ..]
```

- 如果文件不存在，会自动创建。

<h4>模式</h4>

使用`vim`命令后，会进入一个交互式的编辑界面，在此界面下，`vim`有三种工作模式：

- **命令模式(`Command Mode`)：**使用`vim`命令后默认进入的模式。用于命令操作，在此模式下，输入内容会作为命令被处理而不是编辑内容。
- **输入模式(`Insert Mode`)：**进行文本编辑。
- **命令行模式(`Command-Line Mode`)：**输入命令以进行保存、退出、查找等操作。

其中，命令模式为默认模式，可以在命令模式中输入`i/a/o/I/A`切换到输入模式,输入`:`切换到命令行模式。也可以通过`esc`从输入模式或命令行模式切换到命令模式。

- `i/a/o/I/A/O`不同命令插入的起始位置不同。

| 命令 |        作用        |
| :--: | :----------------: |
|  i   |    在光标前插入    |
|  a   |    在光标后插入    |
|  o   | 在光标下方新起一行 |
|  I   |      行首插入      |
|  A   |      行尾插入      |
|  O   | 在光标上方新起一行 |

<h4>命令</h4>

**命令模式常用命令**

| 命令 | 作用 |
| :--: | :--: |
|  h   |  左  |
|  j   |  下  |
|  k   |  上  |
|  l   |  右  |

**命令行模式常用命令**

|   命令   | 作用         |
| :------: | ------------ |
|    w     | 保存         |
|    q     | 退出         |
|    wq    | 保存并退出   |
|    q!    | 强制退出     |
|   wq!    | 强制保存退出 |
|  set nu  | 显示行号     |
| set nonu | 取消行号     |

## 进程相关

<h3><code>ps</code></h3>

查看当前系统中的进程信息

```bash
ps [options]
```

<h4><code>ps</code>可展示信息</h4>

- `PID`：进程ID。
- `PPID`：父进程ID。
- `USER`：进程所属用户。
- `%CPU`：CPU占用率。
- `%MEM`：内存占用率。
- `VSZ`：虚拟内存大小。
- `RSS`：物理内存大小。
- `TTY`：终端设备。
- `STAT`：进程状态。
- `START`：进程启动时间。
- `TIME`：进程累计CPU占用时间。
- `CMD` | `COMMAND`：启动进程的命令或可执行文件

<h4>常用选项</h4>

- **`-A` | `-e`：** 显示系统中所有进程
- **`-a`：** 显示当前终端下的所有进程（除去 session leader）
- **`-u USER`：** 显示指定用户的进程
- **`-x`：** 显示没有控制终端的进程
- **`-f`：** 以完整格式显示进程信息（UID、PPID、C、STIME、TTY、TIME、CMD）
- **`-l`：** 以长格式显示进程信息，包含更多状态字段（F、S、UID、PID、PPID、C、PRI、NI、ADDR、SZ、WCHAN、TTY、TIME、CMD）
- **`-o FORMAT`：** 自定义输出状态字段，例如 `ps -o pid,ppid,cmd`
- **`--forest`：** 以树状图显示进程父子关系
- **`-t TTY`：** 显示指定终端的进程
- **`-p PID`：** 显示指定 PID 的进程
- **`-r`：**只显示正在运行（running）状态的进程
- **`-T`：**显示系统中的线程信息

<h3><code>top</code></h3>

实时查看系统资源与进程/线程状态

```
top [options]
```

- 默认`top`命令显示的是系统资源和所有进程状态。

<h4>常用参数</h4>

- **`-H`：**显示 **线程**信息

<h4>显示内容</h4>

`top`命令显示的内容分为两部分，上部分为系统信息，下部分为进程列表。

```bash
top - 18:48:43 up  1:28,  1 user,  load average: 0.00, 0.00, 0.00
Tasks:  35 total,   1 running,  34 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :  15808.1 total,  13695.0 free,   1334.9 used,    778.3 buff/cache
MiB Swap:   4096.0 total,   4096.0 free,      0.0 used.  14265.1 avail Mem
```

- **`up`：**系统运行时间（uptime）
- **`load average`：**系统平均负载，分别为最近 1 分钟平均负载，最近 5 分钟平均负载，最近 15 分钟平均负载。**负载 = 运行队列中的平均进程数 + 等待 CPU 的进程数**
- **`Tasks：`**系统进程概览，分别为总进程数，正在运行的进程数，睡眠（阻塞或等待 CPU/IO）的进程数，被停止的进程数，僵尸进程数量（已退出但父进程未回收的进程）。
- **`%Cpu(s)`：**CPU 时间分配情况。
- **`MiB Mem`：**系统物理内存（单位 MiB = Mebibyte），分别为总内存容量，空闲内存，已使用内存，被内核 buffer/cache 占用的内存
- **`MiB Swap`：**交换分区内存信息，分别为总交换分区容量，空闲交换空间，已使用交换空间
- 最后为可用内存，表示系统还能分配给新进程的内存

下部分进程列表会显示进程的PID、用户、CPU占用率、内存占用率、进程状态、启动时间和进程命令。

```
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
      1 root      20   0  165976  10240   7680 S   0.0   0.1   0:00.45 systemd
      2 root      20   0    3060   1536   1536 S   0.0   0.0   0:00.00 init-systemd(Ub
```

<h4>交互模式</h4>

`top`命令还提供交互模式，通过输入命令，用户在可以在运行时动态调整显示和排序

- **`1`：**显示每个 CPU 核心的使用情况，默认情况下显示的是 **总体 CPU 使用率**（所有核心平均）

<h3><code>kill</code></h3>

向进程发送信号。可用于终止进程。

```bash
kill [options] pid
```

- 默认发送信号为**SIGTERM（15）**，即请求进程优雅退出

## 网络相关

<h3><code>netstat</code></h3>

用于查看系统的网络连接和端口使用等网络信息

```
netstat [options]
```

<h4>常用选项</h4>

- **`-n`：** 以数字形式显示地址和端口号，不进行 DNS 或服务名解析，提高显示速度。
- **`-a`：** 显示所有连接和监听端口，包括正在监听的服务端口和已建立的连接。
- **`-p`：** 显示使用连接的进程 PID 和程序名（需要 root 权限查看别的用户进程）。
- **`-t` / `-u` / `-l`：**`-t`：只显示 TCP 连接，`-u`：只显示 UDP 连接，`-l`：只显示正在监听的端口

<h3><code>ip</code></h3>

管理和查看系统的网络配置

```bash
#命令格式为ip 对象 操作 ，不同对象具有不同的操作，也就是子命令
ip [ OPTIONS ] OBJECT { COMMAND | help }
```

<h3><code>ping</code></h3>

检测网络连通性以及测量延迟

```
ping [options] <destination>
```

<h3><code>telnet</code></h3>

用于远程连接服务器或测试 TCP 端口连通性。

```
telnet [options] [host-name [port]]
```

## 权限相关

<h3><code>chmod</code></h3>

修改文件或目录的权限

```bash
chmod [OPTION]... MODE[,MODE]... FILE...

chmod [OPTION]... OCTAL-MODE FILE...
```

`chmod`支持两种模式指定权限，一种是符号方式()`MODE[,MODE]...`)通过固定格式字符串指定

**数字方式(`OCTAL-MODE`)**

通过三位八进制数指定，第一位表示所有者的权限，第二位表示所属组的权限，第三位表示其他用户的权限。

每一位中,`4`表示`r`读权限,`2`表示`w`写权限,`1`表示`x`执行权限，相加表示具有多个权限，0表示没有权限。

<h3><code>useradd</code></h3>

创建新用户

```bash
#创建用户
useradd [options] USERNAME

#查看/修改默认配置
useradd -D [options]
```

<h3><code>groupadd</code></h3>

创建新的用户组，用户组用来管理一组用户的权限。

```
groupadd [options] GROUP
```

## 系统相关

<h3><code>uptime</code></h3>

显示系统已运行时间、当前用户数以及系统负载情况。也就是`top`命令输出的第一行

```bash
uptime
```

<h3><code>free</code></h3>

查看内存使用情况

```
free [options]
```

- 默认以`KB`为单位显示。

<h4>常用选项</h4>

- **`-m`：**以`MB`为单位显示

<h4>输出内容</h4>

```
              total        used        free      shared  buff/cache   available
Mem:           15G         8G         2G         1G          5G         6G
Swap:          2G          0G         2G
```

- **Mem**：物理内存
  - `total`：总内存
  - `used`：已用内存（包括缓存/缓冲区）
  - `free`：未使用内存
  - `shared`：共享内存（tmpfs 等）
  - `buff/cache`：缓冲区和缓存
  - `available`：可用内存
- **Swap**：交换分区
  - `used`：已用 swap
  - `free`：剩余 swap

<h3><code>dmesg</code></h3>

显示 **内核环形缓冲区**中的消息

## 查看系统与内核信息

- 查看系统版本

  ```
  cat /etc/os-release
  ```

- 查看内核信息

  ```
  uname -a
  ```

  - `-a`：查看内核所有相关信息
  - `-r`:查看内核版本
  - `-s`：查看内核名

# WSL

`Windows Subsystem for Linux`，适用于Windows的Linux子系统，是Windows自带的可选功能，可以在 Windows 里**直接运行Linux环境，而无需安装虚拟机**。

<h3><code>WSL 1</code> 和<code>WSL 2</code></h3>

WSL 1是上一代WSL，WSL 2是现在默认安装的WSL版本。它们的实现原理有本质的不同。

`WSL 1`的本质是一个兼容层，它将`Linux`系统调用翻译为Windows系统调用，并不具有真正的 Linux 内核。这导致`WSL 1`不完全兼容 Linux，且因为缺少 `namespace`、`cgroups` 等内核能力，所以无法运行 Docker 等依赖内核特性的程序。

`WSL 2`基于轻量级虚拟化技术(Hypervisor,简称为Hyper-V)，运行一个真正的 Linux 内核。由于拥有完整的内核能力，因此在兼容性、网络支持和容器运行方面表现更好。

# 小知识

## 多行命令

如果命令一行无法写完或者想要增强可读性，可以在行的末尾添加`\`表示命令未完

```
yum rm docker-ce\
		docker-compose-plugins
```

# 面试题

<h2>如果<code>CPU</code>跑到了100%，解决思路是什么</h2>

1. 首先通过`TOP`命令定位占用`CPU`的进程
2. 通过`ps -T -p <PID>`命令找到进程中占用`CPU`的线程
3. 通过`jstack`查看该线程的堆栈信息
4. 根据堆栈信息，定位项目中的代码，确定`CPU`占用高的原因，如死循环等。

<h2>如何查看系统的负载情况</h2>

通过`top`或`uptime`命令查看当前系统的负载情况。命令输出的`load Average`中的三个数字依次是过去1分钟，过去5分钟，过去15分钟的平均负载。平均负载一般不超过CPU核数的1~1.5倍，如果超过了1.5倍，就会严重影响系统，应该进行排查。

- 如果`load average`中的三个数字依次增大，表明系统的负载是下降的趋势。
- 如果`load average`中的三个数字依次降低，表明系统的负载是上升的趋势。‘
- 如果`load average`中的三个数字基本相同，或者相差不大，表明系统的负载是平稳的。

