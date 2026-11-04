# 04 · MySQL（数字当头的怪风格）

> 它和 Java/Python 完全不是一个路子：**开头永远是 `ERROR 数字`**。
> 那个数字比整句英文都值钱——**搜 `mysql 1064` 比搜一整句快十倍，而且一定能搜到。**

---

## 一、先认脸：MySQL 报错的三种长相

### 长相 1：客户端/命令行（你自己敲 SQL 时）

```
ERROR 1064 (42000) at line 1: You have an error in your SQL syntax;
check the manual that corresponds to your MySQL server version for the right syntax
to use near 'form user where id = 1' at line 1
```

### 长相 2：连不上服务器（还没到语法那一步）

```
ERROR 2003 (HY000): Can't connect to MySQL server on 'localhost' (10061)
```

### 长相 3：程序里抛出来的（Java/Python 调数据库）

```
java.sql.SQLException: Access denied for user 'root'@'localhost' (using password: YES)
```

**它们其实是同一件事的三种包皮**，里面都藏着一个数字码或一句关键短语。

---

## 二、四段式拆解（拿到报错先这么划）

```
ERROR 1064 (42000) at line 1: You have an error in your SQL syntax ... near 'form user where id = 1' at line 1
└─┬─┘ └─┬─┘ └─┬──┘  └─┬──┘   └────────────────┬──────────────────────────┘      └──────┬─────────┘
 类型  错误码 SQLSTATE 位置        固定套话（可以完全不看）                        它看不懂的地方
```

- **错误码**（1064）：唯一值得记的东西
- **SQLSTATE**（42000）：标准 SQL 的错误分类，**对你没用，忽略**
- **`check the manual that corresponds to your MySQL server version`**：MySQL 的固定开场白，**每个语法错误都有，别读它**
- **`near '...'`**：**这是金矿**——它看不懂的位置

> **`near` 的用法**：不要读 `near` 后面的整段，**看它前面紧挨着的一个词**。
> 上例 `near 'form user ...'`，问题就是 `form` 写错了，应该是 `from`。

---

## 三、按错误码分类（这份表可以直接当"病名手册"）

### A. 连接类：根本连不上（还没轮到 SQL）

| 报错 | 一句话 | 常见原因 |
|---|---|---|
| `2003 Can't connect to MySQL server on 'localhost' (10061)` | 这个门没人开 | 服务没启动 / 端口不是 3306 / 防火墙 |
| `2002 Can't connect to local MySQL server through socket` | 同上，Linux 走 socket | 服务没起，或 `my.cnf` 里 socket 路径不对 |
| `1045 Access denied for user 'root'@'localhost' (using password: YES)` | 用户名或密码错 | 密码错 / 该用户没这个 host 的权限 |
| `1045 ... (using password: NO)` | 你没给密码 | 连接串里漏了 password |
| `1130 Host '192.168.1.5' is not allowed to connect` | 这个 IP 不许连 | 用户只授权了 localhost，需要 `%` |
| `1040 Too many connections` | 连接数满了 | 连接池没关，或 `max_connections` 太小 |
| `2006 MySQL server has gone away` / `2013 Lost connection` | 连接中途断了 | 超时、包太大、服务重启 |

**排查顺序（背下来）**：
1. 服务起没起？（Windows 服务里找 MySQL；Linux `systemctl status mysql`）
2. 端口对不对？（`netstat -ano | findstr 3306`）
3. 账号密码对不对？（先用命令行 `mysql -u root -p` 试，能进就是程序配置问题）
4. 是不是只允许 localhost？（`localhost` 和 `127.0.0.1` 在 MySQL 里**是两个不同的 host**）

### B. 语法类

| 报错 | 一句话 |
|---|---|
| `1064 You have an error in your SQL syntax` | 这句话我看不懂（**最高频**） |
| `1065 Query was empty` | 你发了个空语句 |
| `1055 Expression #1 of SELECT list is not in GROUP BY clause` | `select` 的字段不在 `group by` 里 |
| `1140 In aggregated query without GROUP BY` | 用了聚合函数但没有 `group by` |

**1064 的六大真凶**（按出现频率排）：
1. 关键字拼错：`form` / `selcet` / `wher`
2. **中文引号**：`“abc”` `‘abc’` —— SQL 只认 `'abc'`
3. 少了逗号，或多了一个逗号（`(1,2,)`）
4. 表名/字段名用了保留字却没加反引号（`` `order` `` `` `group` `` `` `key` ``）
5. 引号没闭合（字符串跨行）
6. 语句顺序错：`where` 写在 `group by` 后面

