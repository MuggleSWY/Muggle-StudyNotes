# venv 虚拟环境的创建与使用

## 一、虚拟环境要解决什么问题

### 1. 不同项目需要分别管理依赖

假设项目 A 需要某个第三方库的一个版本，项目 B 需要同一个库的另一个版本。若都直接安装在同一个基础环境中，一个项目的安装或升级操作可能影响另一个项目。

**虚拟环境是建立在已有 Python 安装之上的、为项目提供相对独立软件包集合的运行环境。** 标准库中的 `venv` 可以创建这种环境。

典型关系如下：

```text
基础 Python 安装
├─ 项目 A 的虚拟环境：项目 A 使用的第三方包
└─ 项目 B 的虚拟环境：项目 B 使用的第三方包
```

两个虚拟环境可以使用相同的 Python 版本，但安装不同的软件包。两个环境中的包名和版本恰好相同，也不能据此判断它们就是同一个环境。

### 2. 隔离的边界

`venv` 不是虚拟机，不会给项目创建一套独立 Windows，也不是把整个基础 Python 安装原封不动复制一遍。它会建立环境配置、解释器入口和软件包目录，并继续依赖基础 Python 安装的相关内容。

默认创建的虚拟环境不直接使用基础环境的第三方包安装集合，但仍可使用 Python 标准库。例如，新环境中没有另外安装 `math`，也可以导入它。

这里的隔离主要用于管理软件依赖，不是把程序与文件系统、网络和操作系统完全隔离。不要因为代码运行在 `.venv` 中，就认为不可信代码不会影响其他文件。

环境行为可以通过特定选项和配置改变。因此，“默认隔离”是有前提的，不宜理解为任何配置下都绝对互不影响。

## 二、目录与命令的准备条件

### 1. 本文使用的示例目录

以下操作面向 Windows 和 PowerShell：

| 对象 | 路径 |
|---|---|
| 基础解释器 | `C:\Development\Python\Python314\python.exe` |
| 项目根目录 | `D:\Study\Python\01-environment` |
| 虚拟环境目录 | `D:\Study\Python\01-environment\.venv` |
| 虚拟环境解释器 | `D:\Study\Python\01-environment\.venv\Scripts\python.exe` |

示例中的版本为 Python 3.14.9，用于说明同一基础安装创建出的环境关系，不代表在其他时间安装时必须选这个补丁版本。

操作前先核对实际目录和已安装的解释器，不要为了照抄示例而覆盖已有环境。已经创建好的 `.venv` 不需要重复执行创建命令。

### 2. 提示符决定命令交给谁

创建和激活环境的命令应在 PowerShell 的提示符后输入：

```text
PS D:\Study\Python\01-environment>
```

如果当前显示 `>>>`，表示正在 Python 交互会话中。先用交互退出方式回到 PowerShell，再执行系统命令。提示符本身不用输入。

在 PowerShell 中查看当前目录：

```powershell
Get-Location
```

### 3. 相对路径和绝对路径

下面这条命令明确指定了虚拟环境解释器，但其中的路径仍然是相对路径：

```powershell
.\.venv\Scripts\python.exe --version
```

开头的 `.` 表示当前目录。只有当前目录是 `D:\Study\Python\01-environment` 时，这条路径才指向本文的虚拟环境解释器。

对应的绝对路径是：

```text
D:\Study\Python\01-environment\.venv\Scripts\python.exe
```

**“明确指定解释器路径”不等于“必然使用绝对路径”。** 使用相对路径时，当前目录是命令成立的重要条件；VS Code 中正在编辑哪个文件，不会自动替所有终端命令选择正确的相对路径起点。

## 三、创建环境并核查结构

### 1. 确定创建所用的 Python

在创建环境之前，检查命令找到的 Python：

```powershell
where.exe python
python --version
```

在没有同名 PowerShell 别名或函数覆盖、且 PATH 顺序已经核实的示例环境中，`python` 指向基础解释器。

版本号相同并不能排除多个 Python 安装。应该同时核对位置；需要消除 PATH 选择歧义时，可以使用已经确认的解释器路径。

### 2. 创建命令

在项目根目录中执行：

```powershell
python -m venv .venv
```

