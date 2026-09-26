# 一、为什么需要 SSH

腾讯云服务器位于腾讯云的数据中心，并不在本地电脑旁边。平时无法直接给云服务器连接显示器、键盘和鼠标，因此需要通过网络远程管理。

SSH 可以让本地 Windows 电脑安全地连接远程 Ubuntu 服务器：
```
Windows 本地电脑
    ↓
SSH 客户端
    ↓
  互联网
    ↓
腾讯云防火墙
    ↓
Ubuntu 服务器的 SSH 服务
    ↓
远程 Shell
```

连接成功以后，虽然命令是在本地键盘上输入的，但命令实际上是由远程 Ubuntu 服务器执行。

# 二、SSH 是什么

SSH 全称是 Secure Shell，是一种用于安全远程连接的网络协议。

SSH 可以用于：
- 远程登录 Linux 服务器
- 在远程服务器上执行命令
- 在本地电脑和服务器之间传输文件
- 使用公钥和私钥进行身份认证
- 建立 SSH 隧道
- 为 Git 等工具提供安全连接

SSH 默认使用：
```
TCP 22
```

这里的 22 是默认端口号，并不是固定不能修改。服务器管理员也可以把 SSH 配置到其他窗口。

# 三、SSH 客户端和 SSH 服务器

一次 SSH 连接==至少==涉及两个角色，一个 SSH 客户端、一个 SSH 服务端

## 1、SSH 客户端

**SSH 客户端运行在本地 Windows 电脑上，负责主动发起连接**。

Windows 系统通常自带 OpenSSH 客户端，命令时：
```
ssh
```

## 2、SSH 服务端

**SSH 服务端运行在 Ubuntu 云服务器上，负责监听连接请求**。

OpenSSH 服务端程序通常叫：
```
sshd
```

其中结尾的 `d` 来自：
```
daemon
```

表示在后台长期运行的服务程序。

因此可以简单理解为：
```
ssh
└── 本地使用的 SSH 客户端命令

sshd
└── 服务器上运行的 SSH 服务程序
```

# 四、Windows Terminal、PowerShell 与 SSH 的关系

## 1、Windows Terminal

Windows Terminal 是一个终端窗口程序，主要负责：
- 显示文字
- 接收键盘输入
- 管理多个终端标签页
- 承载 PowerShell、CMD 等命令行环境

## 2、PowerShell

PowerShell 是 Windows 上的命令行 Shell。

打开 PowerShell 后，通常会看到类似：
```
PS C:\User\用户名>
```

其中：
- `PS` 表示当前使用 PowerShell
- `C:\User\用户名` 表示当前所在的 Windows 目录
- `>` 表示正在等待输入命令

## 3、ssh.exe

ssh.exe 是实际负责建立 SSH 连接的客户端程序。

三者的关系可以理解为：
```
Windows Terminal
└── PowerShell
    └── 执行 ssh.exe
        └── 连接 Ubuntu 服务器
```

因此，Windows Terminal 不是 SSH，PowerShell 也不是 SSH。它们只是用来运行 ssh 命令的环境。

# 五、为什么优先使用 PowerShell 学习

目前可以使用的服务器连接工具有很多，例如：
- Windows Terminal
- PowerShell
- XShell
- FinalShell
- MobaXterm
- 腾讯云网页终端

从零学习 Linux 时，建议优先使用：
```
Windows Terminal + PowerShell + Windows OpenSSH
```

原因是：
- Windows 通常已经自带，不需要额外安装
- 使用的是标准 `ssh` 命令
- 不会把关键过程隐藏在图形界面后面
- 更容易理解用户名、IP、端口和密钥
- 后续学习 Git、SCP、SSH 密钥时可以沿用
- 使用方式与开发工作中的命令行环境更加接近

腾讯云网页终端可以作为应急工具，当 SSH 配置错误或服务器网络异常时，可以通过腾讯云控制台进入服务器修复。

# 六、检查本地是否已经安装了 SSH 客户端

在 Windows 上打开 PowerShell，执行：
```
ssh -V
```

