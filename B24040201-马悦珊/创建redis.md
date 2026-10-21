# Linux 实验二　Redis 服务器搭建与 RedisInsight 可视化连接实验报告

| 项目 | 内容 |
| --- | --- |
| 姓名 | 马悦珊 |
| 学号 | B24040201 |
| 班级 | B240402 |
| 指导教师 | 王磊 |
| 实验日期 | 2026-10-21 |
| 实验环境 | Windows 主机 ＋ VMware Workstation ＋ Ubuntu 虚拟机 |
| 虚拟机普通用户 | mys |
| Redis 客户端 | RedisInsight 3.8.0 |
| Redis 连接地址 | 127.0.0.1:6379 |

> 说明：本报告中的操作均在本人 Ubuntu 虚拟机内完成。截图按照实际操作顺序插入，记录 RedisInsight 安装、版本验证、程序启动、连接测试以及数据库连接成功的过程。

---

## 一、实验目的

1. 掌握在 Ubuntu 中使用 APT 安装 Redis 服务器的方法。
2. 掌握 Redis 配置文件的修改、服务重启与状态检查方法。
3. 理解 Redis 的 `bind`、`requirepass` 等基础安全配置。
4. 学会下载并安装 RedisInsight 图形化管理工具。
5. 掌握使用 RedisInsight 连接本机 Redis 数据库的方法。
6. 能够通过图形化界面确认 Redis 服务器连接状态。

---

## 二、实验环境

| 项目 | 内容 |
| --- | --- |
| 主机操作系统 | Windows |
| 虚拟机软件 | VMware Workstation |
| 虚拟机操作系统 | Ubuntu |
| 虚拟机普通用户 | mys |
| Redis 服务端口 | 6379 |
| Redis 本机地址 | 127.0.0.1 |
| 图形化工具 | RedisInsight 3.8.0 |
| 安装包文件 | `Redis-Insight-linux-amd64.deb` |

> 说明：实验中使用密码 `123456` 仅用于课程实验环境。实际部署 Redis 时应使用更复杂的密码，并限制可访问 Redis 的网络范围。

---

## 三、实验内容与步骤

### （一）安装并启动 Redis 服务器

**实验内容：** 使用 Ubuntu 的 APT 软件包管理器安装 Redis 服务器，并设置 Redis 开机自动启动。

在 Ubuntu 终端中执行：

```bash
sudo apt update
sudo apt install -y redis-server
sudo systemctl enable --now redis-server
```

安装完成后查看服务状态：

```bash
sudo systemctl status redis-server
```

当输出中出现以下状态时，表示 Redis 服务已经正常运行：

```text
Active: active (running)
```

随后使用 Redis 命令行客户端测试服务：

```bash
redis-cli ping
```

正常情况下返回：

```text
PONG
```

**步骤小结：** `apt install redis-server` 完成 Redis 软件安装，`systemctl enable --now redis-server` 同时设置 Redis 开机自启并立即启动服务。

---

### （二）配置 Redis 访问地址与密码

**实验内容：** 修改 Redis 配置文件 `/etc/redis/redis.conf`，将监听地址设置为允许远程连接，并增加访问密码。

首先备份配置文件：

```bash
sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.bak
```

使用 `nano` 编辑配置文件：

```bash
sudo nano /etc/redis/redis.conf
```

找到 `bind` 配置项，将：

```conf
bind 127.0.0.1 -::1
```

修改为：

```conf
bind 0.0.0.0
```

再找到 `requirepass` 配置项，去掉注释并设置密码：

```conf
requirepass 123456
```

保存并退出 `nano`：

```text
Ctrl+O
Enter
Ctrl+X
```

也可以使用以下命令完成同样的修改：

```bash
sudo sed -i 's/^bind 127\.0\.0\.1 -::1$/bind 0.0.0.0/' /etc/redis/redis.conf
sudo sed -i 's/^# requirepass foobared$/requirepass 123456/' /etc/redis/redis.conf
```

检查配置是否修改成功：

```bash
sudo grep '^bind' /etc/redis/redis.conf
sudo grep '^requirepass' /etc/redis/redis.conf
```

预期显示：

```text
bind 0.0.0.0
requirepass 123456
```

重启 Redis 使配置生效：

```bash
sudo systemctl restart redis-server
```

使用密码验证 Redis：

```bash
redis-cli -a '123456' ping
```

预期返回：

```text
PONG
```

> 注意：`bind 0.0.0.0` 表示 Redis 可以监听所有 IPv4 网络接口。如果只在当前虚拟机内部使用 Redis，建议恢复为 `bind 127.0.0.1 -::1`，以减少网络暴露风险。

