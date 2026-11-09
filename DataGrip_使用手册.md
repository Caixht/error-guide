# DataGrip 使用手册（Windows · 中文 · 2026.3 校对版）

> **版本**：已按 JetBrains 官方文档（help/datagrip，页面更新日期 2026-10-30 ~ 2026-11-05）核对，对应当前正式版 **DataGrip 2026.3**。
> 兼容：2025.3 / 2026.1 / 2026.2 基本一致；更早的 2024.x 菜单名略有差别（新旧名称对照见第 0 章）。
> 配套练习库：`atguigudb`（讲义 1.10 的种子数据，20 名员工 / 7 个部门）
> 目标：连库 → 写 SQL → 执行 → 看结果 → 改数据 → 导数据，一条龙。

---

## 第 0 章 先看这里：2026.3 的界面叫什么（新手最容易懵的四处）

| 你以前听说的名字 | 2026.3 的正式名字 | 备注 |
|---|---|---|
| Database 工具窗 | **Database Explorer**（数据库资源管理器） | 就是左边那棵树 |
| 结果面板 / Results | **Database Output**（数据库输出）工具窗，里面有 **Result** 和 **Output** 两个标签 | 查询结果在 Result，报错/影响行数在 Output |
| 结果只能在下方面板看 | 现在还可以开 **In-Editor Results**（编辑器内结果，结果显示在 SQL 下方） | 工具栏有个按钮切 |
| 经典界面 | **新 UI 是默认**；想要老样子要装 **Classic UI** 插件 | 网上老教程多是经典界面 |

> 所以：**左边 = Database Explorer，下面 = Database Output**。记住这两个名字，看官方文档就不会迷路。

---

## 第 1 章 五分钟连上数据库

1. 打开 **Database Explorer**：`View → Tool Windows → Database Explorer`（或点最左边竖排的数据库图标）。
2. 点面板左上角 **`+` → Data Source → MySQL**。
3. 填连接信息：

| 字段 | 填什么 |
|---|---|
| Name | 随便起，如 `本地MySQL` |
| Host | `localhost` |
| Port | `3306` |
| User | `root` |
| Password | 你的 root 密码（可勾 Save password） |
| Database | 可留空 |

4. 首次会提示 **Download missing driver files**，点它自动下 MySQL 驱动（要联网）。
5. 点 **Test Connection**，出现 `Succeeded` 后点 **OK**。
6. 在数据源的 **Schemas** 选项卡里勾选要显示的库（如 `atguigudb`）。**不勾就不显示**——"我的库去哪了"九成是这个原因。

> 连不上先看第 12 章：90% 是时区（serverTimezone）或公钥（allowPublicKeyRetrieval）问题。

---

## 第 2 章 界面地图

```
┌──────────────┬───────────────────────────┬────────────┐
│ Database      │ 查询控制台（写 SQL 的地方）  │ 工具栏      │
│ Explorer      │                           │ ▶ 执行      │
│ （左树）       │                           │ Tx 事务模式 │
├──────────────┴───────────────────────────┴────────────┤
│ Database Output：  [Result]  [Output]                   │
└────────────────────────────────────────────────────────┘
```

- **左树**：数据源 → 库 → 表 / 视图 / 存储过程；展开表可见 Columns / Keys / Indexes。
- **中间**：每个标签页是一个查询控制台（Query Console）。
- **下面**：Database Output，`Result` 看数据，`Output` 看日志和报错。
- **Tx**：事务模式（Auto / Manual），第 6 章讲。

---

## 第 3 章 SQL 到底在哪写（重点）

### 3.1 新建查询控制台的 5 种方式（官方给的）

| 方式 | 操作 |
|---|---|
| **最快** | 左树点中数据源 → 按 **`F4`**（打开它的默认控制台） |
| 快捷键 | 点中数据源 → **`Ctrl + Shift + Q`** |
| 菜单 | `File → New → Query Console` |
| 右键 | 右键数据源 → **New → Query Console** |
| Alt | 点中数据源 → `Alt + Insert` → Query Console |

