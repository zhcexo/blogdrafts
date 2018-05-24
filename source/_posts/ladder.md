---
title: 记一次搭梯子的过程
date: 2018-05-24 17:21:32
tags: server
---
之前一直用的 Japan 的线路，结果越来越不稳定，后来干脆变成不可用了，所以换了一家 VPS 换了一条线路，以此文章记录一下这次的过程，方便以后折腾。

服务器安装的是 **ubuntu 18 LTS x64 版本**，准备使用 55 和 kcp 当梯子。

## 步骤：(以下步骤默认都是 root 下执行)

### 1. 安装 python-pip 和 55

使用两个命令即可：

```shell
apt-get install python-pip
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```

执行第二个命令如果报错，例如 `ImportError: No module named setuptools`，只需要再安装 setuptools 即可。

先查看自己服务器的 Python 版本：

> 1. 在终端上输入 python，进入 python shell
> 2. 输入 help() 查看 python 版本
> 3. 查看完毕后，输入 exit() 退出 python shell

接下来安装 setuptools：

```shell
wget https://bootstrap.pypa.io/ez_setup.py -O - | sudo python
# 或者
wget https://bootstrap.pypa.io/ez_setup.py -O - | sudo python3.4
```

### 2. 配置和启用 55

安装完之后，在 `/etc/` 目录下创建配置文件，命名为 `shadowsocks.json`：

```javascript
{
    "server":"0.0.0.0",             // your server ip goes here
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"**********",        // your ss password goes here
    "timeout":300,
    "method":"aes-256-gcm",
    "fast_open": false
}
```

启动和停止 55 的方法是：

```shell
ssserver -c /etc/shadowsocks.json -d start      # 这是后台启动的方式
ssserver -c /etc/shadowsocks.json -d stop       # 这是对应的停止的方式
```

### 3. 安装 kcptun

直接访问项目地址吧，安装过程不说了，[https://github.com/kuoruan/shell-scripts](https://github.com/kuoruan/shell-scripts)。

安装过程中可能会提示 `iptables` 相关的错误，原因是 `iptables` 在 centos 和 ubuntu 上有差异。脚本似乎是针对 centos 写的，不过无所谓，脚本里面已经提示了自己去解决 `iptables` 问题，但直接按提示去解决会出错。

解决办法无外乎两步：首先添加相应规则到 `iptables`，然后重启 `iptables` 服务。

添加 `iptables` 配置
```shell
iptables -I INPUT -p udp --dport 29900 -j ACCEPT        # 29900 换成你自己的
```

保存 `iptables` 的配置（ubuntu 下）
```shell
iptables-save
```

重启 `iptables`，因为 ubuntu 用的是 `ufw` 作为 `iptables` 的前端，所以使用如下命令重启：
```shell
service ufw restart
```

### 4. 注意

在配置 kcptun 的时候，加速的 IP 填写外网的 IP，如果填本地 IP（127.0.0.1 或者 0.0.0.0）都会出来 `dial tcp 127.0.0.1:8388: connect: connection refused` 这种错误。

### 5. 参考资料

[https://blog.kuoruan.com/110.html](https://blog.kuoruan.com/110.html)