---

### （三）下载并安装 RedisInsight

**实验内容：** 从 Redis 官方网站下载适用于 Ubuntu 的 RedisInsight 安装包，并使用 APT 完成安装。

在 Ubuntu 浏览器中打开 RedisInsight 官方页面：

```text
https://redis.io/insight/
```

在下载页面的 `Operating system` 选项中选择：

```text
Linux Debian
```

Ubuntu 属于 Debian 系列，因此应选择 `Linux Debian`，而不是 `Linux Red Hat / Fedora`。下载得到的文件名为：

```text
Redis-Insight-linux-amd64.deb
```

进入系统实际下载目录并查看安装包：

```bash
cd "$(xdg-user-dir DOWNLOAD)"
ls -lh
```

使用 APT 安装本地 `.deb` 软件包：

```bash
sudo apt install ./Redis-Insight-linux-amd64.deb
```

安装过程中，APT 正确识别软件包名称 `redisinsight`，并完成解压和配置。实际操作过程如图 1-1 所示。

![图 1-1 RedisInsight 安装过程](Snipaste_2026-10-21_16-35-40.png)

*图 1-1　在 Ubuntu 下载目录中使用 APT 安装 RedisInsight 3.8.0*

安装时曾出现以下提示：

```text
N: 由于文件无法被用户 _apt 访问，已脱离沙盒并提权为根用户来进行下载。
   - pkgAcquire::Run (13: 权限不够)
```

这是因为安装包位于普通用户家目录下，APT 的沙盒用户 `_apt` 无法读取该文件。该提示没有阻止安装，软件包最终安装成功。为避免该问题，也可以先把安装包复制到 `/tmp` 后再安装：

```bash
sudo cp Redis-Insight-linux-amd64.deb /tmp/redis-insight.deb
sudo chmod 644 /tmp/redis-insight.deb
sudo apt install /tmp/redis-insight.deb
```

安装完成后检查软件包状态：

```bash
dpkg-query -W -f='${Status} ${Version}\n' redisinsight
```

实际输出为：

```text
install ok installed 3.8.0
```

该结果说明 RedisInsight 3.8.0 已正确安装，如图 1-2 所示。

![图 1-2 RedisInsight 安装结果验证](Snipaste_2026-10-21_16-38-43.png)

*图 1-2　使用 `dpkg-query` 验证 RedisInsight 已安装及版本号*

---

### （四）启动 RedisInsight

**实验内容：** 打开 RedisInsight，确认图形化界面能够正常启动。

安装完成后，可以在 Ubuntu 应用菜单中搜索 `RedisInsight` 并打开，也可以在终端中执行：

```bash
redisinsight
```

程序启动后进入 `Redis Databases` 页面。此时左侧数据库数量为 `0`，页面提示：

```text
No databases yet, let's add one!
```

表示 RedisInsight 已经正常运行，但尚未添加 Redis 数据库连接。启动界面如图 2-1 所示。

![图 2-1 RedisInsight 初次启动界面](Snipaste_2026-10-21_16-39-56.png)

*图 2-1　RedisInsight 成功启动，当前尚未添加数据库*

---

### （五）配置 RedisInsight 数据库连接

**实验内容：** 在 RedisInsight 中填写 Redis 服务器地址、端口和密码，并测试连接。

在 RedisInsight 页面点击 `Add Redis database`，进入数据库连接配置页面，填写以下内容：

| 配置项 | 填写内容 |
| --- | --- |
| Database alias | `My Redis` |
| Host | `127.0.0.1` |
| Port | `6379` |
| Protocol | `Auto (IPv4 & IPv6)` |
| Username | 留空 |
| Password | `123456` |
| TLS | 关闭 |
| Logical Database | 使用默认数据库 |

填写完成后点击 `Test Connection`。

测试结果显示：

```text
Connection is successful
```

表示 RedisInsight 已使用 `127.0.0.1:6379` 和配置的密码成功连接 Redis 服务。连接配置与测试结果如图 3-1 所示。

![图 3-1 RedisInsight 连接测试成功](Snipaste_2026-10-21_16-45-44.png)

*图 3-1　填写连接信息后测试成功，显示 `Connection is successful`*

测试通过后点击 `Add Redis Database` 保存连接。

---

### （六）确认数据库连接结果

**实验内容：** 返回数据库列表，确认 `My Redis` 已加入 RedisInsight。

保存连接后，RedisInsight 的 `Redis Databases` 列表中新增了：

```text
My Redis
```

其连接信息显示为：