### C. 对象不存在（名字写错 / 库选错）

| 报错 | 一句话 |
|---|---|
| `1146 Table 'shop.users' doesn't exist` | 这个表不存在（**注意前面带了库名 `shop.`**） |
| `1049 Unknown database 'shopp'` | 这个库不存在 |
| `1054 Unknown column 'user_name' in 'field list'` | 这个字段不存在（拼错，或表不对） |
| `1305 PROCEDURE xxx does not exist` | 存储过程不存在 |

**1146 的经典陷阱**：你以为在 A 库，其实连的是 B 库。看报错里的 `库名.表名`，那个库名就是**它实际在找的地方**。

### D. 约束类（数据本身不合法）

| 报错 | 一句话 | 怎么办 |
|---|---|---|
| `1062 Duplicate entry '1' for key 'PRIMARY'` | 主键/唯一键重复 | 换 id，或 `INSERT ... ON DUPLICATE KEY UPDATE` |
| `1048 Column 'name' cannot be null` | 这个字段不许空 | 给值，或改表允许 NULL |
| `1364 Field 'name' doesn't have a default value` | 没给值，也没默认值 | 插入时补上这个字段 |
| `1452 Cannot add or update a child row: a foreign key constraint fails` | 外键指向的那条记录不存在 | 先插主表，再插从表 |
| `1215 Cannot add foreign key constraint` | 建外键失败 | 两边类型/字符集/引擎（InnoDB）要一致 |
| `1406 Data too long for column 'title'` | 内容超长 | 改字段长度，或截断 |
| `1366 Incorrect string value: '\xE4\xB8\xAD' for column` | **字符集不对**（存中文） | 表/库/连接都改成 `utf8mb4` |
| `1264 Out of range value for column 'age'` | 数值超范围 | 检查类型（tinyint 只到 127） |
| `1292 Incorrect datetime value: '2024-13-01'` | 日期不合法 | 月份 13 不存在 |

### E. 权限与安全模式

| 报错 | 一句话 |
|---|---|
| `1142 SELECT command denied to user 'x'@'localhost' for table 'users'` | 这个账号没这张表的权限 |
| `1227 Access denied; you need (at least one of) the SUPER privilege(s)` | 需要更高权限 |
| `1175 You are using safe update mode` | 安全模式不让你无 `where` 地改数据 |

### F. 锁与超时（并发时才会遇到）

| 报错 | 一句话 |
|---|---|
| `1205 Lock wait timeout exceeded` | 等锁等超时了（有事务没提交） |
| `1213 Deadlock found when trying to get lock` | 死锁，MySQL 主动干掉了一个事务 |

---

## 四、程序里调数据库时，报错被"包"了一层

Java 里你看到的往往是这样：

```
java.sql.SQLException: Access denied for user 'root'@'localhost' (using password: YES)
	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:...)
```

**别被 `at ...` 吓到**，那是驱动内部的堆栈。**真正的内容是第一行 `SQLException:` 后面那半句**，和你在命令行里看到的一模一样。

Python（pymysql）里：

```
pymysql.err.OperationalError: (1045, "Access denied for user 'root'@'localhost' (using password: YES)")
```

**括号里第一个数字就是 MySQL 错误码**，后面是原文。

---

## 五、这一族的高频词

| 词 | 意思 |
|---|---|
| syntax | 语法 |
| near | 在……附近（它看不懂的位置） |
| access denied | 拒绝访问 |
| using password | 使用了密码（YES/NO） |
| unknown database / column | 未知的库 / 字段 |
| doesn't exist | 不存在 |
| duplicate entry | 重复条目 |
| constraint | 约束 |
| foreign key | 外键 |
| cannot be null | 不能为 null |
| doesn't have a default value | 没有默认值 |
| data too long | 数据太长 |
| incorrect string value | 字符串值不正确（编码问题） |
| out of range | 超出范围 |
| denied to user | 对该用户拒绝 |
| safe update mode | 安全更新模式 |
| lock wait timeout | 锁等待超时 |
| deadlock | 死锁 |
| gone away / lost connection | 连接断了 |
| Too many connections | 连接数过多 |

---

## 六、MySQL 排查顺序（贴墙上）

1. **先看数字码**，`mysql + 码` 直接搜
2. **1064** → 看 `near` 前面那个词 + 查中文引号
3. **1146/1054** → 看报错里的**库名.表名.字段名**，跟你以为的是不是一回事
4. **1045/2003** → 先命令行连一次，能连就是程序配置问题
5. **1366** → 一律怀疑字符集，`utf8mb4` 三处都要改
