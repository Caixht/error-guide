# 03 · Python 报错（Traceback）

> 特点：**结构最规整**。永远最后一行才是错误，上面全是"我是怎么走到这里的"。
> 铁律：**从上往下找到第一行 `File "xxx.py", line N`，再去最后一行的错误类型。**

---

## 一、拆一条完整 Traceback

```
Traceback (most recent call last):
  File "main.py", line 12, in <module>
    data = load(paths[3])
  File "main.py", line 5, in load
    return json.load(open(p))
FileNotFoundError: [Errno 2] No such file or directory: 'data/b.csv'
```

| 行 | 它在说什么 |
|---|---|
| `Traceback (most recent call last)` | 下面是调用栈，**越往下越靠近出事点** |
| `File "main.py", line 12, in <module>` | 在 main.py 第 12 行、模块层（最外层）调的 |
| `data = load(paths[3])` | 它把出事的**那行代码原文**也打出来了 |
| `File "main.py", line 5, in load` | 进入 load 函数，第 5 行 |
| `FileNotFoundError: [Errno 2] No such file or directory: 'data/b.csv'` | **病名 + 主诉**：文件不存在，路径是 `data/b.csv` |

**它其实已经告诉你答案**：去找 `data/b.csv`，它不在。可能是路径写错、工作目录不对、文件真没生成。

> `[Errno 2]` 是系统错误号，可以忽略；**冒号后面那半句才有用**。

---

## 二、缩进与语法（编辑时就能发现）

### `IndentationError: expected an indented block after 'if' statement on line 4`

**一句话**：`if` 后面该缩进，你没缩。
**怎么办**：Python 靠缩进划分代码块，**统一用 4 个空格**，不要混 Tab。

### `IndentationError: unindent does not match any outer indentation level`

**一句话**：缩进对不齐（常见于 Tab 和空格混用）。
**怎么办**：全选 → 用编辑器的 "Convert Indents to Spaces"。

### `SyntaxError: invalid syntax`

**一句话**：这句话不合语法（少冒号、少括号、字符串没闭合、用了中文标点）。
**怎么办**：看 `^` 指的位置，**问题通常在它前面一点点**；重点查中文全角符号：`（` `）` `，` `：` `"`。

### `SyntaxError: unexpected EOF while parsing` / `'(' was never closed`

**一句话**：文件/括号没闭合——**少了一个 `)`、`]`、`}` 或引号**。
**怎么办**：从下往上数括号；用编辑器配对的括号高亮。

---

## 三、名字与类型（最高频）

### `NameError: name 'usr_name' is not defined`

**一句话**：用了从没定义过的名字。
**怎么办**：拼写检查（`usr_name` vs `user_name`）、变量作用域、忘 import。

### `TypeError: unsupported operand type(s) for +: 'int' and 'str'`

**一句话**：`1 + "2"` 不合法，数字和字符串不能直接加。
**怎么办**：`str(n) + s` 或 `int(s) + n`，先统一类型。

### `TypeError: can only concatenate str (not "int") to str`

**一句话**：同上，反过来说一遍。
**怎么办**：`f"{s}{n}"` 最省事。

### `TypeError: 'NoneType' object is not subscriptable` / `'NoneType' object has no attribute 'xxx'`

**一句话**：**这个变量是 None**，你却拿它当下标/对象用。
**为什么**：函数忘了 return、查询没查到、`dict.get()` 返回 None。
**怎么办**：在那一行前 `print(type(x), x)`，确认它到底是不是 None。

### `TypeError: 'int' object is not callable` / `'str' object is not callable`

**一句话**：你把变量名和函数名撞了（比如 `list = [1,2]` 之后又 `list()`）。
**怎么办**：改名，别用内置名（`list` `dict` `str` `sum` `id`）。

### `TypeError: f() takes 1 positional argument but 2 were given`

**一句话**：函数只收 1 个参数，你给了 2 个（常见：忘了写 `self`，或方法调用方式不对）。
**怎么办**：看函数定义；类方法第一个参数必须是 `self`。

### `TypeError: f() missing 1 required positional argument: 'b'`

**一句话**：少传了一个必填参数。
**怎么办**：对照函数定义补参数，或给默认值。

