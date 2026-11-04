# 报错阅读手册 · 总目录

> 面向：大一信息类专业，英语几乎零基础，但要天天面对英文报错。
> 目标不是"背会所有报错"，而是**拿到一屏红字，30 秒内知道先看哪一行、该去改什么**。
>
> 配套文件：`核心词汇_零基础程序员版.txt`
> 那本是"认词"，这套是"读句"。

---

## 一、文件清单（按顺序看）

| 文件                   | 管什么                                   | 什么时候翻                     |
| ---------------------- | ---------------------------------------- | ------------------------------ |
| `00_怎么读一条报错.md` | **方法总纲**：四种长相、四步法、怎么搜   | **先看这个**，看一遍就够       |
| `01_IDEA与PyCharm.md`  | JetBrains 家：SDK、依赖、端口、卡顿      | 项目打不开、红波浪线、跑不起来 |
| `02_Java报错.md`       | Java 语言本身：NPE、越界、类找不到       | 运行窗口一堆 `at ...`          |
| `03_Python报错.md`     | Python Traceback：缩进、类型、模块、编码 | PyCharm 跑脚本报红             |
| `04_MySQL报错.md`      | 数字码风格：1064/1045/1146/1062…         | 数据库连不上、SQL 跑不通       |
| `05_虚拟机与Linux.md`  | VMware 开不了机、Linux 命令报错          | 装虚拟机、连服务器             |
| `06_网络与HTTP.md`     | 连不上、下不动、推不上去                 | pip/npm/git 失败、接口 4xx/5xx |

---

## 二、20 秒速查表（看到这句，先想到这个）

### 通用

| 你看到                              | 它在说                        | 先做                            |
| ----------------------------------- | ----------------------------- | ------------------------------- |
| `Traceback (most recent call last)` | Python 出事了，**看最后一行** | 找最后一个 `File "你的文件.py"` |
| `Exception in thread "main"`        | Java 出事了                   | 找 `Caused by` 或第一个 `at`    |
| `ERROR 数字`                        | MySQL 出事了                  | 直接搜 `mysql 数字`             |
| `exit code 1`                       | 程序自己退了，**这条没信息**  | 往上翻真正的异常                |
| `exit code 137`                     | 被系统杀了，通常内存不够      | 看内存 / `-Xmx`                 |
| `Caused by:`                        | **根因在这里**                | 从最下面那个 `Caused by` 读     |

### Java / IDEA