注意这里是==大写字母==：
```
-V
```

不是小写字母 `v`

正常情况下会显示类似：
```
OpenSSH_for_Windows_9.xp1,LibreSSL...
```

具体版本号可能不同，只要能够显示 OpenSSH 版本信息，就说明 SSH 客户端已经可以使用。

还可以执行：
```
Get-Command ssh
```

它会显示 `ssh.exe` 的安装位置

常见位置类似：
```
C:\Windows\System32\OpenSSH\ssh.exe
```

>`ssh -V` 用于查看版本
>`ssh -v` 则用于输出详细连接调试信息
> Linux  和命令行参数通常要==区分大小写==，因此两者含义不同

# 七、建立 SSH 连接需要哪些信息

连接服务器通常需要四类信息：

|   信息   |   当前服务器对应内容   |
| :----: | :-----------: |
| 服务器地址  | 腾讯云服务器公网 IPv4 |
| SSH 端口 |   默认是 `22`    |
| 登录用户名  |   `ubuntu`    |
|  身份认证  |   当前使用密码认证    |

最基本的 SSH 登录命令格式是：
```
ssh 用户名@服务器地址
```

当前服务器的命令格式为：
```
ssh ubuntu@服务器地址
```

需要将服务器地址更换为腾讯云控制台显示的公网 IPv4

# 八、第一次连接时的主机指纹提示

第一次连接某台服务器时，可能看到类似提示：
```
The authenticity of host 'SERVER_IP' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

这段提示的大致意思是：
```
以前没有连接过这台服务器，
本地电脑还不认识它，
是否信任这台服务器提供的主机密钥？
```

确认公网 IP 输入正确，并且确定连接的是自己的腾讯云服务器后，输入：
```
yes
```

==必须输入完整的 yes==，然后回车

之后可能看到：
```
Warning: Permanently added 'SERVER_IP' (ED25519) to the list of known hosts.
```

这表示服务器的主机密钥已经记录到本地

# 九、什么是 SSH 主机密钥

SSH 主机密钥用于证明：
```
当前连接的服务器，是不是之前连接过的那台服务器
```

可以把它理解为服务器的身份指纹

第一次连接后，Windows 会把服务器的公钥信息记录在：
```
C:\Users\当前Windows用户名\.ssh\known_hosts
```

下次连接时，SSH 会比较：
```
服务器当前提供的主机密钥
            与
本地 known_hosts 中保存的主机密钥
```

如果一致，连接可以继续；如果不一致，SSH 就会发出安全警告

> 主机密钥与用户密钥不是同一个概念。主机密钥用于识别服务器，用户密钥用于证明登录用户的身份。

# 十、系统重装后可能出现的警告

服务器以前安装和使用过其他环境，现在重新安装 Ubuntu。重新安装操作系统后，服务器的 SSH 主机密钥通常也会重新生成。

因此，假如 Windows 电脑以前连接过同一个公网 IP，再次连接时可能看到：
```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

或者：
```
Host key verification failed.
```

完整的提示可能类似：
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

这不一定代表服务器被攻击。

在当前场景下，很可能是因为：
```
服务器公网 IP 没有变化
        +
Ubuntu 系统被重新安装
        +
服务器生成了新的 SSH 主机密钥
        +
Windows 中仍然保存着旧主机密钥
```

确认服务器确实是自己刚刚重装的实例后，在本地 PowerShell 执行：
```
ssh-keygen -R SERVER_IP
```

例如：
```
ssh-keygen -R 115.xxx.xxx.xxx
```

这条命令会删除本地 `known_hosts` 中与该 IP 对应的旧记录。

然后重新连接：
```
ssh ubuntu@服务器地址
```

再次确认新主机指纹并输入:
```
yes
```

> 不能在任何情况下看到主机密钥变化就直接删除记录。如果服务器没有重装、IP 没有调整，却突然出现该警告，应先检查是否连接错了服务器 ，或者是否存在中间人攻击风险

# 十一、输入服务器密码

接收主机密钥后，会看到类似：
```
ubuntu@SERVER_IP's password:
```