> 注意：**查询控制台不属于项目文件**，它存在 IDE 的内部目录 `Scratches and Consoles | Database Consoles`（`Alt + 4` 打开 Files 窗口能看到），所以别在项目目录里找它。

### 3.2 选库（schema 下拉框）
控制台工具栏**右上角有个 `<schema>` 下拉框**，选 `atguigudb`，之后写 `SELECT * FROM employees` 就不用带库名。

### 3.3 打开现成的 `.sql` 文件
`File → Open` 或把文件拖进窗口 → 右上角选关联的数据源 → 才能执行。
**限制**：超过 **20 MB** 的 SQL 文件只加载前 **2.5 MB**（官方说明），大脚本要拆开跑。

### 3.4 多条语句怎么写、怎么只跑一条
- 用 `;` 分隔；
- `Ctrl + Enter` 默认执行**光标所在的那一条**；
- 选中一段再 `Ctrl + Enter` = 只执行选中部分；
- 一次执行多条时会弹 **Statements** 选择窗，`Enter` 跑高亮那条，也可以选 "run all"。

---

## 第 4 章 执行 SQL 的完整姿势

### 4.1 执行 / 取消
| 动作 | 操作 |
|---|---|
| 执行光标所在语句 | **`Ctrl + Enter`**（或工具栏 ▶ Execute） |
| 执行选中片段 | 选中 → `Ctrl + Enter` |
| 取消正在跑的语句 | **`Ctrl + F2`** |
| 跑整个 `.sql` 文件 | 右键文件 → **Run**，或 `Ctrl + Shift + F10` |
| 结果直接写进文件 | 右键 `SELECT` 语句 → **Execute to File** → 选格式 |

### 4.2 查刚才跑过什么
- 工具栏的 **Browse Query History** 按钮，或 **`Ctrl + Alt + E`**：按时间列出所有执行过的语句，双击可粘回控制台。

### 4.3 不离开编辑器看表结构
把光标放在表名上按 **`Ctrl + Q`**（Quick Documentation），会弹出建表 DDL、字段类型、注释。比去左边翻树快得多。

### 4.4 一个容易踩的概念：Resolve mode（解析模式）
- 查询控制台默认 **Playground**：每条语句各自独立解析，`USE xxx` 不影响下一条的补全；
- `.sql` 文件默认 **Script**：`USE` 会改变后面语句的解析上下文。
- 表现就是"我明明 `USE atguigudb` 了，下面还是红波浪线"——这是模式差异，不是 SQL 写错了。

---

## 第 5 章 看结果（Database Output）

### 5.1 两个标签
- **Result**：查询结果表格，标签标题通常是"库.表"；
- **Output**：执行日志、`Affected rows`、**报错信息**（先看这里）。

### 5.2 结果表格工具栏（2026.3 都有）
| 按钮 | 用途 |
|---|---|
| 分页 / 页大小 | 结果按页显示，可改每页行数（不是全表都加载） |
| **Reload Page（`Ctrl + F5`）** | 重新从数据库取数（改完数据刷一下） |
| **Cancel Running Statements（`Ctrl + F2`）** | 停掉正在跑的查询 |
| **Enable Local Filter** | 每列出现筛选框，客户端过滤，不发 SQL |
| **Record View** | 侧边栏看/改单条记录 |
| **Go to DDL** | 直接跳到这张表的建表语句 |
| **Find on Current Page（`Ctrl + F`）** | 在当前页里搜 |
| **SQL** | 看产生这个结果集的 SQL，可一键重跑 |

