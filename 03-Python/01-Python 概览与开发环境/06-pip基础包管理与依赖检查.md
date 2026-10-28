# pip 基础包管理与依赖检查

## 一、安装工具、软件包与导入

### 1. pip 的职责

`pip` 是 Python 软件包安装与管理工具。它可以查询安装信息、安装软件包，并检查已安装包的依赖是否满足。

`pip` 不是 Python 解释器，也不是虚拟环境创建工具。`venv` 负责创建环境，解释器负责执行代码，pip 负责针对所用环境进行包管理。这些工具可以配合使用，但职责不同。

### 2. 安装和导入是两个阶段

安装把软件包及相关文件准备到目标环境中。导入是 Python 程序运行时查找、加载模块并引入名称的操作。

```python
import requests
```

这条语句不会默认从网上下载安装缺少的库。反过来，安装好 `requests`，也不会让每个新程序自动获得可以直接使用的名称 `requests`。

**先确认目标环境具备需要的包，再在程序中按相应规则导入。**

### 3. PyPI 与 pip 的关系

PyPI（Python Package Index）是发布、发现和获取 Python 软件包的索引服务。pip 默认可以从 PyPI 获取安装材料，也可以按配置使用其他来源。

pip 是执行安装的工具，PyPI 是软件包来源服务，二者不是同一个东西。一个包能被下载，并不意味着它已经成为 Python 标准库；第三方库仍由相应项目独立维护。

## 二、先确定 pip 属于哪个 Python 环境

### 1. 为什么采用 `python -m pip`

下面这条命令通过当前解析到的 Python 解释器运行 pip：

```powershell
python -m pip --version
```

| 部分 | 含义 |
|---|---|
| `python` | 选择并启动一个 Python 解释器 |
| `-m pip` | 由这个解释器运行 pip 模块 |
| `--version` | 显示 pip 版本、位置及所关联的 Python 版本系列 |

只输入 `pip` 时，Shell 会查找名为 pip 的命令入口；它未必与当前预期的解释器一致。使用 `python -m pip` 可以将 pip 与前面的解释器联系起来，但仍然必须确认这个 `python` 指向谁。

因此，`python -m pip` 不是自动保证“装进项目环境”的万能写法：若 `python` 指向基础环境，pip 就通过基础解释器运行。

### 2. 用明确路径分别检查两个环境

以下命令使用 Windows PowerShell，当前目录为：

```text
D:\Study\Python\01-environment
```

检查基础安装：

```powershell
C:\Development\Python\Python314\python.exe -m pip --version
```

检查项目虚拟环境：

```powershell
.\.venv\Scripts\python.exe -m pip --version
```

虚拟环境路径以 `.` 开头，是相对于当前目录的路径。它不依赖终端是否激活，但依赖当前目录是否正确。

一组实际输出可以是：

```text
pip 26.2.2 from C:\Development\Python\Python314\Lib\site-packages\pip (python 3.14)
pip 26.2.2 from D:\Study\Python\01-environment\.venv\Lib\site-packages\pip (python 3.14)
```

两条输出的版本号相同，路径不同，说明访问的是不同环境中的 pip 安装位置。

`site-packages` 是 Python 安装第三方包时常用的目录。`Scripts` 中则有解释器和命令入口；两种目录的用途不同。

### 3. 安装目标的适用边界

使用某个环境中的解释器运行 pip，是普通包管理流程中明确环境归属的重要做法。若另外设置了改变安装目标的 pip 选项、环境变量或配置，则还需要核对那些设置。

本篇命令不指定额外的安装目标。安装后再通过包列表、`pip show` 的 `Location` 以及实际导入验证，确认结果比只记住一种命令写法更可靠。

## 三、使用 pip list 查看已安装软件包

### 1. 列表表示什么

```powershell
.\.venv\Scripts\python.exe -m pip list
```

输出通常包括 `Package` 和 `Version` 两列，分别表示 pip 能识别到的已安装软件包及其版本。

新建环境的一组示例输出是：

```text
Package Version
------- -------
pip     26.2.2
```

这里只出现 pip，并不表示这个环境只能使用 pip。标准库中的 `math` 等模块，不会作为需要分别安装的软件包列在这里；自己的 `.py` 文件也不是这份清单的统计对象。

**pip list 不是这个解释器全部可用功能或全部可导入模块的目录。**

### 2. 比较两个环境

```powershell
C:\Development\Python\Python314\python.exe -m pip list
.\.venv\Scripts\python.exe -m pip list
```

如果两次都只显示 pip，只能说明当前列出的包名、版本相同，不能据此判断两个环境共用一份安装。

正常的基础列表查询读取本地安装信息，不安装、更新或卸载软件包。对于本例使用的 pip，普通 `pip list` 也不需要为了列出这些信息去查询在线最新版本；带有其他查询选项时，应按具体命令判断联网行为。

## 四、安装 requests 并观察依赖

### 1. 安装命令与影响范围

在项目根目录中执行：

```powershell
.\.venv\Scripts\python.exe -m pip install requests
```

