---
title: 为nginx配置vhosts（Windows开发环境）
date: 2018-10-29 16:57:36
tags: server
---
公司开发了自己的页面渲染系统，而且涉及需要迁移的站点非常多，所以需要在开发中配置一些虚拟主机。以前玩的都是 Apache，但是后端推荐使用比较轻量的 nginx，所以扒了一下 nginx 的 vhosts 的配置方法。

**首先**，下载 nginx 的可执行文件，是个压缩文件，解压之后，放到一个容易访问的目录，以下用 `nginx_dir` 代替。ngnix 的默认配置文件是 `nginx_dir/conf/nginx.conf`。

**其次**，在 `nginx_dir/conf/` 下新建一个目录，叫 `vhosts`，并且在 `nginx_dir/conf/vhosts/` 下新建一个文件，名为 `vhosts.conf`。

**第三**，在 `nginx_dir/conf/nginx.conf` 的配置里，位于 `http {...}` 的最后面，于 `}` 之前，添加 `include vhosts/*.conf;`。添加的代码意义为：在 `nginx.conf` 中引入 `vhosts` 下所有的 `.conf` 文件。

以下即是 `vhosts.conf` 里面的配置，因为开发中主要用到反向代理，其他的按文档按需添加就行：

```shell
# example 代理相关配置
server {
    # 监听端口号
    listen 7777;
    
    # 反向代理的域名
    server_name example.localhost;
    
    location / {
        #本地模拟服务器地址及端口
        charset utf-8;
        proxy_pass http://127.0.0.1:1987;
        proxy_set_header    Custom-Header 1;
    }
}
```

**第四**，统统设置完之后，打开命令行，定位到 `nginx_dir`，使用如下命令检查配置是否正确：

```bash
$ nginx.exe -t
```

如果出现以下提示，表明配置没问题，出现其他提示自行排查：

```bash
nginx: the configuration file nginx_dir/conf/nginx.conf syntax is ok
nginx: configuration file nginx_dir/conf/nginx.conf test is successful
```

**最后**，因为是在 windows 环境下，所以启动 nginx 使用以下命令：

```bash
$ start nginx.exe
```

使用 `start` 运行而不是直接运行，是让命令行的窗口可以进行别的操作，而不是一直僵在前台。

**补充**：

- 使用 `nginx -s quit` 命令来关闭运行中的 nginx 进程
- 使用 `nginx -s reload` 命令来重启运行中的 nginx，一般用于修改配置文件之后操作