| 部分 | 作用 |
|---|---|
| `python` | 启动当前解析到的 Python 解释器 |
| `-m` | 让这个解释器把后面指定的模块作为程序运行 |
| `venv` | 标准库提供的虚拟环境创建模块 |
| `.venv` | 要创建的目标目录，按当前目录解释 |

`.venv` 是常见命名习惯，不是语法规定，也不是通过这个名字就能自动获得隔离能力。环境之所以成立，是因为创建命令建立了相应文件和配置。

创建过程中可能没有持续输出。判断是否完成，应观察命令是否结束、是否出现报错，以及后续验证结果，而不是只看是否显示进度条。

### 3. 常见目录结构

```text
01-environment/
├─ .vscode/
├─ .venv/
│  ├─ Include/
│  ├─ Lib/
│  │  └─ site-packages/
│  ├─ Scripts/
│  │  ├─ python.exe
│  │  └─ Activate.ps1
│  ├─ .gitignore
│  └─ pyvenv.cfg
├─ hello.py
└─ check_requests.py
```

不同版本和创建选项可能使辅助文件略有差异。对基本使用而言，主要关注 `Scripts` 中的解释器与激活脚本、`Lib\site-packages` 中的软件包，以及记录环境信息的 `pyvenv.cfg`。

`hello.py` 等项目代码位于环境目录之外。创建环境不是把自己的代码移动进去。

检查结构和解释器能否启动：

```powershell
Get-ChildItem .venv
.\.venv\Scripts\python.exe --version
```

示例输出版本为：

```text
Python 3.14.9
```

这一步只验证解释器能够启动并报告版本，还没有验证某个第三方库的导入或业务功能。

## 四、激活环境的含义与方法

### 1. 激活主要改变当前 Shell 的命令查找环境

PowerShell 是接收命令的 Shell。执行虚拟环境激活脚本后，它会调整当前会话中的环境变量，通常将 `.venv\Scripts` 放到 PATH 前面，并增加提示标识。

激活后直接输入 `python`，便于优先找到这个环境中的解释器。基础解释器没有被覆盖，虚拟环境文件也不是到激活时才创建。

**创建环境准备文件，激活环境调整当前终端；两者不是同一操作。**

### 2. PowerShell 激活命令

在项目根目录、PowerShell 提示符后执行：

```powershell
.\.venv\Scripts\Activate.ps1
```

成功时，提示符通常变为：

```text
(.venv) PS D:\Study\Python\01-environment>
```

前缀是界面提示，不应作为唯一验证依据。提示符可以被定制，某些工具也会自动处理环境，因此还需要检查实际命令路径。

### 3. 验证查找结果

```powershell
where.exe python
python --version
```

在本文示例中，预期第一个相关路径为：

```text
D:\Study\Python\01-environment\.venv\Scripts\python.exe
```

随后可能继续列出基础解释器，以及 WindowsApps 中的系统安装引导入口。出现多条路径本身不是错误，重点是当前命令解析实际使用哪个程序。

版本仍然可以显示 Python 3.14.9。环境切换不是以“版本号必须变化”为条件。

## 五、PowerShell 阻止激活脚本时的处理

### 1. 先区分输入位置错误与执行策略限制

在 Python 的 `>>>` 后输入激活脚本路径，可能得到：

```text
SyntaxError: invalid syntax
```

原因是把 PowerShell 命令交给了 Python，不是激活脚本本身一定写错。

如果已经处于 `PS ...>`，但看到“在此系统上禁止运行脚本”、`PSSecurityException` 等信息，则应检查 PowerShell 执行策略。该错误不能直接推出 `.venv` 损坏。

### 2. 查看设置和实际生效策略

```powershell
Get-ExecutionPolicy -List
Get-ExecutionPolicy
```

第一条列出不同作用范围的配置，第二条查看当前生效结果。

优先级从高到低为：`MachinePolicy`、`UserPolicy`、`Process`、`CurrentUser`、`LocalMachine`。前两项属于组策略范围，普通的进程级调整不能覆盖它们。

`Undefined` 表示该范围没有明确配置，不等于允许运行所有脚本。在 Windows 客户端的 Windows PowerShell 中，全部未配置时通常会使用默认的 `Restricted`，从而限制 `.ps1` 脚本运行，但仍允许输入普通命令。

