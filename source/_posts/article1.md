---
title: 通过frp搭建自己的内网穿透服务
date: 2024-06-24 11:48:44
categories:
  - 运维
tags: 
  - frp
  - 内网穿透
---

本文介绍如何通过frp在自己的云服务器上搭建内网穿透服务，提供对外暴露的web服务。

内网穿透可以将本机的服务通过云服务器拥有的公有ip暴露到公网上供他人访问，应用场景有很多，比如做微信小程序开发时需要调试接口，或者向别人演示自己还未正式上线的小demo等。

在阅读本文内容之前，需要有一些前置准备：

- 你拥有一个域名
- 你拥有一台具有公网ip地址的云主机

<!-- more -->

## frp

frp 是一款高性能的反向代理应用，专注于内网穿透。它支持多种协议，包括 TCP、UDP、HTTP、HTTPS 等，并且具备 P2P 通信功能。使用 frp，您可以安全、便捷地将内网服务暴露到公网，通过拥有公网 IP 的节点进行中转。

## 下载

如果你是win系统，可以通过frp的[github releases](https://github.com/fatedier/frp/releases)点击下载文件。

如果你是linux系统，你可以通过wget下载:

```sh
wget https://github.com/fatedier/frp/releases/download/v0.58.1/frp_0.58.1_linux_amd64.tar.gz
```

这里以win为例，下载完压缩包解压之后，你会看到如下的目录结构：

```txt
--frp_0.58.1_windows_amd64
|-----frpc.toml
|-----frpc.exe
|-----frps.exe
|-----frps.toml
|-----LICENSE
```

这里介绍一下，这个frpc.toml是客户端的配置文件，客户端指的是你需要进行内网穿透的机器。c就是client的意思。frpc.exe就是客户端的frp程序的启动文件。

frps.toml就是源服务器的配置文件，源服务器指的就是你的云服务器（拥有公网ip的）。同理，这个s就是source的意思。

那其实我们现在用的win机器，只需要保留frpc.toml和frpc.exe就行了，剩下的frps.toml和frps.exe可以删除掉。因为云服务器使用linux系统这个exe他也用不了，所以我们在云服务器上还需要从giithub上下载这个frp的文件。

同样的，在云服务器上下载了文件后，也可以删除掉frpc.toml和frpc，只需要保留frps.toml和frps即可。

接下来介绍具体操作步骤：

### 对于客户端（win机器）

1. 通过[github releases](https://github.com/fatedier/frp/releases)下载文件

2. 保留文件中的`frpc.toml`和`frpc.exe`即可

3. 配置`frpc.toml`
4. 启动frpc服务

### 对于云主机（linux机器）

1. 通过sh命令获取frp文件

   ```sh
   wget https://github.com/fatedier/frp/releases/download/v0.58.1/frp_0.58.1_linux_amd64.tar.gz
   ```

2. 保留文件中的`frps.toml`和`frps`即可
3. 配置`frps.toml`
4. 启动frps服务

如果你不想每次都进入frp安装目录下通过 `xxx/frps -c xxx/frps.toml` 来启动的话，可以通过`systemd`来添加frp服务，通过`systemctl`统一管理。具体步骤如下：

1. 在`/lib/systemd/system/`下创建 `frps.service`文件, 内容如下：

   ```sh
   [Unit]
   # 服务名称，可自定义
   Description = frp server
   After = network.target syslog.target
   Wants = network.target
   
   [Service]
   Type = simple
   # 启动frps的命令，需修改为您的frps的安装路径
   ExecStart = /path/to/frps -c /path/to/frps.toml
   
   [Install]
   WantedBy = multi-user.target
   ```

2. 通过systemd管理frps

   ```sh
   # 设置自启动
   sudo systemctl enable frps
   # 启动frp
   sudo systemctl start frps
   # 停止frp
   sudo systemctl stop frps
   # 重启frp
   sudo systemctl restart frps
   # 查看frp状态
   sudo systemctl status frps
   ```

## 对外提供web服务

1. 在云主机上配置frps.toml

   ```sh
   bindPort = 7000
   vhostHTTPPort = 8080
   ```

2. 在win机器（客户端）上配置frpc.toml

   ```sh
   serverAddr = "x.x.x.x"  <-云主机公网ip地址
   serverPort = 7000
   
   [[proxies]]
   name = "web"
   type = "http"
   localPort = 80   <-暴露本机80端口的服务
   customDomains = ["www.yourdomain.com"]
   
   [[proxies]]
   name = "web2"
   type = "http"
   localPort = 8080  <-暴露本机8080端口的服务
   customDomains = ["www.yourdomain2.com"]
   ```

   > 记得把自己的域名解析到云主机上！

3. 启动服务

   ```sh
   to/path/frps -c frps.toml
   ```

   或者 通过systemd

​	win机器同理，通过powershell或者cmd执行exe程序并通过 `-c` 指定应用的配置文件。

至此，内网穿透已经实现，你可以愉快的向别人分享你的demo啦！😊

---

结尾附上[frp的官方文档](https://gofrp.org/zh-cn/docs/)，本文还有些不完善的地方或者没有提到的功能，可以参考frp官方文档。

