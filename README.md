# BT-WebPanel 安装脚本说明

> 本项目提供 `install.sh`，用于在 Linux 服务器上部署 BT-WebPanel（宝塔面板）运行环境。
>
> ⚠️ **重要提示：这是高影响系统安装脚本。**
>
> 脚本会以 `root` 身份安装软件、下载并执行远程资源、创建 `/www` 目录、配置服务、调整防火墙规则，并可能修改 SELinux、Swap、系统时间、软件源及已有面板文件。  
> **请仅在全新系统、测试机或已完成完整备份的服务器上运行。**

---

## 功能概览

`install.sh` 的主要工作包括：

- 检查当前用户是否为 `root`
- 检查系统是否为 64 位，并拒绝不支持的旧系统
- 检测已有的 Nginx、Apache、PHP-FPM、MySQL/MariaDB 等服务
- 自动识别 Yum/DNF 或 APT 包管理器
- 自动选择下载节点并下载面板及运行依赖
- 安装系统依赖、Python 运行环境及 Python 包
- 将面板安装到 `/www/server/panel`
- 创建网站、日志、备份等目录
- 创建并启用 `bt` 服务
- 生成面板账号、密码、随机安全入口
- 自动设置面板端口
- 配置防火墙开放常用端口及面板端口
- 在内存较小时自动创建约 1 GB Swap
- 支持通过命令行参数指定面板账号、密码、端口和安全入口
- 安装完成后输出外网访问地址、内网访问地址、账号与密码

---

## 系统要求

### 基础要求

- 64 位 Linux 系统
- `root` 用户权限
- 能访问脚本使用的下载站点与接口
- 建议使用全新、纯净的 VPS 或云服务器
- 建议至少 1 GB 内存；内存不足时脚本会尝试创建 Swap
- 磁盘需要有足够空间用于面板、运行环境、网站文件、日志和备份

### 脚本明确不支持的环境

以下环境会被脚本拒绝或可能无法正常安装：

| 环境 | 说明 |
|---|---|
| 32 位系统 | 脚本要求 64 位架构 |
| CentOS 6 | 脚本会直接退出 |
| Ubuntu 16 以下 | 脚本会直接退出 |
| 空 hostname 的系统 | 脚本会退出 |
| 已部署生产 Web/MySQL 环境的服务器 | 脚本会警告可能造成冲突或影响现有业务 |

### 兼容系统

脚本包含 Yum/DNF 与 APT 安装逻辑，主要面向以下系统系列：

- CentOS / RHEL / Rocky Linux / AlmaLinux 等 RPM 系统
- Ubuntu
- Debian
- Alibaba Cloud Linux、TencentOS、OpenCloudOS、Anolis 等部分兼容发行版
- Arch Linux 存在基础依赖安装逻辑，但实际兼容性需要自行测试

> 建议优先在较新的、纯净的 Ubuntu、Debian 或 RHEL 兼容发行版中测试使用。

---

## 安装前准备

### 1. 备份重要数据

如果服务器上已经存在网站、数据库、Docker 容器、反向代理或其他服务，请先备份：

- 网站目录
- 数据库文件与数据库逻辑备份
- `/etc/nginx/`
- `/etc/apache2/` 或 `/etc/httpd/`
- `/etc/mysql/`、`/var/lib/mysql/`
- `/etc/ssh/sshd_config`
- `/etc/hosts`
- `/etc/fstab`
- `/etc/yum.repos.d/` 或 `/etc/apt/`
- 防火墙规则

### 2. 确认 SSH 可访问

安装脚本会配置防火墙。执行前请确认：

- SSH 端口正确
- 云服务商安全组已放行 SSH 端口
- 你拥有服务器控制台/VNC/救援模式访问能力
- 不要在未知 SSH 端口或远程连接不稳定时盲目执行

### 3. 检查系统资源

建议先执行：

```bash
uname -m
cat /etc/os-release
free -h
df -h
hostname
```

确认：

- 架构为 `x86_64`、`aarch64` 或其他受支持的 64 位架构
- 根分区与 `/www` 所在磁盘有充足空间
- hostname 不为空
- 网络能够正常访问外网

### 4. 查看脚本内容

不要直接对未知来源脚本执行 `curl | bash`。建议先上传或下载脚本后检查：