`install` 表示安装操作，`requests` 是目标包名。这条命令会修改虚拟环境中的软件包文件，通常会联网获取需要的安装材料；与前面的只读查询不同。

命令没有指定版本时，pip 会结合来源、Python 兼容性、依赖约束及现有安装状态选择候选版本。不能据此断言任何时候都一定安装同一个版本，也不能认为重复运行安装命令就必然升级到最新版本。

### 2. 为什么会安装不止一个包

某个库完成自身功能时，可能需要其他库提供支持。这种“需要其他软件包”的关系称为依赖关系。

一组 Requests 安装结果如下，版本来自具体环境示例，不代表其他时间的最新版本：

| 软件包 | 示例版本 |
|---|---|
| requests | 2.34.2 |
| urllib3 | 2.8.0 |
| certifi | 2026.10.23 |
| idna | 3.20 |
| charset-normalizer | 3.5.2 |

这里的 urllib3 提供 HTTP 通信相关能力，certifi 提供证书信任集合，idna 支持国际化域名处理，charset-normalizer 支持文本编码识别相关工作。它们与 Requests 一起出现在安装日志中，并不是随意额外安装了一批无关软件。

在没有跳过依赖处理等特殊选项时，pip 会解析目标包的依赖要求，并为需要的依赖选择、准备相应版本。

### 3. 阅读安装日志

典型日志可以按阶段理解：

| 输出内容 | 大意 |
|---|---|
| `Collecting ...` | 正在收集目标包或依赖的安装信息 |
| `Downloading ...` | 正在下载安装所需的文件或信息 |
| `Installing collected packages ...` | 正在将选定内容安装到环境中 |
| `Successfully installed ...` | 这次安装报告成功，并列出相应包 |

不能因为中途出现一条 WARNING 就断定整个安装失败，也不能只看前面开始下载就认定已经安装成功。应阅读最终状态，并结合安装后的查询核对。

### 4. SSL 连接中断与重试

安装过程中可能出现：

```text
WARNING: Retrying ...
SSLEOFError
[SSL: UNEXPECTED_EOF_WHILE_READING]
```

它表示一次加密连接异常结束。单凭这段信息不能断定具体原因一定是代理、证书或某一家网络服务；也不能把所有 SSL 错误都归为同一种问题。

如果后续重试成功、最终报告安装完成，并且查询与导入正常，本次无需仅因该条警告重新安装。反复失败时，应保留完整日志，结合实际网络、代理和软件包来源调查，不应直接关闭 HTTPS 证书验证。

### 5. 用两个环境的列表观察隔离

安装后再分别查询：

```powershell
.\.venv\Scripts\python.exe -m pip list
C:\Development\Python\Python314\python.exe -m pip list
```

在默认隔离、没有其他安装目标配置的示例中，虚拟环境包含 pip、requests 和四个依赖，共六个已列出的包；基础环境仍只列出 pip。

这说明这次安装修改的是项目环境中的软件包集合，基础环境的列表没有因该操作同步增加 Requests。这里的数量仍然不是 Python 的全部功能数量。

## 五、通过程序验证导入与版本信息

### 1. 完整示例

在项目根目录创建 `check_requests.py`：

```python
import requests

print("requests 导入成功")
print("requests 版本号为：", requests.__version__)
```

用 VS Code 运行时，应选择项目虚拟环境，并核对终端实际调用的解释器路径；也可以在 PowerShell 中明确运行：

```powershell
.\.venv\Scripts\python.exe check_requests.py
```

在安装 Requests 2.34.2 的示例环境中，预期输出：

```text
requests 导入成功
requests 版本号为： 2.34.2
```

如果安装版本不同，版本输出也会相应变化。

### 2. 为什么这些输出能够说明导入成功

程序先执行导入，再执行后面的输出。在这个没有异常处理分支的简单文件中，导入若失败，会中断到达后续语句的正常执行。因此，在相应运行命令下看到后面的输出，可以确认这次导入没有失败。

`requests.__version__` 通过模块提供的属性读取版本字符串。属性访问不等于函数调用，所以这里没有为 `__version__` 加括号。

第二个 `print` 用逗号分隔了两个传入值。默认情况下，print 会在它们之间加入一个空格，再在行末换行。因此，字符串末尾不必再手动加一个额外空格来达到分隔效果。

这个例子不会发送 HTTP 请求。导入成功和版本信息正确，不能证明网络连接、服务器证书或所有 HTTP 功能都已经验证。

### 3. 环境选择错误的典型表现

同一源文件交给另一个没有准备 Requests、且无法从其他位置找到它的 Python 环境时，可能得到：

```text
ModuleNotFoundError: No module named 'requests'
```

不能只因为代码没有改动，就认为两个环境执行时必然有相同结果。遇到此类错误，应先核对实际解释器和包位置，而不是马上重装编辑器或把库安装到所有环境。

代码规范方面，访问版本信息使 `requests` 名称真正被使用，解决的是未使用导入问题；导入区空行是否符合 Ruff 的要求，仍需另外检查。

