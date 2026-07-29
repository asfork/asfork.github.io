---
title: "搬瓦工 VPS 购买与 sing-box 一键部署教程"
date: 2026-06-12
description: "从购买 BandwagonHost（搬瓦工）VPS、安装 Debian，到使用 fscarmen/sing-box 一键脚本部署代理并导入客户端的完整教程。"
tags:
  - 搬瓦工
  - BandwagonHost
  - VPS
  - sing-box
  - Linux
---

# 搬瓦工 VPS 购买与 sing-box 一键部署教程

想自己搭建一台 sing-box 服务器，但不熟悉 VPS 购买、Linux 命令和节点配置？这篇文章将从购买搬瓦工（BandwagonHost）开始，带你完成系统安装、SSH 登录、服务器初始化，以及使用 [fscarmen/sing-box](https://github.com/fscarmen/sing-box/) 一键脚本部署 sing-box。

整个过程不要求你手工编写 JSON 配置，准备好一台 VPS 和一个 SSH 客户端即可。

> **合规提示：** 请遵守服务器所在地及使用所在地的法律法规、服务商条款和网络使用政策。本教程仅用于合法的远程访问、隐私保护、网络测试与技术学习。

> **推广说明：** 本文中的搬瓦工购买链接包含 Affiliate ID `81109`。如果你通过该链接购买，我可能获得推广佣金，但不会增加你的订单价格。感谢支持。

## 一、为什么选择搬瓦工部署 sing-box

搬瓦工是 BandwagonHost 的中文俗称，其 VPS 使用 KVM 虚拟化，并通过自研的 KiwiVM 面板提供重装系统、开关机、备份、快照、迁移机房和查看流量等功能。

对新手来说，它比较适合部署 sing-box 的原因包括：

- 提供独立公网 IP 和完整 root 权限；
- 可在 KiwiVM 面板中快速重装 Debian、Ubuntu 等 Linux 系统；
- 套餐通常包含月流量、快照和自动备份；
- 部分套餐可以在多个数据中心之间迁移；
- 1 GB 内存的入门配置已经足以运行个人使用的 sing-box。

搬瓦工属于自管理 VPS。服务商负责服务器硬件和网络，系统安全、软件安装、配置备份和日常维护需要由用户自己完成。

## 二、购买搬瓦工 VPS

### 1. 打开搬瓦工官网

通过下面的推广链接进入搬瓦工官网：

**[进入搬瓦工 BandwagonHost 官网（Affiliate ID：81109）](https://bandwagonhost.com/aff.php?aff=81109)**

如果主域名无法打开，也可以尝试官方镜像域名：

**[通过 bwh81.net 镜像进入（Affiliate ID：81109）](https://bwh81.net/aff.php?aff=81109)**

### 2. 选择套餐

如果只用于个人 sing-box 节点，一般不需要购买高配服务器。建议重点关注：

| 项目 | 入门建议 |
|---|---|
| 内存 | 1 GB 或以上 |
| 硬盘 | 10–20 GB 或以上 |
| CPU | 1 核或以上 |
| 月流量 | 按实际使用量选择，建议至少 500 GB |
| 虚拟化 | KVM |
| 公网地址 | 至少 1 个独立 IPv4 |

截至本文更新时，官网普通 PROMO 入门方案为 20 GB SSD、1 GB 内存、2× Intel Xeon、每月 1 TB 流量，年付 49.99 美元。套餐价格、库存和可选机房可能随时变化，请以下单页面显示为准。

机房并不是越贵就一定越适合你。实际速度与所在地区、宽带运营商、国际出口和晚高峰拥堵情况都有关系。比较稳妥的做法是：

1. 先根据预算选择套餐；
2. 优先选择物理距离较近或针对自己网络优化的机房；
3. 开通后分别测试延迟、丢包和晚高峰速度；
4. 如果套餐支持免费迁移，再通过 KiwiVM 尝试其他机房。

### 3. 填写订单

进入套餐页面后，依次完成：

1. 选择付款周期；
2. 确认套餐配置和机房范围；
3. 注册或登录 BandwagonHost 账户；
4. 填写真实、有效的联系信息；
5. 选择页面当前支持的付款方式；
6. 核对续费价格、退款条件和服务条款；
7. 完成支付。

不要盲目使用网上流传多年的优惠码。优惠码可能过期，也可能只适用于特定套餐，以结算页面实际显示的折扣和最终价格为准。

## 三、开通 VPS 并安装 Debian

支付完成后，登录搬瓦工客户中心：

1. 打开 **Services > My Services**；
2. 找到刚购买的 VPS；
3. 点击 **KiwiVM Control Panel**；
4. 在面板中查看服务器 IP、状态和流量信息；
5. 打开 **Install new OS**，选择 Debian 的稳定版本；
6. 确认重装并保存系统生成的 root 密码。

本文建议使用 Debian 12 或服务商当前提供的较新稳定版。`fscarmen/sing-box` 项目同时支持 Ubuntu、Debian、CentOS、Alpine、Armbian 和 Arch Linux，并建议优先使用长期支持或稳定版本。

> 重装系统会清空 VPS 上的全部数据。如果服务器不是刚购买的空机器，请先完成备份。

## 四、使用 SSH 登录服务器

假设服务器 IP 是 `203.0.113.10`，在 Windows PowerShell、macOS 终端或 Linux 终端中运行：

```bash
ssh root@203.0.113.10
```

请把示例 IP 替换成 KiwiVM 面板显示的真实 IP。

首次连接时会看到服务器指纹确认提示。核对无误后输入：

```text
yes
```

接着输入重装系统时生成的 root 密码。输入密码时终端不会显示星号或字符，这是正常现象。

登录后可以先确认系统信息：

```bash
cat /etc/os-release
```

然后更新系统并安装常用工具：

```bash
apt update
apt -y upgrade
apt -y install wget curl ca-certificates
```

如果升级了内核，可以重启一次：

```bash
reboot
```

等待约一分钟，然后重新通过 SSH 登录。

## 五、一键部署 sing-box

### 方案 A：中文极速安装

`fscarmen/sing-box` 提供中文快速部署参数 `-l`。运行：

```bash
bash <(wget -qO- https://raw.githubusercontent.com/fscarmen/sing-box/main/sing-box.sh) -l
```

脚本会自动补充安装参数并部署多种协议。根据服务器性能和 GitHub 网络状况，安装通常需要几分钟。执行期间不要关闭 SSH 窗口。

安装完成后，请保存终端输出的以下信息：

- 节点链接；
- 订阅地址；
- UUID、密码和端口；
- Reality 公钥、短 ID 等连接参数；
- 脚本生成的二维码或客户端配置。

这些信息相当于服务器的访问凭证，不要公开到博客截图、网盘或聊天群中。

### 方案 B：先检查脚本再运行

一键命令会以 root 权限直接执行远程脚本。如果你更重视安全，可以先下载并阅读脚本：

```bash
wget -O sing-box.sh https://raw.githubusercontent.com/fscarmen/sing-box/main/sing-box.sh
less sing-box.sh
```

确认来源和内容后，再运行中文极速安装：

```bash
bash sing-box.sh -l
```

按 `q` 可以退出 `less` 查看界面。

### 方案 C：交互式安装

如果你不希望一次启用所有协议，可以运行不带 `-l` 的交互模式：

```bash
bash <(wget -qO- https://raw.githubusercontent.com/fscarmen/sing-box/main/sing-box.sh)
```

根据菜单选择语言、协议、端口和订阅设置。个人使用时没有必要启用所有协议；可以先部署客户端支持良好的 Reality 类协议，并按网络情况增加 Hysteria2 或其他协议。

## 六、查看和管理 sing-box

安装后，直接输入下面的命令即可打开管理菜单：

```bash
sb
```

常用命令如下：

| 命令 | 用途 |
|---|---|
| `sb` | 打开管理菜单 |
| `sb -n` | 重新显示或导出节点信息 |
| `sb -d` | 修改配置参数 |
| `sb -r` | 添加或删除协议 |
| `sb -s` | 停止或启动 sing-box 服务 |
| `sb -a` | 停止或启动 Argo Tunnel |
| `sb -u` | 卸载 |

还可以使用 systemd 检查服务状态：

```bash
systemctl status sing-box --no-pager
```

检查服务器正在监听的端口：

```bash
ss -lntup
```

如果想确认服务是否随系统启动，可以执行：

```bash
systemctl is-enabled sing-box
```

## 七、把节点导入客户端

该脚本可以输出适用于 V2rayN、Clash Verge、Shadowrocket、Throne 和 sing-box 客户端的节点或订阅信息。最方便的方式通常是使用安装完成后生成的订阅地址。

通用导入步骤：

1. 在服务器上运行 `sb -n`；
2. 找到与你的客户端对应的订阅地址或节点链接；
3. 打开客户端的“订阅”“配置”或“从剪贴板导入”功能；
4. 粘贴链接并更新订阅；
5. 选择一个节点，先测试延迟，再尝试建立连接。

如果客户端支持扫码，也可以扫描脚本生成的二维码。扫描前应确认屏幕周围没有无关人员，也不要把二维码截图上传到公开平台。

## 八、基础安全设置

### 1. 修改 root 密码

登录服务器后运行：

```bash
passwd
```

使用长度足够、没有在其他网站重复使用的随机密码。

### 2. 配置 SSH 密钥

在自己的电脑上生成密钥：

```bash
ssh-keygen -t ed25519
```

将公钥复制到服务器后，确认密钥登录正常，再考虑关闭 SSH 密码登录。不要在未验证密钥可用前直接禁用密码，否则可能把自己锁在服务器外。

### 3. 保持系统和脚本更新

定期更新 Debian：

```bash
apt update
apt -y upgrade
```

同时关注 [fscarmen/sing-box 项目更新记录](https://github.com/fscarmen/sing-box/)；升级脚本或 sing-box 前，建议先在 KiwiVM 创建快照。

### 4. 减少不需要的协议

协议和监听端口越多，维护成本也越高。通过下面的命令移除不用的协议：

```bash
sb -r
```

只保留自己真正使用的节点，并定期检查：

```bash
ss -lntup
```

### 5. 保护订阅地址

订阅 URL、UUID、密码、Reality 密钥和节点二维码都属于敏感信息。若怀疑泄露，应立即通过 `sb` 管理菜单重新生成相关凭证，而不是只修改客户端中的节点名称。

## 九、常见问题

### 1. 提示 `wget: command not found`

运行：

```bash
apt update
apt -y install wget ca-certificates
```

然后重新执行安装命令。

### 2. 无法下载 GitHub 脚本

先测试：

```bash
curl -I https://raw.githubusercontent.com/fscarmen/sing-box/main/sing-box.sh
```

如果连接失败，可能是 VPS 到 GitHub 的临时网络或 DNS 问题。稍后重试，并检查：

```bash
cat /etc/resolv.conf
```

不要从来历不明的网盘或陌生镜像下载修改版 root 脚本。

### 3. 安装成功，但客户端无法连接

依次检查：

```bash
systemctl status sing-box --no-pager
ss -lntup
sb -n
```

然后确认：

- 客户端中的 IP、端口、UUID、密码和 Reality 参数是否完整；
- VPS 或系统防火墙是否放行了对应 TCP/UDP 端口；
- 客户端时间和服务器时间是否准确；
- 当前网络是否限制 UDP；如果是，先测试 TCP 类协议；
- 订阅是否已经更新到最新配置。

### 4. SSH 断开后怎样继续管理

重新连接服务器，然后运行：

```bash
sb
```

脚本安装完成后会创建 `sb` 快捷命令，不需要每次重新下载安装脚本。

### 5. 怎样重新查看订阅和节点

运行：

```bash
sb -n
```

### 6. 怎样卸载

运行：

```bash
sb -u
```

卸载后再次使用 `ss -lntup` 检查监听端口，并根据需要删除不再使用的防火墙规则和订阅记录。

## 十、总结

完整流程可以概括为：

1. 通过[搬瓦工推广链接](https://bandwagonhost.com/aff.php?aff=81109)购买合适的 KVM VPS；
2. 在 KiwiVM 面板重装 Debian 稳定版；
3. 使用 SSH 登录服务器并更新系统；
4. 运行 `fscarmen/sing-box` 中文极速安装命令；
5. 使用 `sb -n` 获取订阅或节点信息；
6. 将配置导入客户端并完成连接测试；
7. 配置 SSH 密钥、定期更新，并保护好订阅凭证。

一键脚本降低了部署门槛，但 VPS 仍然需要日常维护。请保持系统更新、减少无用端口、妥善保管访问凭证，并在重大升级前创建快照。

## 参考资料

- [BandwagonHost 官方网站](https://bandwagonhost.com/)
- [BandwagonHost VPS 套餐页面](https://bandwagonhost.com/cart.php)
- [BandwagonHost Affiliate Program](https://bandwagonhost.com/affiliates-info.php)
- [fscarmen/sing-box GitHub 项目](https://github.com/fscarmen/sing-box/)