```text
Host:Port         127.0.0.1:6379
Connection Type   Standalone
Last connected    less than a few seconds
```

这说明 RedisInsight 已经成功连接到本人 Ubuntu 虚拟机中的 Redis 服务器，连接结果如图 3-2 所示。

![图 3-2 RedisInsight 数据库连接成功](Snipaste_2026-10-21_16-46-17.png)

*图 3-2　数据库列表中出现 `My Redis`，地址为 `127.0.0.1:6379`*

---

## 四、实验结果

1. 在 Ubuntu 中使用 APT 成功安装并启动了 Redis 服务器。
2. 完成了 Redis 配置文件修改，将监听地址设置为 `bind 0.0.0.0`，并配置访问密码 `requirepass 123456`。
3. 从 Redis 官方页面下载了适用于 Debian/Ubuntu 的 RedisInsight 安装包。
4. 成功安装 RedisInsight 3.8.0，并通过 `dpkg-query` 验证软件包状态为 `install ok installed`。
5. RedisInsight 能够正常启动并显示数据库管理界面。
6. 使用 `127.0.0.1:6379`、空用户名和密码 `123456` 测试连接成功。
7. `My Redis` 数据库成功出现在 RedisInsight 数据库列表中，连接类型为 `Standalone`。

---

## 五、遇到的问题与解决方法

### 1. 下载目录不是 `~/Downloads`

Ubuntu 系统使用中文目录名时，浏览器下载目录可能是 `~/下载`，而不是 `~/Downloads`。可使用以下命令查找系统实际下载目录：

```bash
xdg-user-dir DOWNLOAD
```

随后进入该目录：

```bash
cd "$(xdg-user-dir DOWNLOAD)"
```

### 2. APT 提示 `_apt` 用户没有访问权限

安装本地 `.deb` 文件时出现：

```text
N: Download is performed unsandboxed as root
```

该提示表示 APT 无法通过沙盒用户 `_apt` 读取家目录中的安装包，因此改用 root 权限处理。可以将安装包复制到 `/tmp` 后再执行安装：

```bash
sudo cp Redis-Insight-linux-amd64.deb /tmp/redis-insight.deb
sudo chmod 644 /tmp/redis-insight.deb
sudo apt install /tmp/redis-insight.deb
```

### 3. 软件包名称中是否含有连字符

RedisInsight 的软件包名称是：

```text
redisinsight
```

不是：

```text
redis-insight
```

因此检查安装状态时应使用：

```bash
dpkg-query -W -f='${Status} ${Version}\n' redisinsight
```

### 4. RedisInsight 连接失败时的检查方法

如果 RedisInsight 无法连接，可以先检查 Redis 服务：

```bash
sudo systemctl status redis-server
```

然后验证密码和 Redis 响应：

```bash
redis-cli -a '123456' ping
```

还应确认 `/etc/redis/redis.conf` 中的 `bind` 和 `requirepass` 配置已经生效：

```bash
sudo grep '^bind' /etc/redis/redis.conf
sudo grep '^requirepass' /etc/redis/redis.conf
```

---

## 六、实验总结

本次实验完成了 Redis 服务器的安装、配置、启动和图形化连接。通过 APT 安装 Redis 后，使用 `systemctl` 管理服务状态，并修改 `redis.conf` 中的 `bind` 和 `requirepass` 配置，使 Redis 具备指定的访问方式和密码保护。之后下载并安装 RedisInsight 3.8.0，通过图形化连接界面填写主机地址、端口和密码，最终测试结果显示 `Connection is successful`，数据库列表中也成功出现 `My Redis`。

通过本次实验，我掌握了 Linux 下服务安装与配置的基本流程，理解了配置文件修改后需要重启服务才能生效，也熟悉了 RedisInsight 的连接配置方法。同时认识到：开放 Redis 网络监听时应配置强密码和防火墙，`bind 0.0.0.0` 不应在无保护的公网环境中直接使用。

---

## 附：截图索引

| 编号 | 内容 | 来源 |
| --- | --- | --- |
| 图 1-1 | 使用 APT 安装 RedisInsight 3.8.0 | 本人操作截图 |
| 图 1-2 | 验证 RedisInsight 安装状态与版本 | 本人操作截图 |
| 图 2-1 | RedisInsight 初次启动、尚未添加数据库 | 本人操作截图 |
| 图 3-1 | 填写 Redis 连接信息并测试成功 | 本人操作截图 |
| 图 3-2 | 数据库列表显示 `My Redis` 连接成功 | 本人操作截图 |

> 本报告使用 Markdown 编写。报告与截图位于同一目录时，可直接显示文中的图片。
