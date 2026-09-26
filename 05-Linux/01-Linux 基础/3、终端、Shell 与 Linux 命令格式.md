# 一、登录服务器后发生了什么

在 Windows PowerShell 中执行：
```bash
ssh ubuntu@SERVER_IP
```

连接成功后，屏幕上通常会出现类似内容：
```bash
ubuntu@Muggle-Ubuntu:~$
```

此时可以输入：
```bash
pwd
```

```bash
ls
```

```bash
whoami
```

这些命令并不是由 Windows 执行，而是被发送到远程 Ubuntu 服务器，并由服务器上的 Shell 解释和执行。

整个过程可以表示为：
```
键盘输入命令
    ↓
Windows Terminal 显示输入
    ↓
PowerShell 中的 ssh 客户端传输命令
    ↓
Ubuntu 服务器接收输入
    ↓
Bash 解析命令
    ↓
找到并运行对应程序
    ↓
程序产生输出
    ↓
SSH 把输出传回本地
    ↓
Windows Terminal 显示结果
```

# 二、什么是终端

终端英文是：
```
Terminal
```

终端主要负责：
- 接收键盘输入
- 显示文字内容
- 显示命令执行结果
- 为 Shell 提供一个交互界面

在 Windows 中，常见终端程序包括：
- Windows Terminal
- PowerShell 窗口
- CMD 窗口
- Intellij IDEA Terminal

在现代系统中，人们平时所说的 "终端" ，通常实际指的是：
```
终端模拟器
```

它模拟了传统物理终端的输入和显示能力。

终端本身通常不负责理解 Linux 命令，真正负责解释命令的是 Shell。

# 三、什么是 Shell

Shell 是一个命令解释器，它位于用户与操作系统之间，负责接收并解释用户输入的命令。

```
用户
 ↓
终端
 ↓
Shell
 ↓
Linux 内核与应用程序
```

Shell 会完成以下工作：
1. 读取用户输入
2. 识别命令、选项和参数
3. 处理变量、引号和通配符
4. 查找对应的命令或程序
5. 启动程序
6. 等待程序执行
7. 显示结果并返回新的命令提示符

常见 Shell 包括：

| Shell |          说明           |
| :---: | :-------------------: |
| Bash  | Linux 中最常见的 Shell 之一  |
|  Zsh  |    功能丰富，常用于个人开发环境     |
| Fish  |        更注重交互体验        |
| Dash  |   Ubuntu 中常用于执行系统脚本   |
|  Sh   | 传统 Shell 名称，也代表一类兼容规范 |

Ubuntu 默认用户通常使用 Bash。

# 四、Bash 是什么

Bash 全称为：
```bash
Bourne Again Shell
```

它是 Linux 中广泛使用的一种 Shell。

Bash 可以用于：
- 交互式输入 Linux 命令
- 管理环境变量
- 执行 Shell 脚本
- 处理重定向和管道
- 连接多个命令
- 自动执行重复任务

登录 Ubuntu 后，可以执行：
```bash
echo $SHELL
```

通常会显示：
```bash
/bin/bash
```

这表示当前用户配置的默认登录 Shell 是 Bash。

不过，`echo $SHELL` 更准确地说是在查看：
```
当前用户配置的默认 Shell
```

要查看当前正在运行的 Shell 进程，可以执行：
```bash
ps -p $$ -o comm=
```

通常会输出：
```bash
bash
```

其中：
```bash
$$
```

代表当前 Shell 进程的 PID，也就是进程编号。

# 五、终端、Shell 与命令的关系

可以将三者理解为：
```
终端
└── 提供输入和显示界面

Shell
└── 读取并解释输入内容

命令或程序
└── 完成具体工作
```

例如执行：
```bash
ls -1 /etc
```

过程大致是：
```
终端接收：ls -l /etc
        ↓
Bash 将其拆分为命令、选项和参数
        ↓
Bash 找到 ls 程序
        ↓
ls 查看 /etc 目录
        ↓
ls 按详细格式输出结果
        ↓
    终端显示结果
```

# 六、认识命令提示符

登录后可能看到：
```
ubuntu@Muggle-Ubuntu:-$
```

它不是命令，而是 Shell 显示的提示符。

提示符表示：
```
Shell 已经准备好，可以接收下一条命令
```

可以拆解为：
```
ubuntu @ Muggle-Ubuntu : ~ $
  │             │        │ │
当前用户       主机名    目录 普通用户
```

## 1、当前用户

```
ubuntu
```

表示当前登录用户是 `ubuntu`

可以通过下面的命令确认：
```bash
whoami
```

## 2、主机名

```bash
Muggle-Ubuntu
```

表示当前正在操作的计算机名称。

可以执行：
```bash
hostname
```

查看完整主机名。

腾讯云控制台中的实例名称和 Ubuntu 系统内部的主机名不一定完全相同。

## 3、当前目录

```bash
~
```

波浪号表示当前用户的家目录。

对于 `ubuntu` 用户，一般代表：
```bash
/home/ubuntu
```

可以执行：
```bash
pwd
```

查看当前位置的完整路径。

## 4、`$`与 `#`

提示符末尾的：
```bash
$
```

通常表示当前是普通用户。

而：
```bash
#
```

通常表示当前是 `root` 用户。

```
ubuntu@hostname:~$
                   └── 普通用户

root@hostname:~#
                └── root 用户
```

>在教程或文档中，命令前面经常会显示 `$` 或 `#` ，它们通常只是提示符，不属于命令本身，复制命令时不要一起复制。

例如文档写着：
```bash
$ pwd
```

实际只需要输入：
```bash
pwd
```

# 七、