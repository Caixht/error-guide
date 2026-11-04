# 05 · 虚拟机与 Linux

> 这一族有个共同点：**报错经常是"操作系统在跟你说话"，而不是程序**。
> 好消息是：它们的句式极其固定，翻来覆去就那十来句。

---

# 第一部分：VMware / VirtualBox

## 1. `此主机支持 Intel VT-x，但 Intel VT-x 处于禁用状态`

**英文原版**：`This host supports Intel VT-x, but Intel VT-x is disabled.`

**一句话**：CPU 的虚拟化开关没开。
**怎么办**：重启进 BIOS → 找 `Intel Virtualization Technology` / `VT-x` / `SVM Mode`（AMD）→ 设为 Enabled。

## 2. `VMware Workstation 与 Hyper-V 不兼容`

**英文原版**：`VMware Workstation and Hyper-V are not compatible.`

**一句话**：Windows 自带的 Hyper-V / WSL2 / 内存完整性把虚拟化抢走了。
**怎么办**：关掉 Hyper-V 和"内核隔离-内存完整性"，重启；或者升级到较新的 VMware 版本（新版可以共存）。

## 3. `无法打开虚拟机: ... .vmdk 或 .vmem 被锁定` / `Cannot open the disk ... because it is locked`

**一句话**：上次没正常关机，残留了锁文件。
**怎么办**：关掉所有 VMware 进程 → 去虚拟机目录删掉 `.lck` 文件夹 → 重启虚拟机。

## 4. 开机黑屏 / 卡在 `Booting from Hard Disk...` / 光标一直闪

**常见三个原因**：
1. ISO 没挂上（`虚拟机设置 → CD/DVD → 使用 ISO 映像文件`）
2. 启动顺序不对（进 BIOS 把 CD 调到第一）
3. 显卡/3D 加速不兼容（关掉 `加速 3D 图形`）

## 5. 装完系统没网（`ping` 不通、上不了网页）

**先看虚拟网络模式**：

| 模式 | 效果 | 什么时候用 |
|---|---|---|
| NAT | 虚拟机能上网，宿主机访问它要端口转发 | 默认，最省事 |
| 桥接（Bridged） | 虚拟机像局域网里另一台电脑，有独立 IP | 要和别的机器互访 |
| 仅主机（Host-Only） | 只能和宿主机通信，不能上网 | 做内网实验 |

**常见坑**：桥接选错了物理网卡（笔记本有 WiFi 和有线两块）；VMware 的 `VMnet8` 被禁用。

## 6. 共享文件夹在 `/mnt/hgfs` 里看不到

**一句话**：没装 VMware Tools，或者共享没开。
**怎么办**：`虚拟机 → 安装 VMware Tools` → 挂载 → 解压 → `sudo ./vmware-install.pl`。

## 7. `There is not enough space on the file system for the selected operation`

**一句话**：宿主机磁盘满了，或虚拟磁盘设的太小。
**怎么办**：清理宿主机；或 `虚拟机设置 → 硬盘 → 扩展`（扩展后还要在系统里扩分区）。

---

# 第二部分：Linux 命令行

> 学习顺序建议：**先认这 12 句，能覆盖 80% 的日常**。

## 1. `bash: xxx: command not found`

**一句话**：系统里没有这个命令。
**三个原因**（按顺序查）：
1. 真没装 → `sudo apt install xxx` / `sudo yum install xxx`
2. 装了但不在 PATH 里 → `which xxx`、`echo $PATH`
3. 拼错了（`gti` / `pyhton`）

## 2. `bash: ./run.sh: /bin/bash^M: bad interpreter: No such file or directory`

**一句话**：**脚本是 Windows 换行（CRLF），Linux 不认那个 `\r`**。
**怎么办**：`sed -i 's/\r$//' run.sh` 或 `dos2unix run.sh`。
> 这个报错长得吓人，其实是"文件格式"问题，跟代码无关。**在 Windows 上写 shell 脚本必踩。**

## 3. `bash: ./run.sh: Permission denied`

**一句话**：这个文件没有执行权限。
**怎么办**：`chmod +x run.sh`，再 `./run.sh`。

## 4. `Permission denied`（访问文件/目录时）

**一句话**：当前用户没权限。
**怎么办**：`ls -l` 看权限位；`sudo` 提权（临时）；`chown` 改属主（永久）。
**注意**：Linux 里 **root 也会被 SELinux 拦**，那时候 `sudo` 没用。

## 5. `No such file or directory`（明明文件就在那）

**三种可能**：
1. 路径写错（大小写敏感！`Data.txt` ≠ `data.txt`）
2. 相对路径的基准不对（你在别的目录执行）
3. 是脚本解释器缺失（`bad interpreter`，见第 2 条）

## 6. `Read-only file system`

**一句话**：这个分区被挂成只读了（磁盘故障保护、或 mount 参数）。
**怎么办**：`mount | grep ro` 找到它；`sudo mount -o remount,rw /`。

## 7. `No space left on device`

**一句话**：磁盘满了（**inode 满了也是这句**）。
**怎么办**：`df -h` 看空间，`df -i` 看 inode；`du -sh /* | sort -h` 找大文件。

## 8. `Killed`（进程莫名其妙没了）

**一句话**：被 OOM Killer 杀了——内存不够。
**怎么办**：`dmesg | grep -i kill` 确认；加内存/加 swap/减小数据量。

## 9. `Connection refused` / `Connection timed out`（在 Linux 里也常见）

**一句话**：`refused` = 端口没人监听；`timed out` = 网络不通或被防火墙挡。
**怎么办**：`ss -lntp | grep 3306` 看有没有在听；`telnet ip port` 试连通。

## 10. `Unit xxx.service not found.`（systemctl）

**一句话**：没有这个服务（名字错 / 没装 / 是 init.d 脚本）。
**怎么办**：`systemctl list-unit-files | grep xxx`；名字通常带 `.service`。

## 11. `Permission denied (publickey).`（SSH 连服务器）

**一句话**：服务器不认你的密钥。
**怎么办**：确认 `~/.ssh/id_rsa` 存在、公钥已加到服务器 `authorized_keys`、权限是 `600`（**权限太开放 SSH 会拒绝**）。

## 12. `Host key verification failed.`

**一句话**：服务器换了密钥（重装过），你本地记录的还是旧的。
**怎么办**：`ssh-keygen -R 服务器IP` 删掉旧记录，再连。

---

## 三、这一族的高频词

| 词 | 意思 |
|---|---|
| permission denied | 权限被拒 |
| read-only | 只读 |
| no space left | 空间不足 |
| bad interpreter | 解释器坏了/找不到（换行符问题） |
| command not found | 命令不存在 |
| locked | 被锁定 |
| not compatible | 不兼容 |
| disabled | 已禁用 |
| enabled | 已启用 |
| mount / remount | 挂载 / 重新挂载 |
| device | 设备 |
| host / guest | 宿主机 / 虚拟机（客户机） |
| snapshot | 快照 |
| unit | 服务单元（systemd 的叫法） |
| publickey | 公钥 |
| killed | 被杀掉（通常是内存不足） |

---

## 四、虚拟机排查顺序

1. **能不能开机** → BIOS 虚拟化 / Hyper-V 冲突
2. **能不能上网** → NAT 还是桥接 / VMnet 是否启用
3. **能不能互通** → 防火墙 / IP 网段是否同一段
4. **文件传不进去** → 共享文件夹 / VMware Tools / scp