## 六、使用 pip show 读取安装位置与依赖方向

### 1. 查看一个包的详细信息

```powershell
.\.venv\Scripts\python.exe -m pip show requests
```

这条命令读取已安装软件包的信息，不是在网上搜索最新版本。

| 字段 | 含义 |
|---|---|
| `Name` | 软件包名称 |
| `Version` | 当前安装版本 |
| `Summary` | 简要说明 |
| `Location` | 软件包的安装位置 |
| `Requires` | 这个包声明依赖的其他包 |
| `Required-by` | 当前环境中哪些已安装包声明依赖它 |

作者、许可证等字段也可能出现；某个说明字段为空，不足以说明安装失败。

### 2. 用 Location 确认目标环境

示例结果：

```text
Location: D:\Study\Python\01-environment\.venv\Lib\site-packages
```

这个位置与项目 `.venv` 对应。它不是 `check_requests.py` 的保存位置，也不是解释器所在的 `Scripts` 目录。

检查安装是否进入预期环境时，Location 是一项直接信息，应与实际运行命令中的解释器路径结合阅读。

### 3. Requires 与 Required-by 从相反方向描述关系

Requests 的信息包含：

```text
Requires: certifi, charset_normalizer, idna, urllib3
Required-by:
```

再查看：

```powershell
.\.venv\Scripts\python.exe -m pip show urllib3
```

示例结果包含：

```text
Requires:
Required-by: requests
```

两组信息说明同一条依赖关系：**requests 依赖 urllib3**。

| 观察对象 | 字段 | 读法 |
|---|---|---|
| requests | `Requires` 中出现 urllib3 | requests 需要 urllib3 |
| urllib3 | `Required-by` 中出现 requests | urllib3 被 requests 需要 |

它们不是在描述两个包互相依赖。也不能因为 urllib3 的 Requires 为空，就推断它运行时完全不需要 Python 标准库或操作系统。

### 4. Required-by 不统计自己的脚本

`check_requests.py` 中写了 `import requests`，并不会使 `Required-by` 自动出现这个文件名。pip 读取的是已安装软件包之间声明的依赖关系，不是扫描全部项目源代码来统计谁使用了某个模块。

因此，Required-by 为空只意味着在相应安装记录中没有列出依赖它的其他包，不能推出“没有任何程序使用它，可以安全删除”。

## 七、使用 pip check 检查依赖要求

### 1. 依赖不仅有名称，还可能有版本条件

假设软件包 A 需要软件包 B，且要求 B 至少为 2.0。那么 B 未安装与 B 已安装但版本为 1.0，都会造成依赖要求不满足。

所以，pip list 中出现了某个包名，只能证明列表中有这个安装记录，不能单独证明它的版本满足所有已安装包的要求。

### 2. 执行检查

```powershell
.\.venv\Scripts\python.exe -m pip check
```

这条命令针对该虚拟环境检查已安装包的依赖兼容性，不会自动升级、安装或卸载包。

如果未发现问题，输出为：

```text
No broken requirements found.
```

这里的含义是：检查没有发现已安装软件包的声明依赖缺失或不兼容问题。它不是“整个项目的所有要求都已验证”的声明。

### 3. 三种验证各有范围

| 验证 | 主要回答的问题 | 不能由它直接证明的事情 |
|---|---|---|
| 安装结果、pip list、pip show | 装了什么、版本和位置是什么 | 程序一定运行正确 |
| 运行 check_requests.py | 该解释器这次能否导入 Requests、读取版本 | 网络功能全部可用 |
| pip check | 已安装包的声明依赖是否兼容 | 业务逻辑、网络状态及安全性都没有问题 |

依赖检查通过不等于漏洞检查通过，也不保证检查工具理解了项目全部实际需求。如果某个包没有被安装、也没有被任何已安装包声明为依赖，pip check 不会仅凭自己的源文件使用了它，就自动知道项目缺了这个包。

## 八、按问题选择查询命令

下面都以当前目录为项目根目录为前提：

| 目的 | 命令 |
|---|---|
| 查看虚拟环境中的 pip 版本和位置 | `.\.venv\Scripts\python.exe -m pip --version` |
| 查看已安装软件包和版本 | `.\.venv\Scripts\python.exe -m pip list` |
| 在该环境中安装 Requests | `.\.venv\Scripts\python.exe -m pip install requests` |
| 查看 Requests 详细信息 | `.\.venv\Scripts\python.exe -m pip show requests` |
| 查看 urllib3 及其反向依赖 | `.\.venv\Scripts\python.exe -m pip show urllib3` |
| 检查声明依赖是否满足 | `.\.venv\Scripts\python.exe -m pip check` |

安装命令会修改环境；本表中的版本、列表、详细信息和依赖检查命令不执行软件包的安装、升级或卸载。

**包管理操作的基本顺序是：明确解释器和目标环境，执行必要操作，再从安装记录、依赖检查与实际程序运行三个角度验证结果。**