### 5.3 复制 vs 导出（很多人搞混）
- `Ctrl + C`：**只复制当前页选中的行**；
- 复制整页：点一个单元格 → `Ctrl + A` → `Ctrl + C`；
- 复制整张表：右键单元格 → **Export Table to Clipboard**；
- 导出成文件：工具栏 **Export Data** 图标 → `Extractor` 选 **Excel (xlsx) / CSV / JSON / SQL Inserts** → **Export to File**。

> Excel 导出是 2026.x 内置的 extractor，不需要装插件。

---

## 第 6 章 改数据与事务（最容易翻车）

### 6.1 打开数据编辑器
双击表名，或选中表按 `F4`，或右键 → **Edit Data**。打开的是 **Data 标签**（和查询控制台是两个东西）。

### 6.2 改数据的完整流程
1. 双击单元格改值 → 回车；
2. 点工具栏 **Submit（`Ctrl + Enter`）** —— **不点 Submit 等于没改**；
3. 改错了想撤销：**Revert Selected（`Ctrl + Alt + Z`）**；
4. 提交前想先看会发什么 SQL：点 **Preview Pending Changes**。

### 6.3 事务模式（Tx）
工具栏的 **Tx** 按钮有两个模式：

| 模式 | 行为 | 什么时候用 |
|---|---|---|
| **Auto**（默认够用） | 点 Submit 时自动提交到数据库 | 学习、练习 |
| **Manual** | Submit 只是放进事务，还要点 **Submit and Commit（`Ctrl + Alt + Shift + Enter`）** 才真正生效；不想提交就 **Roll Back** | 改生产数据、要能反悔 |

> 练习阶段建议：**Auto 模式 + 改完立刻 Submit + 不确定就 Roll Back**。这样不会出现"我改的为什么不生效"或"我改错了却已经生效"。

### 6.4 增删行
- 加一行：`Alt + Insert`
- 删选中行：`Ctrl + Y`
- 刷新：`Ctrl + F5`

---

## 第 7 章 表结构、DDL 与 ER 图

### 7.1 图形化建表 / 改表
- 新建：右键库 → **New → Table**，填列名/类型/主键/默认值 → **Execute**；
- 修改：右键表 → **Modify Table…**，改完 → **Execute**；
- 学习阶段仍然建议**手写 `CREATE TABLE` 再 `Ctrl+Enter`**，考试和面试都考手写。

### 7.2 看 / 导出建表语句（DDL）
| 需求 | 操作 |
|---|---|
| 单张表的 DDL 复制走 | 右键表 → **SQL Scripts → Generate DDL to Clipboard** |
| DDL 直接开在控制台里改 | 右键表 → **SQL Scripts → Generate DDL to Query Console** |
| 批量生成整个库的 DDL / 存成一堆文件 | 右键库 → **SQL Scripts → SQL Generator…（`Ctrl + Alt + G`）**，右上有"复制/保存到文件/在控制台打开"，还能按 schema、按对象拆文件 |
| 在表上直接改列定义 | 右键表 → **Modify Table…**，或打开 DDL 脚本改（DataGrip 支持直接改对象源码） |

> MySQL 的存储过程/函数导出时，SQL Generator 里可以勾 **Skip DEFINER clause**，否则换台机器执行常报权限错。

### 7.3 ER 图
- 右键数据源 / 库 / 表 → **Diagrams → Show Diagram**（功能在 Diagrams 插件里，默认已启用）；
- 图里可以拖表进来、按 `Ctrl + F` 搜节点、`Alt` 悬停放大；
- 导出图片：右键图 → **Export Diagram → Export to Image…**（PNG），或导出 Mermaid / PlantUML / draw.io 格式。

---

## 第 8 章 导入导出

### 8.1 导出数据
1. 右键表（可多选）→ **Import/Export → Export Data to File…**；
2. 在 Export Data 对话框里选 **Extractor**：`CSV` / `Excel (xlsx)` / `JSON` / `SQL Inserts` …
3. 勾 `Add column header`（要表头就勾）、选输出文件 → **Export to File**。
4. 结果集也可以导出：在 Result 标签工具栏点 **Export Data** 图标，流程一样。

