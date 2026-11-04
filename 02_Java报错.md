# 02 · Java 报错（语言本身）

> 特点：**一长串英文，但其中只有 3 行有用**。
> 铁律：有 `Caused by` 就**从最下面那个 `Caused by` 往上读**，那才是根因。

---

## 一、先学会拆一条完整堆栈

```
Exception in thread "main" java.lang.RuntimeException: 加载用户失败
	at com.demo.service.UserService.load(UserService.java:42)
	at com.demo.Main.main(Main.java:15)
Caused by: java.lang.NullPointerException: Cannot invoke "com.demo.User.getName()" because "user" is null
	at com.demo.service.UserService.load(UserService.java:38)
	... 2 more
```

逐行翻译：

| 行 | 它在说什么 |
|---|---|
| `Exception in thread "main"` | 出事的线程是 main（主线程） |
| `java.lang.RuntimeException: 加载用户失败` | 最外层：包装了一下，说"加载用户失败" |
| `at ...UserService.load(UserService.java:42)` | 调用链：Main 第 15 行 → UserService 第 42 行 |
| `Caused by: java.lang.NullPointerException` | **真正的病**：空指针 |
| `Cannot invoke "getName()" because "user" is null` | **主诉**：想调 getName()，但 user 是 null |
| `at ...UserService.load(UserService.java:38)` | **病灶**：第 38 行 |

**结论**：去 `UserService.java` 第 38 行，看 `user` 为什么是 null（没查到？参数没传？查询返回空？）。

> `... 2 more` 意思是"后面还有 2 层跟上面重复，省略了"，**不是错误**。

---

## 二、编译期错误（还没运行就红）

### `cannot find symbol`

```
Main.java:5: error: cannot find symbol
        Stirng name = "x";
        ^
  symbol:   class Stirng
  location: class Main
```

**一句话**：编译器说"我不认识 `Stirng` 这个名字"。
**先看 `symbol:` 那一行**——它直接告诉你它不认识谁。
**三大原因**：拼错 / 没 import / 依赖没加。

### `incompatible types: possible lossy conversion from double to int`

**一句话**：类型不匹配，大范围往小范围塞会丢精度。
**怎么办**：显式转换 `(int) d`，或改类型。

### `incompatible types: String cannot be converted to int`

**一句话**：字符串不能当整数用。
**怎么办**：`Integer.parseInt(s)`。

### `missing return statement`

**一句话**：方法声明了要返回值，但有分支没 return。
**怎么办**：补 return，或改成 `void`。

### `variable name might not have been initialized`

**一句话**：局部变量没赋初值就用了。
**怎么办**：`String name = "";` 先给个默认值。

### `unreachable statement`

**一句话**：这行代码永远执行不到（前面已经 return/throw 了）。
**怎么办**：删掉，或调整逻辑。

### `';' expected` / `')' expected` / `reached end of file while parsing`

**一句话**：**标点没配平**——少分号、少括号、少大括号。
**怎么办**：从 `reached end of file` 那句往上找，通常是最近改过的那段；用格式化快捷键（Ctrl+Alt+L）能快速看出缩进错位。

---

## 三、运行期错误（跑起来才炸，最常见）

### `NullPointerException`（NPE，空指针）

```
java.lang.NullPointerException: Cannot invoke "String.length()" because "s" is null
```

**一句话**：你拿一个"空"的东西去用了（调方法、取属性、算长度）。
**为什么**：对象没 new、查询返回 null、`get()` 拿不到、`.equals()` 左边是 null。
**怎么办**：在报错那一行前面打印/断点看这个变量；用 `Objects.requireNonNull()` 或先判空。

### `ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 3`

**一句话**：数组只有 3 个位置（0/1/2），你要第 5 个。
**怎么办**：注意**下标从 0 开始**，循环条件写 `i < arr.length` 而不是 `<=`。

### `StringIndexOutOfBoundsException: String index out of range: 10`

**一句话**：同上，但是对字符串 `charAt` / `substring`。
**怎么办**：检查 `substring(begin, end)` 的 end 是不是超了长度。

### `NumberFormatException: For input string: "abc"`

**一句话**：想把 `"abc"` 变成数字，变不了。
**怎么办**：`Integer.parseInt` 前判空和格式；用户输入要 try-catch。

### `ClassCastException: class A cannot be cast to class B`

**一句话**：把 A 硬说成 B，说不过去。
**怎么办**：`instanceof` 判断后再转。

### `ArithmeticException: / by zero`

**一句话**：除以 0 了。
**怎么办**：判除数是否为 0。

### `IllegalArgumentException` / `IllegalStateException`

**一句话**：方法说"你给我的参数不合法" / "现在这个状态不能干这事"。
**怎么办**：**看冒号后面的消息**，通常写得很清楚（比如 `port out of range: 70000`）。

### `UnsupportedOperationException`

**一句话**：这个操作不支持（常见于 `Arrays.asList()` 后 add、不可变集合）。
**怎么办**：换成 `new ArrayList<>(Arrays.asList(...))`。

### `ConcurrentModificationException`

**一句话**：边遍历边删集合。
**怎么办**：用 `Iterator.remove()`，或 `removeIf()`。

### `ClassNotFoundException` / `NoClassDefFoundError`

**一句话**：运行时找不到某个类——**缺 jar 包**，不是代码逻辑。
**区别**：`NotFound` 是"加载时就没找到"，`NoClassDefFound` 是"编译时在、运行时没"（依赖 scope 写错、静态初始化失败）。
**怎么办**：检查依赖、`provided` 改成默认、重新 `mvn clean package`。

### `StackOverflowError`

**一句话**：递归太深 / 自己调自己没出口。
**怎么办**：找递归出口，或改循环。

### `OutOfMemoryError: Java heap space`

**一句话**：堆内存不够（一次读太大文件、死循环里不断 new）。
**怎么办**：`-Xmx2g` 调大；更重要的是找内存泄漏点。

---

## 四、这一族的高频词

| 词 | 意思 |
|---|---|
| exception / error | 异常 / 错误 |
| thrown / throw | 被抛出 |
| caused by | 由……引起（根因） |
| at | 位于（后面是类.方法(文件:行)） |
| symbol | 符号（类名、方法名、变量名） |
| location | 位置 |
| incompatible | 不兼容 |
| out of bounds | 越界 |
| out of range | 超出范围 |
| initialized | 已初始化 |
| unreachable | 不可达 |
| index / length | 下标 / 长度 |
| cast | 类型转换 |
| heap | 堆（内存） |
| stack | 栈 |

---

## 五、定位套路（背下来）

1. **看 `Caused by`**，没有就看第一条 `at`
2. **看行号**，去那一行找**变量**
3. **问自己**：这个变量在这一刻，可能是 null / 0 / 空集合 吗？
4. **在那一行前面打印它**（`System.out.println(x)`），跑一次，答案立刻出来