```bash
less install.sh
```

也可以检查文件基本信息：

```bash
ls -lh install.sh
head -n 30 install.sh
```

---
## 安装方法

## 一键安装：

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/kkkbox/bt/main/install.sh)
```

如果服务器没有安装 `curl`，可使用 `wget`：

```bash
bash <(wget -qO- https://raw.githubusercontent.com/kkkbox/bt/main/install.sh)
```

## 非交互式安装

跳过脚本中的 `y/n` 确认提示，直接开始安装：

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/kkkbox/bt/main/install.sh) -y
```

## 自定义面板信息

例如指定面板用户名、密码、端口和安全入口：

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/kkkbox/bt/main/install.sh) \
  -y \
  --user admin \
  --password 'ChangeThisToAStrongPassword_2026!' \
  --port 18888 \
  --safe-path my-private-panel
```

安装后通常通过以下形式访问：

```text
http://服务器IP:18888/my-private-panel
```

若脚本启用了面板 SSL，则使用：

```text
https://服务器IP:18888/my-private-panel
```

## 命令行参数

脚本支持以下参数：

| 参数 | 长参数 | 说明 |
|---|---|---|
| `-u` | `--user` | 指定面板登录用户名 |
| `-p` | `--password` | 指定面板登录密码 |
| `-P` | `--port` | 指定面板访问端口 |
| 无 | `--safe-path` | 指定面板安全入口路径 |
| `-y` | 无 | 跳过交互确认，直接开始安装 |

## 更安全的推荐方式

生产服务器不建议直接 `curl | bash`。可以先下载、检查，再执行：

```bash
curl -fL https://raw.githubusercontent.com/kkkbox/bt/main/install.sh -o install.sh && \
chmod +x install.sh && \
less install.sh
```

确认内容无误后执行：

```bash
sudo bash install.sh -y
```

脚本要求 `root` 权限，并会下载远程组件、安装依赖、配置系统服务与防火墙、可能创建 Swap、修改 SELinux/软件源设置；请在干净系统或已有完整快照的服务器上使用。脚本支持 `-y`、`--user`、`--password`、`--port`、`--safe-path` 等参数。 

---

## 安装目录与文件

脚本默认使用 `/www` 作为安装根目录。

| 路径 | 用途 |
|---|---|
| `/www/server/panel` | 面板主程序目录 |
| `/www/server/panel/pyenv` | 面板 Python 运行环境 |
| `/www/server/panel/data` | 面板配置、端口、账号与数据库文件 |
| `/www/server/panel/logs` | 面板日志目录 |
| `/www/wwwroot` | 网站根目录 |
| `/www/wwwlogs` | 网站日志目录 |
| `/www/backup/site` | 网站备份目录 |
| `/www/backup/database` | 数据库备份目录 |
| `/www/swap` | 脚本自动创建的 Swap 文件，视系统内存情况而定 |
| `/etc/init.d/bt` | 面板服务脚本 |
| `/usr/bin/bt` | `bt` 管理命令软链接 |
| `/var/bt_setupPath.conf` | 安装路径记录文件 |

---

## 安装完成后

安装结束时，脚本会在终端输出类似信息：

```text
外网面板地址: http://SERVER_IP:面板端口/安全入口
内网面板地址: http://内网IP:面板端口/安全入口
username: 用户名
password: 密码
```

请立即将以下信息保存到密码管理器：

- 面板外网地址
- 面板内网地址
- 面板用户名
- 面板密码
- 面板端口
- 面板安全入口路径
- 服务器 SSH 登录方式
- 云服务器安全组配置

### 查看当前面板端口

```bash
cat /www/server/panel/data/port.pl
```

### 查看当前安全入口

```bash
cat /www/server/panel/data/admin_path.pl
```

### 查看初始密码文件

```bash
cat /www/server/panel/default.pl
```

> 初始密码文件属于敏感信息。确认登录成功并修改密码后，建议妥善处理并限制文件访问权限。

---

## 面板服务管理

脚本会安装 `bt` 服务，并创建 `/usr/bin/bt` 命令链接。

### 启动面板

```bash
bt start
```

或：

```bash
/etc/init.d/bt start
```

### 停止面板

```bash
bt stop
```

### 重启面板

```bash
bt restart
```

### 查看面板状态

```bash
bt status
```

### 查看面板日志

```bash
tail -f /www/server/panel/logs/error.log
```

如日志文件名称不同，可先查看日志目录：

```bash
ls -lah /www/server/panel/logs/
```

---

## 防火墙与端口

脚本会尝试配置系统防火墙，并开放以下 TCP 端口：

| 端口 | 用途 |
|---:|---|
| 20 | FTP 数据连接 |
| 21 | FTP 控制连接 |
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 面板随机端口或指定端口 | 面板访问 |
| SSH 实际端口 | 从 `sshd_config` 中读取 |
| 39000-40000 | 被动 FTP 端口范围 |

不同系统使用的防火墙方案可能不同：

- Ubuntu / Debian：尝试安装并配置 `ufw`
- CentOS 7 等旧系统：可能使用 `iptables`
- 较新 RPM 系统：可能安装并使用 `firewalld`

### 云服务器安全组

即使脚本已放行本机防火墙，云服务器仍可能无法访问面板。请在云平台控制台的安全组/防火墙中额外放行：

- SSH 端口
- 面板端口，例如 `18888`
- 网站端口 `80`、`443`
- 若使用 FTP，还需放行 `20`、`21` 与 `39000-40000`

---

## 脚本的系统修改

执行前请了解脚本可能进行的系统修改。

### 软件与依赖

脚本可能安装：

- `curl`
- `wget`
- `tar`
- `gcc`
- `make`
- `zip` / `unzip`
- OpenSSL 相关组件
- Python 编译与运行依赖
- `firewalld`、`ufw` 或其他防火墙组件
- `ntp`、`ruby`、`lsb-release` 等系统包
- 多个 Python 依赖包

### SELinux

在 RPM 系统中，脚本会执行：

```bash
setenforce 0
```

并尝试将：

```text
SELINUX=enforcing
```

修改为：

```text
SELINUX=disabled
```

即可能临时或永久关闭 SELinux。

### 系统时间与时区

脚本包含时间同步逻辑，并在部分 RPM 系统中设置：

```text
Asia/Shanghai
```

时区。

如果你的服务器需要保持 UTC、美国时区或其他业务时区，请在安装后检查：

```bash
timedatectl
```

例如设置为洛杉矶时区：

```bash
timedatectl set-timezone America/Los_Angeles
```

### Swap

当脚本检测到服务器内存不大于 1 GB 且没有有效 Swap 时，可能会：

- 创建 `/www/swap`
- 创建约 1 GB Swap 文件
- 启用 Swap
- 将 Swap 写入 `/etc/fstab`

查看 Swap 状态：

```bash
free -h
swapon --show
```

### 软件源

在特定 CentOS、阿里云或华为云环境中，脚本可能修改或备份 Yum 仓库配置，例如：

```text
/etc/yum.repos.d/
/etc/yumBak/
```

安装完成后如遇到 Yum/DNF 更新问题，请优先检查软件源配置。

---

## 已有环境冲突

脚本会检测以下正在运行的服务：

- MySQL / MariaDB
- PHP-FPM
- Nginx
- Apache / httpd

如检测到已有 Web 或数据库服务，脚本会提示：

```text
检查已有其他Web/mysql环境，安装宝塔可能影响现有站点及数据
输入yes强制安装:
```

除非你明确理解现有环境、端口、配置文件、数据目录和服务管理方式，否则不要输入 `yes` 强制安装。

### 不建议直接安装的场景

- 已经运行生产网站
- 已经使用 Docker 部署 Nginx、MySQL、PHP、WordPress 等服务
- 已经运行 Caddy、Traefik、Nginx Proxy Manager
- 已经部署其他服务器面板
- 已经有现成的 LEMP/LAMP 环境
- 服务器磁盘空间不足
- 云服务器安全组策略严格且不能自行调整

---

## 常见问题

### 1. 提示必须使用 root 权限

错误示例：

```text
请使用root权限执行宝塔安装命令！
```

解决方法：

```bash
sudo -i
bash install.sh
```

或：

```bash
sudo bash install.sh
```

---

### 2. 提示不支持 32 位系统

错误示例：

```text
当前面板版本不支持32位系统
```

解决方法：

- 更换为 64 位 Linux 系统
- 新建 64 位 VPS
- 不建议继续在 32 位系统上尝试安装

---

### 3. 面板安装后无法访问

请按以下顺序排查。

确认面板服务状态：

```bash
bt status
```

重启面板：

```bash
bt restart
```

查看监听端口：

```bash
PANEL_PORT=$(cat /www/server/panel/data/port.pl)
ss -lntp | grep "$PANEL_PORT"
```

查看防火墙规则：

```bash
firewall-cmd --list-ports 2>/dev/null
```

或：

```bash
ufw status
```

检查云安全组是否已放行面板端口。

确认访问地址中包含正确的安全入口：

```bash
cat /www/server/panel/data/admin_path.pl
```

---

### 4. 浏览器提示证书不安全

若安装过程中启用了面板 SSL，脚本可能使用自签名证书。浏览器提示不安全属于预期现象。

处理方式：

- 确认访问的是自己的服务器地址
- 通过浏览器“高级”选项继续访问
- 登录后配置自己的域名与受信任 SSL 证书
- 不要在不可信网络、未知 IP 或非本人服务器上忽略证书告警

---

### 5. 下载失败或网络超时

脚本会尝试从多个下载节点获取资源。若仍失败，请检查：

```bash
curl -I [https://download.bt.cn/](https://download.bt.cn/)
curl -I [https://new.baota.sbs/](https://new.baota.sbs/)
ping -c 4 1.1.1.1
```

还应检查：

- DNS 是否正常
- IPv4/IPv6 路由是否可用
- 服务器是否被出口防火墙限制
- 系统时间是否严重错误
- 云厂商网络是否限制相关域名
- `/tmp`、`/dev/shm`、`/www` 是否可写

---

### 6. 安装后 SSH 无法连接

脚本会调整防火墙规则。请优先通过云厂商控制台、VNC 或救援模式进入系统，检查：

```bash
ss -lntp | grep ssh
grep -E '^[[:space:]]*Port' /etc/ssh/sshd_config
systemctl status sshd
```

对于 UFW：

```bash
ufw status numbered
ufw allow 22/tcp
```

对于 firewalld：

```bash
firewall-cmd --list-ports
firewall-cmd --permanent --add-service=ssh
firewall-cmd --reload
```

如 SSH 使用自定义端口，请替换命令中的 `22`。

---

## 安全建议

安装完成后，建议立即执行以下安全措施：

1. 修改默认或安装时设置的面板密码。
2. 使用高强度、唯一密码，不要复用 SSH、邮箱或数据库密码。
3. 不要将面板端口暴露给所有公网 IP。
4. 在云安全组中仅允许自己的固定 IP 访问面板端口。
5. 配置 HTTPS、域名和可信证书。
6. 定期更新系统与面板组件。
7. 定期备份 `/www`、网站文件和数据库。
8. 使用 SSH 密钥登录，并尽量关闭密码登录。
9. 修改 SSH 默认端口前，确认安全组与防火墙规则已同步更新。
10. 不要在生产服务器上直接运行未经审查的第三方脚本。
11. 定期检查 `/www/server/panel/logs/` 中的异常日志。
12. 若不需要 FTP，请不要暴露 FTP 端口和被动端口范围。

---

## 卸载与回滚说明

本脚本未提供完整、无损的一键卸载逻辑。

因为安装过程会修改：

- 软件包
- 系统服务
- 防火墙规则
- SELinux 配置
- Swap 配置
- Yum 软件源
- 时区或系统时间设置
- `/www` 目录与其内部数据

因此，最可靠的回滚方式是：

1. 在安装前创建 VPS 快照。
2. 在测试环境确认流程无误后再用于正式环境。
3. 若需要彻底清理，优先从快照恢复或重装系统。
4. 不建议仅通过删除 `/www` 目录来“卸载”，否则可能残留服务、防火墙规则、系统包和配置修改。

---

## 使用风险声明

本脚本会从远程站点下载面板文件、初始化脚本、Python 环境及依赖包，并以 `root` 权限执行安装操作。运行脚本前，请自行完成：

- 脚本代码审查
- 下载域名与镜像源可信度确认
- 数据与快照备份
- 网络与防火墙规划
- 云安全组规则配置
- 测试环境验证

因使用本脚本导致的数据丢失、服务中断、端口暴露、配置冲突或安全问题，应由使用者自行评估并承担相应风险。