### 8.2 导入 CSV
右键表 → **Import/Export → Import Data from File…** → 选文件 → 映射列 → 导入。
（Excel 先另存为 CSV 最省事。）

### 8.3 整库备份 / 恢复（mysqldump）
- 备份：右键库 → **Export with 'mysqldump'**，在 `Path to executable` 填 **mysqldump 全路径**，例如
  `C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqldump.exe`，再填输出文件 → **Run**；
- 恢复：新建一个空库 → 打开 dump 的 `.sql` → `Ctrl + Shift + F10` 整个跑；
- 大文件（>20 MB）建议用命令行，DataGrip 只加载前 2.5 MB：
  ```
  mysql -u root -p atguigudb < D:\backup\atguigudb.sql
  ```

> 记住：**结构（DDL）和数据是分开导的**——SQL Generator 出结构，Export Data 出数据。

---

## 第 9 章 Navicat ↔ DataGrip 对照表（按 2026.3 重写）

> Navicat 一列是 17 的常见默认键位（Ctrl+Q 新建查询、Ctrl+R 运行、F5 刷新），以你机器上的菜单为准。

| 你要做的事 | Navicat 里 | DataGrip 2026.3 里 |
|---|---|---|
| 新建连接 | 连接 → MySQL | `+` → Data Source → MySQL |
| **新建查询** | 右键连接 → 新建查询（Ctrl+Q） | 点中数据源 → **`F4`**，或 **`Ctrl + Shift + Q`** |
| 执行当前语句 | 运行（Ctrl+R） | **`Ctrl + Enter`** |
| 停止执行 | （任务管理器…） | **`Ctrl + F2`** |
| 执行整个脚本 | 运行 SQL 文件 | 打开 `.sql` → `Ctrl + Shift + F10` |
| 刷新数据 | F5 | **`Ctrl + F5`**（Reload Page） |
| 看表数据 | 双击表 | 双击表 / `F4` / 右键 Edit Data |
| 改完提交 | 自动或点 √ | **Submit（`Ctrl + Enter`）**，Manual 模式再 Submit and Commit |
| 撤销改动 | 直接改回 | **Revert Selected（`Ctrl + Alt + Z`）** |
| 设计表 | 右键 → 设计表 | 右键 → **Modify Table…** |
| 新建表 | 右键 → 新建表 | 右键库 → **New → Table** |
| 查看 DDL | 右键 → 对象信息 | 右键 → **SQL Scripts → Generate DDL to Clipboard** |
| 导出向导 | 右键 → 导出向导 | 右键 → **Import/Export → Export Data to File…** |
| 导入向导 | 右键 → 导入向导 | 右键 → **Import/Export → Import Data from File…** |
| ER 图 | 右键库 → 逆向数据库到模型 | 右键 → **Diagrams → Show Diagram** |
| 美化 SQL | 美化 SQL 按钮 | **`Ctrl + Alt + L`** |
| 执行历史 | 日志 | **`Ctrl + Alt + E`**（Query History） |
| 看表结构提示 | 悬停 | 光标放表名 → **`Ctrl + Q`** |
| 命令行 | 工具 → 命令行界面 | 底部 Terminal（`Alt + F12`） |

**一句话记**：Navicat 的"新建查询"= 点中连接按 **F4**；"运行"= **Ctrl+Enter**；"刷新"= **Ctrl+F5**。

---

## 第 10 章 快捷键（已在 2026.3 文档核对）