此时输入重装系统时设置的密码。

输入密码时，终端不会显示：
- 字母
- 数字
- 圆点
- 星号
- 光标移动

这是正常的安全设计，不代表键盘没有输入。

**正确操作是**：
1. 正常输入密码
2. ==不用在意屏幕有没有变化
3. 输入完成后按回车

密码正确后即可进入 Ubuntu 服务器。

密码错误时，可能看到：
```
Permission denied, please try again.
```

然后系统会要求重新输入。

# 十二、认识 Linux 命令提示符

登录成功后，可能看到类似：
```
ubuntu@Muggle-Ubuntu:~$
```

实际主机名可能与示例不同。

这段提示符可以拆分为：
```
ubuntu @ Muggle-Ubuntu : ~ $
  │            │         │ │
当前用户     主机名    当前目录 普通用户
```

## 1、ubuntu

表示当前登录用户
```
ubuntu
```

## 2、@

用于分隔用户名和主机名。

## 3、主机名

表示当前正在操作哪台计算机。

例如：
```
Muggle-Ubuntu
```

但腾讯云控制台中的实例名称和 Ubuntu 系统内部的主机名不一定完全相同

## 4、~

`~` 表示当前用户的家目录。

对 Ubuntu 用户来说，通常代表：
```
/home/ubuntu
```

## 5、$

`$` 通常表示当前是普通用户。

如果提示符末尾是：
```
#
```

通常表示当前是 `root` 用户。

可以简单记忆为：
```
$
└── 普通用户

#
└── root 用户
```

但不要仅凭提示符判断安全性，实际用户应通过 `whoami` 命令确认。

# 十三、确认当前已经进入服务器

登录成功后，依次执行：
```bash
whoami

```

```bash
hostname
```

```bash
pwd
```

```bash
echo $SHELL
```

```bash
uname -m
```

## 1、whoami

作用：`
```
查看当前正在使用的用户
```

当前通常应该输出：
```
ubuntu
```

## 2、hostname

作用：
```
查看当前服务器的主机名
```

输出内容取决于腾讯云的镜像设置。

## 3、pwd

`pwd` 可以理解为：
```
print working directory
```

作用：
```
查看当前所在目录
```

刚登录时通常输出：
```
/home/ubuntu
```

## 4、echo $SHELL

作用：
```
查看当前用户默认使用的 SHELL
```

通常会输出：
```
/bin/bash
```

表示默认 SHELL 是 Bash

这里的：
```bash
$SHELL
```

是一个环境变量，后续会专门学习。

## 5、uname -m

作用：
```
查看当前系统的硬件架构
```

常见输出是：
```
X86_64
```

表示 64 位 X86 架构。

# 十四、如何判断命令是在本地还是服务器执行

连接前，PowerShell 提示符可能类似：
```
PS C:\Users\用户名>
```

这时命令在本地 Windows 电脑上执行。

连接后，提示符可能类似：
```
ubuntu@服务器主机名：~S
```

这时命令在远程 Ubuntu 服务器上执行。

可以通过提示符快速区分：

|        提示符示例        |        命令执行位置         |
| :-----------------: | :-------------------: |
| `PS C:\Users\用户名>`  |      本地 Windows       |
| `ubuntu@hostnam:-$` |       远程 Ubuntu       |
| `root@hostname:-#`  | 远程 Ubuntu 的 `root` 用户 |

这一点非常重要。

例如，在 SSH 会话中执行：
```bash
rm 文件名
```

也可以执行：
```bash
logout
```

或者按：
```bash
Ctrl + D
```

退出成功后，终端会返回本地 PowerShell 提示符：
```
PS C:\Users\用户名>
```

`Ctrl + C` 和 `Ctrl + D` 的区别：
```
Ctrl + C
└── 中断当前正在运行的命令

Ctrl + D
└── 发送文件结束信号，在空命令行下通常会退出当前 Shell
```

因此， `Ctrl + C` 通常不是退出 SSH 的标准方式。

# 十六、SSH 命令的基本格式

