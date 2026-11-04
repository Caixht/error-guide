# 06 · 网络与 HTTP（连不上、下不动、推不上去）

> 网络报错的核心只有**五个词**：`refused`、`timeout`、`reset`、`not found`、`certificate`。
> 认出这五个，你就能判断到底是"没人开门"、"路不通"、"半路被掐"、"地址错"、还是"证书不对"。

---

## 一、先建立一个四层模型

`域名 → IP → 端口 → 服务`

- **域名解析**（DNS）：`www.abc.com` 到底是谁 → 失败会说 `UnknownHost` / `Could not resolve host`
- **IP 可达**：这台机器在不在线 → 失败会说 `timeout` / `No route to host`
- **端口开放**：这个门开没开 → 失败会说 `Connection refused`
- **服务正常**：门开了，里面的人听不听得懂 → 失败会说 `404` / `500`

**排错永远从第一层往下走**，别一上来就怀疑代码。

---

## 二、连接类报错（最常见的五句）

### `java.net.UnknownHostException: api.example.com`

**一句话**：这个域名解析不出 IP（DNS 问题 / 域名拼错 / 没网）。
**怎么办**：`ping api.example.com`，`nslookup api.example.com`。

### `fatal: unable to access '...': Could not resolve host: github.com`（Git）

**一句话**：同上，DNS 挂了（校园网/代理最常见）。
**怎么办**：换 DNS（`8.8.8.8`）、关掉代理、或走镜像。

### `Connection refused`

**一句话**：**IP 通了，但这个端口没人监听**。
**怎么办**：`ss -lntp | grep 8080`（Linux）/ `netstat -ano | findstr 8080`（Windows）；服务没起或端口不对。

### `Connection timed out` / `Read timed out`

**一句话**：包发出去了，没人回，等到超时。
**怎么办**：防火墙、安全组、VPN、代理；`telnet ip port` 试。

### `Connection reset by peer`

**一句话**：对面直接把连接掐了（服务崩了、被限流、协议不对）。
**怎么办**：看服务端日志；确认是不是 `http` 连了 `https` 端口。

### `javax.net.ssl.SSLHandshakeException: PKIX path building failed`

**一句话**：证书验不过（自签名证书、公司代理、JDK 缺根证书）。
**怎么办**：把证书导入 JDK 的 `cacerts`；临时方案是信任所有证书（**只在自己电脑上这么干**）。

---

## 三、HTTP 状态码（服务端说"我听懂了，但我这样回答你"）

| 码 | 名字 | 一句话 | 该怪谁 |
|---|---|---|---|
| 200 | OK | 成功 | — |
| 301/302 | Moved / Found | 搬家了，看 `Location` 头 | 你请求的地址旧了 |
| 400 | Bad Request | 你的请求格式不对 | 你 |
| 401 | Unauthorized | 没登录 / token 无效 | 你 |
| 403 | Forbidden | 登录了，但没权限 | 权限配置 |
| 404 | Not Found | 这个地址没有东西 | 路径写错 / 路由没配 |
| 405 | Method Not Allowed | 方法不对（用 GET 打了 POST 接口） | 你 |
| 408 | Request Timeout | 你发得太慢 | 网络 |
| 413 | Payload Too Large | 上传的东西太大 | 你 / 服务端限制 |
| 429 | Too Many Requests | 请求太频繁，被限流 | 你 |
| 500 | Internal Server Error | 服务端自己炸了 | 服务端 |
| 502 | Bad Gateway | 网关后面那个服务挂了 | 后端 |
| 503 | Service Unavailable | 服务不可用（过载/维护） | 服务端 |
| 504 | Gateway Timeout | 网关等后端等超时了 | 后端太慢 |

**记住三句话**：
- **4xx 是你的问题**（请求错、没权限、地址错、太频繁）
- **5xx 是服务端的问题**（但如果是你自己写的后端，那就是你的问题）
- **502/504 先把眼睛移到后端服务的日志上**，Nginx 那层看不出来

---

## 四、下载依赖失败（pip / npm / maven）

### `pip: Read timed out.` / `WARNING: Retrying (Retry(total=4...))`

**一句话**：连 PyPI 超时（国内网络常见）。
**怎么办**：换源
```
pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### `ERROR: Could not install packages due to an OSError: [SSL: CERTIFICATE_VERIFY_FAILED]`

**一句话**：证书校验失败（公司代理 / Python 没装证书）。
**怎么办**：升级 pip；或指定 `--trusted-host`（仅本地）。

### `npm ERR! code ENOTFOUND` / `ETIMEDOUT`

**一句话**：npm 连不上 registry。
**怎么办**：`npm config set registry https://registry.npmmirror.com`。

### `maven: Could not transfer artifact ... Connection timed out`

**一句话**：Maven 中央仓库连不上。
**怎么办**：`settings.xml` 换阿里云镜像。

---

## 五、Git 的几句"经典台词"

| 报错 | 一句话 | 怎么办 |
|---|---|---|
| `fatal: not a git repository` | 这个目录不是仓库 | `cd` 到项目根目录，或 `git init` |
| `Permission denied (publickey)` | SSH 密钥没配好 | 加公钥到 GitHub；测 `ssh -T git@github.com` |
| `rejected (fetch first)` / `non-fast-forward` | 远程有你没有的提交 | 先 `git pull`，再 `push` |
| `CONFLICT (content): Merge conflict in xxx` | 两边改了同一处 | 打开文件找 `<<<<<<<` 手动合并 |
| `refusing to merge unrelated histories` | 两个仓库没有共同祖先 | `git pull --allow-unrelated-histories` |
| `Your branch is ahead of 'origin/main' by 2 commits` | 本地有 2 个提交没推（**这不是报错**） | `git push` |
| `detached HEAD` | 你不在任何分支上 | `git switch -c new-branch` 或 `git switch main` |
| `fatal: destination path 'x' already exists` | 同名文件夹已存在 | 换名字或先删 |

---

## 六、四个命令解决 80% 的网络问题

```
ping 8.8.8.8              # 有没有网
nslookup github.com       # 域名能不能解析
telnet github.com 443     # 这个端口通不通
curl -v https://api.xxx   # 把整个过程打出来（最有用）
```

> Windows 没有 `telnet` 的话：`Test-NetConnection github.com -Port 443`。

---

## 七、这一族的高频词

| 词 | 意思 |
|---|---|
| connection | 连接 |
| refused | 被拒绝（没人监听） |
| timed out / timeout | 超时 |
| reset | 被重置（强制断开） |
| resolve / resolution | 解析（域名→IP） |
| host | 主机 |
| peer | 对端 |
| gateway | 网关 |
| certificate | 证书 |
| handshake | 握手 |
| unauthorized | 未授权（没登录） |
| forbidden | 禁止（没权限） |
| not found | 未找到 |
| bad gateway | 网关错误 |
| unavailable | 不可用 |
| retry / retrying | 重试 |
| request / response | 请求 / 响应 |
| header | 请求头/响应头 |
| payload | 载荷（传过去的数据） |

---

## 八、一句话总结

**看到网络报错，先问四个问题**：
1. 域名解析出来了吗？（`UnknownHost` → DNS）
2. IP 通吗？（`timeout` → 路由/防火墙）
3. 端口开了吗？（`refused` → 服务没起）
4. 对方听懂了但拒绝了？（`4xx/5xx` → 看日志）
