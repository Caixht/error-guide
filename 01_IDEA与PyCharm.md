# 01 · IDEA / PyCharm

> JetBrains 家的东西长一个样：**红色波浪线 = 编辑器在提醒你（不影响运行）**；
> **Run / Build 窗口里的红字 = 真的跑不起来**。先分清这两件事，能省你一半时间。

---

## 一、先分清"三种红"

| 位置               | 长什么样                             | 性质                     | 要紧吗                        |
| ------------------ | ------------------------------------ | ------------------------ | ----------------------------- |
| 代码里的红色波浪线 | `Cannot resolve symbol 'Stirng'`     | 编辑器检查（inspection） | 通常要紧，但有时只是没 import |
| 底部 Run 窗口      | `Exception in thread "main" ...`     | 程序真的炸了             | 要紧，**这是程序的问题**      |
| 底部 Build 窗口    | `[ERROR] Failed to execute goal ...` | 构建/依赖失败            | 要紧，**这是环境的问题**      |

**第一条**：看 Build 输出时，**只看第一条 `[ERROR]`**，后面几十条基本都是它的连锁反应。

---

## 二、配置类（最常见的"打不开、跑不了"）

### `Project SDK is not defined` / `No SDK specified`

这个项目不知道用哪个 JDK / Python 来编译。
**为什么**：新导入的项目、换了电脑、镜像里没装 JDK。
**怎么办**：

- IDEA：`File → Project Structure → Project SDK` 选一个（没有就 `Add SDK → Download JDK`）
- PyCharm：`File → Settings → Project → Python Interpreter` 选一个解释器

### `Invalid Python interpreter selected for the project`

你选的解释器路径失效了（重装过 Python、虚拟环境被删了）。
**怎么办**：重新选解释器；如果是 `venv`，找到项目下 `.venv/Scripts/python.exe`。

### `Cannot start compilation: the output path is not specified for module 'xxx'`

没告诉 IDEA 编译出来的 class 放哪。
**怎么办**：`Project Structure → Modules → Paths`，设一个 output 目录。

### `The goal you specified requires a project to execute but there is no POM in this directory`

Maven 在说"我在这个文件夹里找不到 pom.xml"。
**怎么办**：**看 Run 配置里的 Working directory**，十有八九指错了目录。

---

## 三、依赖类（红色报错里最容易让人崩溃的一类）

### `Cannot resolve symbol 'String'`（连 String 都不认识）

不是你没 import，是**整个 JDK 没挂上**。
**怎么办**：回到上面那条 `Project SDK is not defined`。

### `Cannot resolve symbol 'xxx'`（业务类名）

三种可能，按顺序排：

1. 拼错了（`Stirng` / `lenght`）
2. 没 `import`（Alt+Enter 可以自动补）
3. 依赖没下下来（去 Maven 面板点刷新）

### `Could not resolve dependencies for project ...`

Maven 下载依赖失败。
**怎么办**：先看它下面那句 `Could not transfer artifact ... from/to central: Connection timed out` —— 这是网络问题，不是代码问题。换阿里云镜像，或在 `settings.xml` 配代理。

### `Could not transfer artifact ... Connection timed out` / `Read timed out`

连不上 Maven 中央仓库。
**怎么办**：换镜像源（阿里云 `https://maven.aliyun.com/repository/public`）；公司/学校网络可能要代理。

### `Non-resolvable parent POM`

`<parent>` 里写的那个父项目下载不到。
**怎么办**：检查 `pom.xml` 里 parent 的版本号是否存在；网络问题同上。

### `ERROR: Could not find a version that satisfies the requirement requests`（pip）

pip 在它的源里找不到这个包（或版本）。
**怎么办**：拼写、换国内源（`-i https://pypi.tuna.tsinghua.edu.cn/simple`）、确认 Python 版本对不对。

### `Unsupported class file major version 61` / `invalid target release: 17`

*JDK 版本和编译目标版本对不上*（61 = Java 17，52 = Java 8）。
**怎么办**：统一三处版本——Project SDK、`pom.xml` 的 `<maven.compiler.source/target>`、`Settings → Build → Compiler → Java Compiler`。

---

## 四、运行类

### `Port 8080 was already in use` / `Address already in use: bind`

8080 端口被别的程序占了（常见：上次的程序没关干净、另一个 Tomcat）。
**怎么办**：Windows 上 `netstat -ano | findstr 8080` 查 PID，再 `taskkill /PID xxx /F`；或改端口。

### `Process finished with exit code 1`

程序自己退出了，返回码 1 = 有错误。
**怎么办**：**这条本身没信息**，往上翻，找真正的异常或 `Caused by`。

### `Process finished with exit code 137`

进程被系统杀了（137 = 128+9，SIGKILL），通常是**内存不够**被 OOM Killer 干掉。
**怎么办**：看内存占用、调大堆（`-Xmx`）、或减小数据量。

### `java.lang.ClassNotFoundException: com.mysql.cj.jdbc.Driver`

运行时找不到这个类——**驱动 jar 没进 classpath**。
**怎么办**：Maven 里加 `mysql-connector-j` 依赖；确认 scope 不是 `provided`。

### `Cannot connect to the target VM, address: '127.0.0.1:xxxxx'`（Debug 时）

调试器连不上你的程序（端口被占 / 上次调试残留 / 防火墙）。
**怎么办**：关掉所有 Java 进程重来；换一个调试端口；检查防火墙。

---

## 五、IDE 自己出问题

### `java.lang.OutOfMemoryError: Java heap space`（IDEA 本尊卡死）

IDE 自己内存不够了，不是你的程序。
**怎么办**：`Help → Change Memory Settings`，把 `-Xmx` 调到 2048 或 4096，重启。

### 索引卡住 / `Indexing paused` / 全局搜索没结果

索引坏了或还没建完。
**怎么办**：`File → Invalidate Caches / Restart`，这是 JetBrains 的万能重启键。

### `Plugin "xxx" is not compatible with this version of IntelliJ IDEA`

插件版本比 IDE 新/旧。
**怎么办**：升级 IDE，或去插件市场找兼容版本，或先禁用。

### 文件右上角 `Read-only` / `Cannot save files: file is read-only`

文件权限是只读（常见于从压缩包/共享目录拷来的项目）。
**怎么办**：Windows 右键属性取消只读；或 `File → Make Directory Read-only` 取消勾选。

### Git：`Merge conflict` / `rejected (fetch first)`

本地和远程都改了同一处 / 远程有你没有的提交。
**怎么办**：先 `Pull`（或 `Fetch` 再 `Merge`）解决冲突；`rejected` 就先拉再推。

---

## 六、这一族的高频词

| 词                      | 意思                              |
| ----------------------- | --------------------------------- |
| resolve / unresolved    | 解析（能不能找到这个东西）        |
| SDK / JDK / interpreter | 开发工具包 / Java 开发包 / 解释器 |
| dependency / artifact   | 依赖 / 构件（jar 包）             |
| module                  | 模块                              |
| compile / compilation   | 编译                              |
| build                   | 构建                              |
| configuration           | 配置                              |
| invalid / incompatible  | 无效的 / 不兼容的                 |
| already in use          | 已被占用                          |
| exit code               | 退出码（0 才是成功）              |
| caches                  | 缓存                              |
| read-only               | 只读                              |
| in use                  | 使用中                            |

---

## 七、JetBrains 三步自救法

1. **Alt+Enter**：光标放在红字上按，能自动修一半的 `Cannot resolve symbol`
2. **Build → Rebuild Project**：清掉旧的编译产物再编一次
3. **File → Invalidate Caches / Restart**：玄学问题的终极手段（真的有用）