### `ValueError: invalid literal for int() with base 10: '12a'`

**一句话**：想把 `"12a"` 转成整数，转不了。
**怎么办**：先清洗字符串，或 try/except 兜住。

### `ValueError: too many values to unpack (expected 2)`

**一句话**：`a, b = ...` 但右边给了 3 个值。
**怎么办**：改成 `a, b, *rest = ...` 或检查数据。

---

## 四、容器类（下标、键）

### `IndexError: list index out of range`

**一句话**：列表没这么长（空列表取 `[0]` 最常见）。
**怎么办**：先判空 `if lst:`，或用 `for x in lst` 而不是 `for i in range(...)`。

### `KeyError: 'user_id'`

**一句话**：字典里没有这个键。
**怎么办**：用 `d.get("user_id")`（没有返回 None）或 `d.get(k, 默认值)`；打印 `d.keys()` 看真实键名。

### `AttributeError: 'DataFrame' object has no attribute 'applay'`

**一句话**：这个对象没有这个属性/方法。
**怎么办**：多半是**拼错方法名**（`apply`）；也可能是类型不对（你以为它是 DataFrame，其实是 list）。

---

## 五、模块与环境（最像"玄学"，其实最机械）

### `ModuleNotFoundError: No module named 'requests'`

**一句话**：当前这个 Python 里没装这个包。
**注意**：**90% 是装到了另一个 Python 里**（系统 Python vs 虚拟环境）。
**怎么办**：
1. 看 PyCharm 右下角解释器是哪个；
2. 用它对应的 pip 装：`<解释器路径> -m pip install requests`；
3. 确认 `pip -V` 显示的路径和解释器一致。

### `ImportError: cannot import name 'xxx' from 'yyy'`

**一句话**：包在，但里面没有这个名字（版本太旧/改名了/循环导入）。
**怎么办**：`pip show yyy` 看版本，查官方文档确认名字；循环导入就把 import 挪到函数里。

### `FileNotFoundError: [Errno 2] No such file or directory: 'data/b.csv'`

**一句话**：路径不对或文件不在。
**怎么办**：`print(os.getcwd())` 看当前工作目录；用绝对路径或 `os.path.join(os.path.dirname(__file__), ...)`。

### `PermissionError: [Errno 13] Permission denied: 'C:\a.txt'`

**一句话**：没权限（文件被 Excel 占用、写到系统目录、只读）。
**怎么办**：关掉占用它的程序；换个路径；Windows 上避免写 `C:\Program Files`。

### `UnicodeDecodeError: 'gbk' codec can't decode byte 0xa1`

**一句话**：**编码不对**——文件是 UTF-8，你用 GBK 读（Windows 默认 GBK）。
**怎么办**：`open(path, encoding="utf-8")`；读 CSV 用 `encoding="utf-8-sig"`。

### `json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)`

**一句话**：你给的东西不是 JSON（HTML 错误页、空字符串、BOM 头）。
**怎么办**：先 `print(text[:200])` 看看你收到的到底是什么。

### `RecursionError: maximum recursion depth exceeded`

**一句话**：递归太深，没出口。
**怎么办**：加终止条件，或改迭代。

---

## 六、这一族的高频词

| 词 | 意思 |
|---|---|
| traceback | 回溯（调用栈） |
| most recent call last | 最近一次调用在最后（所以看底部） |
| File ... line ... | 文件……第……行 |
| module / package | 模块 / 包 |
| attribute | 属性 |
| subscriptable | 可下标访问的（`x[0]`） |
| callable | 可调用的 |
| operand | 操作数（`+` 两边的东西） |
| positional argument | 位置参数 |
| keyword argument | 关键字参数 |
| unpack | 解包（`a, b = ...`） |
| codec | 编解码器 |
| codec can't decode | 解不出来（编码不对） |
| Errno | 系统错误号 |
| expected / unexpected | 期望的 / 意外的 |

---

## 七、Python 三步定位法

1. **看最后一行**：病名 + 冒号后的主诉
2. **找倒数第一个 `File "你的文件.py"`**：那才是你自己的代码（中间很多是库的代码，跳过）
3. **在那一行上面加一句 `print(变量)`**：Python 调试 90% 靠打印，不丢人