| 快捷键 | 作用 | 频率 |
|---|---|---|
| `Ctrl + Enter` | 执行语句 / 数据编辑器里是 Submit | ★★★★★ |
| `Ctrl + Shift + Q` | **新建查询控制台** | ★★★★★ |
| `F4` | 打开数据源默认控制台；对表则是打开数据 | ★★★★★ |
| `Ctrl + Alt + L` | 格式化 SQL | ★★★★★ |
| `Ctrl + F5` | 重新加载数据（刷新） | ★★★★ |
| `Ctrl + F2` | 取消正在执行的语句 | ★★★★ |
| `Ctrl + Space` | 手动触发补全 | ★★★★ |
| `Ctrl + Q` | 快速查看表结构/DDL | ★★★★ |
| `Ctrl + Alt + E` | 查询历史（Query History） | ★★★★ |
| `Ctrl + B` | 跳到定义（表/字段） | ★★★ |
| `Ctrl + Alt + G` | SQL Generator（批量出 DDL） | ★★★ |
| `Ctrl + Alt + Z` | 撤销单元格改动（Revert Selected） | ★★★ |
| `Ctrl + Alt + Shift + Enter` | Submit and Commit（Manual 模式） | ★★★ |
| `Shift + Shift` | 全局搜索 | ★★★ |
| `Ctrl + /` | 注释 / 取消注释 | ★★★ |
| `Ctrl + D` / `Ctrl + Y` | 复制当前行 / 删除当前行 | ★★ |
| `Alt + Insert` | 新建（行、对象、控制台） | ★★ |
| `Ctrl + Shift + F10` | 运行整个文件 | ★★ |
| `Alt + 4` | Files 工具窗（找查询控制台） | ★★ |
| `Alt + F12` | 终端 | ★★ |
| `Ctrl + Alt + S` | 设置 | ★★ |

> 真的只要记三个：**`Ctrl+Enter` 跑、`Ctrl+Shift+Q` 新控制台、`Ctrl+Alt+L` 排版**。

---

## 第 11 章 建议改的设置（`Ctrl + Alt + S`）

| 想改什么 | 设置路径 |
|---|---|
| 每页显示多少行结果 | `Database → Data Editor and Viewer` → **Limit page size to** |
| 执行行为（只跑光标语句/弹窗选择） | `Database → Query Execution` |
| 控制台标签页名字 | `Database → Query Files and Consoles` |
| `select` 自动变大写 | `Editor → Code Style → SQL → Keywords` 选 **UPPER** |
| 字号、行距 | `Editor → Font`（建议 15~16） |
| 主题（深色/跟随系统） | `Appearance & Behavior → Appearance` |
| 查询超过 20 秒提醒 | `Appearance & Behavior → Notifications` → `Database queries that took much time` |
| 改快捷键 | `Keymap`，搜 `Query Console`、`Execute` |

---

## 第 12 章 常见报错与解决

| 报错关键字 | 原因 | 解决 |
|---|---|---|
| `The server time zone value '…' is unrecognized` | MySQL 时区与 JDBC 不一致 | 数据源 → **Advanced** → 加 `serverTimezone` = `Asia/Shanghai` |
| `Public Key Retrieval is not allowed` | MySQL 8 的 `caching_sha2_password` | Advanced → 加 `allowPublicKeyRetrieval` = `true` |
| `Access denied for user 'root'@'localhost'` | 密码错 / root 只允许本机 | 命令行 `mysql -u root -p` 验证 |
| `Communications link failure` | MySQL 服务没启动 | `services.msc` 启动 `MySQL80`，或 `net start mysql80` |
| `Unknown database 'xxx'` | 库名错 / 没建库 | 先 `CREATE DATABASE`，或在 schema 下拉里选对 |
| 表名红色波浪线 | 没选 schema，或解析模式不对 | 右上角 schema 下拉选库；参见第 4.4 节 |
| 驱动下载失败 | 网络/代理 | `Settings → Database → Drivers` 手动指定 mysql-connector-j 的 jar |
| 中文变问号 | 字符集 | Advanced 加 `characterEncoding` = `utf8` |
| 结果只有几百行 | 分页/页大小限制 | 改 `Limit page size to`，或导出全量 |
| 找不到查询控制台 | 它不在项目里 | `Alt + 4` → Scratches and Consoles → Database Consoles |

