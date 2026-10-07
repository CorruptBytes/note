# 概述

> Nginx是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。

- 特点
  1. 占用内存少，并发能力强

## 反向代理

> 作用于服务器端的，将发送来的请求分发给不同的服务器；相反，正向代理就是作用于客户端的，将客户端发出的请求代理给别的客户端发送

## 负载均衡

# Nginx 多域名端口转发原理

可以将多个域名挂载在同一个Nginx服务上，Nginx 会读取 HTTP 请求头中的 Host 字段（即用户访问的域名），然后根据配置，将请求转发给运行在服务器不同端口上的对应服务。

# 配置文件
- Nginx配置文件格式的核心特点为指令驱动、层级嵌套、以分号结尾。
## 基本语法

**指令**
指令就是配置项，分为两种：简单指令和块指令，简单指令由指令名和参数组成，两者必须用空格分隔，必须以英文分号 ; 结尾；
块指令包含名称、参数（可选）和一对花括号，不需要分号结尾。块指令的花括号内部可以继续嵌套其他指令。
 **注释**
 只支持单行注释，使用 # 号开头。
```
# 这是一个正确的配置示例
worker_processes  1;        # 指令 + 参数 + 分号
error_log  logs/error.log;  # 参数是一个文件路径
```
## 上下文
Nginx 通过块指令的嵌套形成树形配置结构，通过块指令嵌套，Nginx创建了不同层级的**配置上下文**，它约束了某个配置指令允许出现的位置，以及该指令的作用范围。Nginx 核心和各功能模块会定义各自支持的上下文，用于组织、解析和管理配置：
- 主上下文（Main）：最外层，全局生效，是唯一未被花括号包围的上下文。
- 事件上下文（Events）：必须在主上下文中，配置网络连接机制。
- HTTP 上下文（Http）：必须在主上下文中，配置 Web 服务核心。
- 服务器上下文（Server）：必须在 http 块内，代表一个虚拟主机（站点）。
- 位置上下文（Location）：必须在 server 块内，匹配具体的 URL 路由。
- 上游上下文（Upstream）：必须在 http 块内，定义后端服务器集群。
```
worker_processes  1;                    # <-- 主上下文 (Main)

events {                                # <-- 事件上下文
    worker_connections  1024;
}

http {                                  # <-- HTTP 上下文
    include       mime.types;
    default_type  application/octet-stream;

    upstream backend {                  # <-- 上游上下文 (在 http 内)
        server 127.0.0.1:3000;
    }

    server {                            # <-- 服务器上下文 (在 http 内)
        listen       80;
        server_name  localhost;

        location / {                    # <-- 位置上下文 (在 server 内)
            root   html;
            index  index.html;
        }

        location /api/ {
            proxy_pass http://backend;  # 引用 upstream
        }
    }
}
```
## 继承与覆盖
- 子上下文会自动继承父上下文中定义的指令。
- 如果子上下文中重新定义了相同的指令，则覆盖父级的设置，否则沿用父级。
- 并非所有指令都允许在子级重写，取决于指令的作用域
## 变量与字符串
变量以 $ 符号开头，用于获取请求信息或进行逻辑判断。字符串可以不加引号，也可以加单引号或双引号（包含空格或特殊字符时建议加）

## Include
为了方便维护，主配置文件通常非常精简，大部分配置放在外部的 .conf 文件，并通过 include 指令引入外部的 .conf 文件以使其生效。

# `stream`模块

`stream`模块是 Nginx 提供的 **四层代理（L4 Proxy）模块**，用于转发 **TCP/UDP 流量**。它工作在四层(传输层)，只理解 TCP/UDP 连接。

## 配置

`stream`模块的配置指令均放在`stream`上下文中管理：

```nginx
stream {

    upstream mysql_backend {
        server 10.0.0.1:3306;
        server 10.0.0.2:3306;
    }


    server {

        listen 3306;

        proxy_pass mysql_backend;
    }
}
```

