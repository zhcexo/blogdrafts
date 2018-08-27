---
title: 安装Simple Obfs
date: 2018-08-16 13:34:45
tags: server
---
非常不爽，不知道最近 WALL 是做了什么升级，我的 55 挂了，使用 chacha20-ietf-poly1305 都不行，之前还用着挺好的。无奈只好又寻方法，找到了 Simple Obfs。

## 服务端配置

**以下命令均在 root 权限下执行，所以无 sudo**

废话不多说，先安装：

```bash
apt-get install --no-install-recommends build-essential autoconf libtool libssl-dev libpcre3-dev libev-dev asciidoc xmlto automake

git clone https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
git submodule update --init --recursive
./autogen.sh
./configure && make
sudo make install
```

使用 Simple Obfs 有两种方式，一种是作为 55 的插件，另一种是独立模式，我怕麻烦，所以使用作为 55 插件的方法。以前用的是 Python 版的 55，不支持外挂插件的样子，所以换成 55-libev。安装之：

```bash
apt-get update
apt-get install shadowsocks-libev
```

如果安装成功了，在命令行下输入 `ss-server` 会出现相应提示的，这一步不详说。（要注意的是，Python 版的命令是 `ssserver`，55-libev 的命令是 `ss-server`，有一横杠的区别）

接下来修改 55 的配置文件，在配置文件里，添加两行参数：

```javascript
"plugin": "obfs-server",
"plugin_opts": "obfs=http"
```

万事具备，使用如下命令启动 55：

```bash
ss-server -c config.json
```

如果输出没有报错，就表明启动正常，接下来按 `Ctrl+C` 结束掉。为啥？当然是重新开始后台启动啊：

```bash
nohup ss-server -c config.json > /dev/null 2>&1 &
```

注释：

- `/dev/null` 代表空设备文件
- `>` 表示重定向到哪里，例如： `echo "123" > /home/123.txt`
- `2` 表示 stderr 标准错误
- `&` 表示等同于的意思，表示 2 的输出重定向等同于 1
- `1` 表示 stdout 标准输出，系统默认是 1，所以 `> /dev/null` 等同于 `1 > /dev/null`
- `> /dev/null` 表示标准输出重定向到空设备文件，也就是不输出任何信息到终端
- `2>&1` 接着，标准错误输出重定向（等同于）标准输出，因为之前标准输出已经重定向到了空设备文件，所以标准错误输出也重定向到空设备文件
- `结尾处的 &` 将命令同时放入后台运行

## 客户端配置

下载 obfs-local，**然后把压缩包里面的两个文件，解压到 55 相同的目录里面**。接着打开 55 的服务器配置界面，把 **插件程序** 和 **插件选项** 两个空位置，填写如下配置：

```
插件程序：obfs-local
插件选项：obfs=http;obfs-host=www.bing.com
```

到此为止，全部完成，终于又能用 Google 了~

## 参考资料

- [如何安装和配置simple-obfs服务端](https://teddysun.com/511.html)
- [55-libev](https://blog.csdn.net/wwzuizz/article/details/78194159)
- [配置55-libev服务器端进行KX上网](https://roxhaiy.wordpress.com/2014/07/29/%E5%9C%A8linux%E9%85%8D%E7%BD%AEshadowsocks-libev%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E8%BF%9B%E8%A1%8C%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/)