### 3. 只在必要时使用进程级临时调整

确认没有更高优先级的组织策略限制，并确认要运行的是自己通过可信 Python 安装创建的激活脚本后，可以在当前 PowerShell 会话中设置：

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
```

这个命令会修改当前会话的策略，不是只读命令。`Process` 限定影响范围；`RemoteSigned` 允许本地创建的脚本运行，而被识别为互联网来源的脚本通常需要符合签名要求。它不证明脚本安全，仍需判断来源。

该范围通常不需要管理员权限，也不永久修改当前用户或所有用户的策略。不要为了激活环境，直接改成全局放行或盲目关闭安全检查。

设置后检查：

```powershell
Get-ExecutionPolicy
```

若输出为 `RemoteSigned`，再运行激活命令，并核查解释器路径。如果仍有错误，应继续根据具体原因处理，而不是不断扩大修改范围。

### 4. 策略设置与环境激活有各自的生命周期

关闭当前 PowerShell 进程后，本次 `Process` 范围设置随之结束；重新打开的独立终端应根据实际情况重新判断。扩展也可能在创建终端时自动应用自己的初始化操作，所以新终端仍应查看实际状态。

`deactivate` 只负责撤销虚拟环境激活时的调整，**不会撤销单独执行的 `Set-ExecutionPolicy` 设置**。退出激活状态后，同一 PowerShell 会话中的临时策略仍可能是 `RemoteSigned`。

同时，隐藏终端面板不等于结束终端进程。不能仅因为切换了面板，就认为进程级状态已经消失。

## 六、终端激活与 VS Code 解释器选择

### 1. 两种选择分别控制什么

| 操作 | 主要影响 |
|---|---|
| 在终端运行 `Activate.ps1` | 当前终端环境及命令查找 |
| 在 VS Code 中选择解释器 | Python 开发功能为项目选择的运行和分析环境 |
| 在命令中明确写出解释器路径 | 这次命令直接启动哪个解释器 |

它们有关联，但不能互相替代。终端激活成功，不等于 VS Code 必然已经切换；VS Code 选中了 `.venv`，也不意味着每个已有终端都已经同步激活。

### 2. 选择并验证 `.venv`

打开命令面板，执行 `Python: Select Interpreter`，选择：

```text
D:\Study\Python\01-environment\.venv\Scripts\python.exe
```

列表没有出现时，可以手动浏览该路径。多个显示项如果指向同一路径，不足以证明存在多个环境，不应据此删除目录。

随后在 `hello.py` 中执行 `Python: Run Python File in Terminal`，查看终端中的实际运行命令是否使用上述解释器。

VS Code 的环境扩展可能自动激活终端，也可能生成初始化命令。这些行为取决于扩展和配置，不能假定每个版本都会完全相同。应分别检查环境准备命令与真正执行 `.py` 文件的命令。

## 七、退出激活状态与不激活运行

### 1. 退出当前终端的激活状态

在已激活的 PowerShell 中执行：

```powershell
deactivate
```

这里是 PowerShell 中由激活脚本提供的命令，不是要在 Python 中调用一个 `deactivate()` 函数。

成功后通常不再显示 `(.venv)`，PATH 恢复到激活前的相应状态。示例中再次执行：

```powershell
where.exe python
```

第一个路径会恢复为基础解释器：

```text
C:\Development\Python\Python314\python.exe
```

这不删除 `.venv`，不卸载其中的软件包，也不自动更改 VS Code 已选定的解释器。

### 2. 不激活仍可以运行

在未激活的 PowerShell、项目根目录中执行：

```powershell
.\.venv\Scripts\python.exe --version
```

仍然能够启动虚拟环境解释器。运行源文件也可以明确指定它：

```powershell
.\.venv\Scripts\python.exe hello.py
```

如果 `hello.py` 是平方根示例，预期输出为：

```text
开始运行 Python 文件
9.0
文件运行结束
```

这是明确选择解释器的写法，不要求先激活。激活是便捷地调整当前 Shell 使用环境的方式，不是解释器运行的必要前提。

**未激活终端、虚拟环境仍存在、VS Code 仍选中 `.venv`，这三个状态可以同时成立。**