SSH 命令的一般格式是：
```
ssh [选项][用户名@]服务器地址
```

当前最常用高德形式是：
```bash
ssh ubuntu@SERVER_IP
```

如果 SSH 服务使用的不是默认 `22` 端口，可以使用：
```bash
ssh -p 端口号 ubuntu@SERVER-IP
```

例如：
```bash
ssh -p 2222 ubuntu@SERVER_IP
```

注意：==SSH  指定端口使用的是小写==：
```bash
-p
```

后面使用密钥登录时，还会接触：
```bash
ssh -i 私钥文件 ubuntu@SERVER_IP
```

# 十七、腾讯云网页终端的作用

腾讯云控制台通常提供：
- OrcaTerm 登录
- 密码登录
- VNC 登录
- 执行命令
- 重置密码

这些方式可以作为应急入口。

例如，出现下面情况时，可以使用腾讯云网页终端：
- SSH 连接超时
- SSH 配置文件修改错误
- 系统防火墙误关闭了 `22` 端口
- SSH 服务没有启动
- 本地网络无法连接服务器
- 忘记服务器密码
- 需要查看服务器启动状态

# 十八、常见连接错误

## 1、Connection timed out

示例：
```bash
ssh: connect to host SERVER_IP port 22: Connection timed out
```

通常表示客户端长时间没有收到服务器响应。

可能原因：
- 公网 IP 输入错误
- 腾讯云防火墙没有开放 TCP `22`
- 服务器已经关机
- 本地网络无法访问服务器
- 系统内部防火墙拦截了 SSH
- SSH 服务没有正常运行

在 ==PowerShell 中==可以检查 `22` 端口连通性：
```bash
Test-NetConnection SERVER_IP -Port 22
```

重点观察：
```bash
TcpTestSucceeded
```

如果显示：
```bash
True
```

表示本地到服务器 `22` 端口基本可达。

如果显示：
```bash
False
```

表示端口不可达，需要检查云防火墙、服务器状态和网络。

## 2、Connection refused

示例：
```bash
ssh: connect to host SERVER_IP port 22: Connection refused
```

这通常表示已经到达服务器，但目标端口没有正常提供 SSH 服务。

可能原因：
- SSH 服务没有启动
- SSH 服务监听了其他端口
- SSH 配置错误
- 系统防火墙主动拒绝连接

可以通过腾讯云网页终端进入服务器，执行：
```bash
sudo systemctl status ssh --no-pager
```

查看 SSH 服务状态。

## 3、Permission denied

示例：
```bash
Permission denied, please try again.
```

或者：
```bash
Permission denied (publickey,password).
```

可能原因：
- 用户名错误
- 密码错误
- 键盘大小写状态错误
- 输入法影响了特殊字符
- 服务器禁止密码登录
- 登录用户没有对应认证权限

当前服务器默认登录用户名是：
```bash
ubuntu
```

不得误写成：
```bash
root
```

或者：
```bash
Ubuntu
```

Linux 用户名区分大小写。

## 4、Could not resolve hostname

示例：
```bash
ssh: Could not resolve hostname ...
```

通常表示服务器地址填写错误。

使用公网 IP 登录时，应检查：
- 是否输入了多余字符
- 是否带有中文标点
- 是否误输入了空格
- 是否把用户名和 IP 顺序写反

正确格式：
```bash
ssh ubuntu@SERVER_IP
```

## 5、Host key verification failed

通常是本地保存的主机密钥与服务器当前密钥不一致。

如果刚刚确认重新安装过系统，可以执行：
```bash
ssh-keygen -R SERVER_IP
```

然后重新连接。

## 6、PowerShell 找不到 ssh

如果出现：
```bash
ssh : 无法将“ssh”项识别为 cmdlet、函数、脚本文件或可运行程序
```

说明 Windows OpenSSH 客户端可能没有安装，或者系统环境存在问题。

可以在 Windows 中打开：
```
设置
→ 系统
→ 可选功能
→ 查看功能
→ OpenSSH 客户端
```

安装完成后重新打开 PowerShell。