---

## 第 13 章 30 分钟上手练习（用 atguigudb）

```sql
-- 1. 确认连的是对的库（点中数据源按 F4 打开控制台）
SELECT * FROM employees;

-- 2. 条件 + 排序
SELECT last_name, salary FROM employees
WHERE salary > 8000 ORDER BY salary DESC;

-- 3. 分组聚合（讲义第 8 章）
SELECT department_id, COUNT(*) AS 人数, AVG(salary) AS 平均工资
FROM employees GROUP BY department_id HAVING COUNT(*) > 2;

-- 4. 多表连接（讲义第 6 章）
SELECT e.last_name, d.department_name
FROM employees e JOIN departments d ON e.department_id = d.department_id;

-- 5. 子查询（讲义第 9 章）
SELECT last_name, salary FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

配 5 个动作：
1. 结果区 **Export Data → Excel**，导一份到桌面；
2. 双击 `employees`，改一个 salary → **Preview Pending Changes** 看一眼 → **Submit** → `Ctrl + F5` 刷新；
3. 光标放 `employees` 上按 **`Ctrl + Q`** 看建表语句；
4. 按 **`Ctrl + Alt + E`** 翻刚才的执行历史；
5. 按 **`Ctrl + Alt + L`** 让 SQL 自动排版。

---

## 第 14 章 官方文档索引（本手册的校对来源）

| 主题 | 链接 |
|---|---|
| 运行查询（执行/取消/Execute to File/历史记录） | https://www.jetbrains.com/help/datagrip/run-a-query.html |
| 查询控制台（F4、Ctrl+Shift+Q、控制台存哪） | https://www.jetbrains.com/help/datagrip/query-consoles.html |
| 数据编辑器（Submit、事务模式、工具栏） | https://www.jetbrains.com/help/datagrip/data-editor-and-viewer.html |
| 导出（DDL、数据、mysqldump） | https://www.jetbrains.com/help/datagrip/export-data.html |
| 导入 | https://www.jetbrains.com/help/datagrip/import-data.html |
| ER 图 | https://www.jetbrains.com/help/datagrip/creating-diagrams.html |
| 连接数据源 | https://www.jetbrains.com/help/datagrip/managing-data-sources.html |
| 快捷键 | https://www.jetbrains.com/help/datagrip/mastering-keyboard-shortcuts.html |
| 2026.3 新功能 | https://www.jetbrains.com/datagrip/whatsnew/ |

### 本版（v2）相对上一版修正了什么

1. **新建控制台有快捷键**：`Ctrl + Shift + Q`、`F4`（v1 写"没有单一快捷键"，错）；
2. 结果面板正式名是 **Database Output**，里面才是 **Result / Output** 标签（v1 写"Results 标签"）；
3. ER 图菜单是 **Diagrams → Show Diagram**（v1 的 Show Visualization 是旧名）；
4. 事务不是"Auto-commit 勾选框"，而是 **Tx 的 Auto / Manual 模式** + Submit and Commit；
5. 导出/导入在 **Import/Export** 子菜单下（v1 直接写 Export Data to File）；
6. 分页设置路径是 `Database → Data Editor and Viewer → Limit page size to`（v1 写 Data Views）；
7. 新增：`Ctrl+F2` 取消、`Ctrl+F5` 刷新、`Ctrl+Q` 看结构、`Ctrl+Alt+E` 历史、`Ctrl+Alt+G` SQL Generator、`Ctrl+Alt+Z` 撤销；
8. 新增：20 MB 文件限制、Resolve mode、In-Editor Results、Export Table to Clipboard。

> 本文件：`C:\Users\24229\Desktop\DataGrip_使用手册.md`；上一版备份在工作区 `D:\deepseek_work\DataGrip_使用手册_v1_备份.md`。
> 建议用桌面 **MarkText** 打开（表格渲染最好）。
