---
title: Shadowsocks代理简介和配置
date: 2016-12-06 16:54:13
tags: 
- shadowsocks
---

# Shadowsocks简介

[Shadowsocks](https://github.com/shadowsocks/shadowsocks)，中文翻译为`影梭`，是一个开源的轻量级的代理工具，可提供安全稳定的Socks5代理。该项目由国人[clowwindy](https://github.com/clowwindy)开源。至于当时和[breakwa11](https://github.com/breakwa11)的争议以及喝茶事件，各位可自行google。

Shadowsocks分为服务端和客户端。服务端安装在提供网络流量出口的服务器上，客户端安装在本地或代理用户所在网络环境中的机器上，提供Socks5代理服务。目前客户端已经支持多种平台，既包括Windows，Linux，Mac OS X等PC系统，也包括iOS，Android等移动平台，也就是说配置一个代理，可以非常方便地为多种终端设备使用，有一种毕其功于一役的痛快感觉。服务端和客户端之间通过类似于Socks5但更简单且加密的协议进行通讯，同时支持TCP和UDP连接。

## Shadowsocks VS SSH Tunnel

Shadowsocks相对于SSH Tunnel在穿越Firewall的优势在于：

1. SSH Tunnel创建隧道使用的是长连接，一旦受到墙的干扰，连接会被中断需要重连。而Shadowsocks连接是实时模式，只有在有数据访问的时候才会创建隧道，并且不需要保持连接，其抗干扰和丢包容错的能力都更强，更适合国情。

2. SSH Tunnel的连接特征相对明显，容易被Firewall通过特征分析进行定向干扰；相对而言Shadowsocks没有明显特征码，被干扰的可能性更小。

# Shadowsocks安装和配置

## 服务端安装和配置

由于Shadowsocks基于python实现，可以方便地使用`pip`安装。至于如何安装python环境和pip包管理工具，可自行google。

```
pip install shadowsocks
```

服务端的运行也非常简单，可以直接一句命令行启动，具体见[文档](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)。也可以通过配置文件的方式来设置运行参数。配置文件可放在任意目录，这里保存为`~/shadowsocks/config.json`。

```
{
    "server":"remote-server-ip",
    "server_port":7777,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"your-password",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open":false,
    "workers":1
}
```

配置内容非常直观，注册修改`server`和`password`，属性含义可参考[官方文档](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)。接着以后台运行的方式启动：

```
ssserver -c ~/shadowsocks/config.json -d start
```

停止服务：

```
ssserver -c ~/shadowsocks/config.json -d stop
```

## 客户端安装和配置

Shadowsocks针对不同平台有不同的图形界面客户端。PC端客户端支持基于PAC的自动代理模式，有[MAC版](https://github.com/shadowsocks/shadowsocks-iOS/releases)，[Windows版](https://github.com/shadowsocks/shadowsocks-windows/releases)等。移动端ios/android可以在各自应用市场下载，支持分应用的代理设置。各种配置都非常直观，这里就不多说了。

除了图形界面客户端，本人更喜欢直接在命令行里启动`sslocal`来启动本机的socks5代理。这里以Mac OS为例。

同样使用pip安装。客户端命令为`sslocal`。可以直接使用与服务端相同的配置文件运行，也可以用`nohup`后台运行。

```
pip install shadowsocks
sslocal -c ~/shadowsocks/config.json
nohup sslocal -c ~/shadowsocks/config.json &
```

此时一切就绪，其他应用就可以使用Socks5代理`127.0.0.1:1080`，最终通过服务端出口来访问网络了。对于Chrome，我们可以使用扩展`SwitchyOmega`来自动直连或使用代理。

不过对于Mac更方便的是使用[Launchd](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html)实现开机自启动。`Launchd`分为Launch Daemons和Agents，前者为系统级，在系统启动时自动运行，后者为用户级，在用户登录时自动运行。只要在系统特定目录创建Property List文件，系统就会自动加载运行。

这里我们使用Launch Agent，创建`plist`文件`/Library/LaunchAgents/local.shadowsocks.proxy.plist`，内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>local.shadowsocks.proxy</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/sslocal</string>
        <string>-c</string>
        <string>config.json</string>
    </array>
    <key>StandardOutPath</key>
    <string>info.log</string>
    <key>StandardErrorPath</key>
    <string>error.log</string>
    <key>WorkingDirectory</key>
    <string>/Users/Jinde/shadowsocks</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
```

注意正确配置`ProgramArguments`和`WorkingDirectory`。`WorkingDirectory`为Shadowsocks配置文件所在目录。之后每次重启，`sslocal`都会自动运行。完毕。