| 你看到                                 | 一句话                | 去改哪                              |
| -------------------------------------- | --------------------- | ----------------------------------- |
| `NullPointerException`                 | 拿空对象用了          | 那一行前打印这个变量                |
| `cannot find symbol`                   | 我不认识这个名字      | 看 `symbol:` 那行；拼写/import/依赖 |
| `ArrayIndexOutOfBoundsException`       | 下标越界（从 0 开始） | 循环条件 `<` 不是 `<=`              |
| `Cannot resolve symbol`                | 找不到这个类/方法     | Alt+Enter；查 import；刷 Maven      |
| `Project SDK is not defined`           | 没配 JDK              | Project Structure → SDK             |
| `Could not resolve dependencies`       | 依赖下载失败          | 换阿里云镜像                        |
| `Port 8080 was already in use`         | 端口被占              | `netstat -ano                       |
| `Unsupported class file major version` | JDK 版本对不上        | 统一三处版本                        |
| `incompatible types`                   | 类型不匹配            | 显式转换                            |

### Python / PyCharm

| 你看到                                | 一句话                             | 去改哪                      |
| ------------------------------------- | ---------------------------------- | --------------------------- |
| `IndentationError`                    | 缩进不对                           | 统一 4 空格，别混 Tab       |
| `SyntaxError`                         | 语法错                             | 查中文标点、括号、冒号      |
| `NameError`                           | 名字没定义                         | 拼写 / 作用域 / import      |
| `TypeError: unsupported operand`      | 类型不能这么算                     | `int()` / `str()` 转换      |
| `'NoneType' object has no attribute`  | 这变量是 None                      | 打印它，查函数有没有 return |
| `IndexError: list index out of range` | 列表没这么长                       | 先判空 / 用 for in          |
| `KeyError`                            | 字典没这个键                       | 用 `.get()`，打印 `.keys()` |
| `ModuleNotFoundError`                 | 没装这个包，**或装到别的解释器了** | 看右下角解释器，用它装      |
| `FileNotFoundError`                   | 路径不对                           | `print(os.getcwd())`        |
| `UnicodeDecodeError`                  | 编码不对                           | `encoding="utf-8"`          |

### MySQL

| 你看到       | 一句话               | 先查                         |
| ------------ | -------------------- | ---------------------------- |
| `ERROR 1064` | SQL 语法我看不懂     | `near` 前面那个词 + 中文引号 |
| `ERROR 1045` | 账号或密码错         | 命令行先连一次               |
| `ERROR 2003` | 连不上服务器         | 服务起没起 / 端口 / 防火墙   |
| `ERROR 1146` | 表不存在             | 报错里的 `库名.表名` 对不对  |
| `ERROR 1054` | 字段不存在           | 字段拼写 / 表对不对          |
| `ERROR 1062` | 主键重复             | 换 id / 用 upsert            |
| `ERROR 1366` | 字符集不对（存中文） | 全改 `utf8mb4`               |
| `ERROR 1452` | 外键指向的记录不存在 | 先插主表                     |
| `ERROR 1205` | 等锁超时             | 有事务没提交                 |

### 虚拟机 / Linux

| 你看到                    | 一句话              | 怎么办                          |
| ------------------------- | ------------------- | ------------------------------- |
| `VT-x is disabled`        | BIOS 没开虚拟化     | 进 BIOS 开 VT-x / SVM           |
| `Hyper-V 不兼容`          | Hyper-V 抢了虚拟化  | 关 Hyper-V + 内存完整性         |
| `被锁定` / `is locked`    | 上次没正常关机      | 删 `.lck` 文件夹                |
| `command not found`       | 没这个命令          | 装它 / 查 PATH / 查拼写         |
| `Permission denied`       | 没权限              | `chmod +x` / `sudo`             |
| `bad interpreter: ^M`     | 脚本是 Windows 换行 | `dos2unix` / `sed -i 's/\r$//'` |
| `No space left on device` | 磁盘或 inode 满了   | `df -h` / `df -i`               |
| `Killed`                  | 内存不够被杀        | `dmesg                          |

### 网络

| 你看到                                   | 一句话                  | 怎么办                |
| ---------------------------------------- | ----------------------- | --------------------- |
| `UnknownHost` / `Could not resolve host` | 域名解析不了            | 换 DNS / 关代理       |
| `Connection refused`                     | 端口没人监听            | 服务没起 / 端口错     |
| `Connection timed out`                   | 路不通                  | 防火墙 / 安全组 / VPN |
| `Connection reset`                       | 被对面掐断              | 看服务端日志          |
| `PKIX path building failed`              | 证书验不过              | 导证书 / 临时信任     |
| `401 / 403`                              | 没登录 / 没权限         | 查 token、角色        |
| `404`                                    | 地址不存在              | 查路由和路径          |
| `500`                                    | 服务端炸了              | 看后端日志            |
| `502 / 504`                              | 网关后面的服务挂了/太慢 | 跳过 Nginx 看后端     |

---

## 三、怎么用这套东西（别当词典背）

1. **第一次**：只读 `00_怎么读一条报错.md`，20 分钟，建立"先看哪"的反射。
2. **每次遇错**：先在速查表里找**长得像**的那一行 → 再去对应文件看完整条目。
3. **每周**：把你这一周真实遇到的报错抄进一个 `我的报错本.md`，只记三样：
   - 原始报错（第一行 + 最后一行）
   - 一句话中文意思
   - 最后怎么解决的
4. **两个月后**：你的报错本会比这整套手册更有用，因为那全是你自己的。

---

## 四、三条底层认知

1. **报错是坐标，不是判决。** 它给你文件名和行号，是让你去看那一行，不是让你害怕。
2. **90% 的报错跟你的业务代码无关。** 是环境、版本、依赖、权限、编码——这些都有固定解法。
3. **报错变了，就是进步。** 从 `NullPointerException` 变成 `IllegalArgumentException`，说明你修对了一半。最怕的是报错一直不变。
