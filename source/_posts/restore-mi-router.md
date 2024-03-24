---
title: 使用 Mac 抢救变砖的小米路由器
date: 2021-09-01T23:21:47+08:00
typora-root-url: ../../static
categories:
series:
tags:
  - Router
  - Wi-Fi
  - OpenWrt
  - Debrick
  - DHCP
  - TFTP
---

## 背景

在对小米路由器进行刷机的时候可能因为各种原因导致路由器无法启动，这种情况下无法再刷入新系统，即我们俗称的「变砖」。但是路由器本身提供了一种方式来修复问题。

## 原理

小米路由器在进入 recovery 模式后，会向 192.168.31.1 地址发送 TFTP 协议的请求，请求名为 `test.bin` 的文件，并通过该文件进行刷机。我们要做的就是利用这个方式将确认可用的镜像[^1]刷到路由器中。

## 准备工作

我们需要用到的硬件有：

- 网线一根
- 如果 Mac 没有网口，需要准备一个 USB 网卡

需要的软件有：

* 下载路由器对应的可用镜像[^1]
* Wireshark 不是必须的，不过最好准备一个，否则只能靠猜，并不知道路由器的状态是否符合预期
* 其他必需软件都是 Mac 自带的

首先将路由器断电，将路由器其他端口上的连接线断开，仅保留一个 LAN 口通过网线和 Mac 连接。

## 设置网卡

为了应对[路由器在 recovery 模式下的行为](#原理)，在 Mac 中将有线网卡的 IP 和子网掩码设置如下。其中路由一栏不填：

![image-20210901232508683](image-20210901232508683.png)

## 启动 DHCP 服务

首先备份 `/etc/bootpd.plist` 文件，并使用下面的配置代替。

```plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>bootp_enabled</key>
    <false/>
    <key>detect_other_dhcp_server</key>
    <integer>1</integer>
    <key>dhcp_enabled</key>
    <array>
        <string>en6</string>
    </array>
    <key>reply_threshold_seconds</key>
    <integer>0</integer>
    <key>Subnets</key>
    <array>
        <dict>
            <key>allocate</key>
            <true/>
            <key>lease_max</key>
            <integer>86400</integer>
            <key>lease_min</key>
            <integer>86400</integer>
            <key>name</key>
            <string>192.168.31</string>
            <key>net_address</key>
            <string>192.168.31.0</string>
            <key>net_mask</key>
            <string>255.255.255.0</string>
            <key>net_range</key>
            <array>
                <string>192.168.31.2</string>
                <string>192.168.31.254</string>
            </array>
        </dict>
    </array>
</dict>
</plist>
```

需要注意，我的网卡是 `en6` 所以 `dhcp_enabled` 对应的参数是 `en6`。

网卡的名称可以使用 ifconfig 查看。不会使用的朋友可以使用 Wireshark 辅助确认：首先通过网络选项查看网卡的名字（如我的网卡叫 `USB 10/100/1000 LAN`，这个因人而异），再通过 Wireshark 找到对应名称的网卡，名称后面显示的就是网卡序号。另外需要注意的是，设备不插入的时候可能不会显示对应的网卡，遇到这样的情况可以先将路由器供电后和 Mac 连接，记下网卡序号后再断电即可。

完成配置后执行如下命令启动 DHCP 服务：

```bash
sudo /bin/launchctl load -w /System/Library/LaunchDaemons/bootps.plist
```

## 启动 Wireshark

启动 Wireshark 并监测 Mac 和路由器连接的网口，在筛选框中输入 `arp || tftp` 筛选这两种协议的记录。

## 启动路由器 Recovery 模式

在路由器不通电的情况下长按 reset 按钮，并重新接入电源，这时橙色提示灯常亮，等待几秒橙色提示灯开始闪烁，此时可以松开按钮。

在 Wireshark 中应该看到路由器询问 192.168.31.1 的设备[^no50]，再通过 TFTP 协议请求 `test.bin` 文件[^no53]。

![image-20210901232535559](image-20210901232535559.png)

## 在 Mac 中启动 TFTP 服务器

我们可以看到路由器（192.168.31.2）请求 `test.bin` 文件超时，因为我们还没有启动 TFTP Server。

将我们的镜像放到目录 `/private/tftpboot` 下，命名为 `test.bin`。执行命令启动 TFTP Server

```bash
sudo launchctl load -F /System/Library/LaunchDaemons/tftp.plist
```

应该看到 TFTP 服务很快开始传入并传输完毕。

![image-20210901232557933](image-20210901232557933.png)

![image-20210901232629308](image-20210901232629308.png)

大约两三分钟后就可以看到路由器上蓝灯闪烁，说明烧录完毕，可以重启路由器。我们先将路由器电源断开。

## 验证结果

首先修改 Mac 的网络配置，将之前配置的固定 IP 改为 DHCP 自动配置 IP。否则 Mac 会和我们的路由器抢 192.168.31.1 这个 IP。

在路由器断电大概10秒过后，插入电源启动路由器。路由器应该会显示橙色灯常亮，之后蓝灯常亮。这说明我们的路由器恢复正常了。

## 环境恢复

停止 TFTP 服务器：

```bash
sudo launchctl unload -F /System/Library/LaunchDaemons/tftp.plist
```

停止 DHCP 服务器

```bash
sudo /bin/launchctl unload -w /System/Library/LaunchDaemons/bootps.plist
```

## 相关链接

- Mac 运行 DHCP 服务参考文章：[Running Mac OS X's built-in DHCP server - swiss network security - swissns GmbH](https://www.swissns.ch/site/2014/05/running-mac-os-xs-built-in-dhcp-server/)
- 使用 Wireshark 调试参考文章：[How to Unbrick TP-Link WiFi Router WR841ND using TFTP and Wireshark | Alexander Molochko](https://crosp.net/blog/hardware/unbrick-tp-link-wifi-router-wr841nd-with-tftp-wireshark/)
- TFTP 服务启动参考文章：[Run a TFTP Server for Network Device Setups : Rick Cogley Central](https://rick.cogley.info/post/run-a-tftp-server-on-mac-osx/)

- 查看路由器提示灯含义：[【路由刷机教程】适用于小米路由器刷机工具 - 小米社区](https://www.xiaomi.cn/post/19134127#:~:text=%E3%80%90%E8%B7%AF%E7%94%B1%E5%99%A8%E6%8C%87%E7%A4%BA%E7%81%AF%E7%8A%B6%E6%80%81%E8%AF%B4%E6%98%8E%E3%80%91)
- 有网友[提供了基于 Docker 的一键安装方式](https://gitlab.com/db260179/xiaomi-m4a/-/blob/stable/README_mi4a.md)，从理论上是可行的，未验证，可以作为参考，同样需要注意需要配置 Mac 的网卡使用固定 IP 地址。
- 另外再推荐一个好用的小米路由器刷 OpenWrt 工具：[acecilia/OpenWRTInvasion: Root shell exploit for several Xiaomi routers: 4A Gigabit, 4A 100M, 4, 4C, 3Gv2, 4Q, miWifi 3C...](https://github.com/acecilia/OpenWRTInvasion)

[^1]: 建议先使用[官网提供的 ROM](http://www.miwifi.com/miwifi_download.html) 完成修复。
[^no50]: 记录 No. 50
[^no53]: 记录 No. 53

