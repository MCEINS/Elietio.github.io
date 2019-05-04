---
title: 自建Shadowsocks频繁被封？赶紧试试simple-obfs流量混淆吧
date: 2019-04-28 22:14:49
tags: [科学上网,Shadowsocks,simple-obfs]
categories: [技术]
mathjax: true
copyright: true
comment: true
photo: 
---

年后用的一直挺稳的搬瓦工VPS搭建的Shadowsocks端口频繁被封，换了几次端口后感觉太麻烦，于是决定网上找找有没有好的思路，于是发现了[simple-obfs](https://github.com/shadowsocks/simple-obfs),其思路是ss客户端对Shadowsocks流量混淆加密伪装成正常http或者https流量，服务端ss-server再对流量解密，以此躲过GFW的封杀。这个方案挺有意思，很多人又再此基础上进行了更多的伪装扩展，稳定性比之前纯Shadowsocks流加密更安全。不过原作者已经不再维护此项目而是转入了[v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin)，由于本人使用的路由器SS固件只支持simple-obfs插件模式，因此决定还是先试试simple-obfs的效果。
<!-- more -->
需要：
- [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)
- [simple-obfs](https://github.com/shadowsocks/simple-obfs)
- VPS
- Nginx
- 支持simple-obfs插件模式的ss客户端



### 安装

首先安装shadowsock，使用[shadowsocks-libev](shadowsocks-libev),此项目支持插件模式使用[simple-obfs](https://github.com/shadowsocks/simple-obfs)，具体部署可以查看github主页。当然如果图省事也可以使用别人的一键安装脚本，例如[shadowsocks_install](https://github.com/teddysun/shadowsocks_install),

```shell
wget --no-check-certificate -O shadowsocks-all.sh <https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh>
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

输入具体参数一路向下即可。

### 配置

#### ss-server： 

最后生成的`/etc/shadowsocks-libev/config.json`配置如下格式。

```json
{                            
    "server":"0.0.0.0",
    "server_port":port,
    "password":"password",
    "timeout":300,
    "user":"nobody",
    "method":"chacha20-ietf-poly1305",
    "fast_open":false,
    "nameserver":"8.8.8.8",
    "mode":"tcp_and_udp",
    "plugin":"obfs-server",
    "plugin_opts":"obfs=http"
} 
```

- `method`:加密方式，推荐使用`AEAD`算法，比如`chacha20`，`aes-256-cfb` 和 `rc4-md5`已经不再安全,很容易识别。
- `plugin`：使用`obfs-server`插件 
- `plugin_opts`：混淆参数，obfs 有 `tls` 和 `http` 两种类型，根据你混淆的类型设置吧，和后面要保持一致

#### Nginx：

之前我们没启用 simple-obfs，ss-server 和ss-local直接进行数据交换，很容易别识别出来。现在ss-local通过simple-obfs伪装，ss-server 再拿掉这层伪装进行数据交换，这样可靠性就得到提升，我们的目的是让ss-local的流量更像正常的http流量，我们可以在VPS上80端口和443端口部署一个静态站点，并申请一个域名，再用Nginx再做一层反向代理，正常流量直接访问静态站点，而ss-local进行simple-obfs混淆域名的流量则代理到ss-server上去，这样从外表上看我们是去访问VPS上部署的网站，而实际却是和ss-local进行数据交互。

nginx配置参考如下，端口用80、443、8080等常用端口更符合http逻辑。

```nginx
server
{
    listen 80;
    server_name your domain;
    index index.html index.htm index.php default.html default.htm default.php;
    charset utf-8;
        
    location / {
      root  /home/wwwroot/...;
      autoindex on;
    }
    
    location =/ {
      root  /home/wwwroot/...;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      if ($http_upgrade = "websocket") {
        proxy_pass http://127.0.0.1:ss-local port;
      }
    }    
    access_log off;
}
```

好了，服务端的部署到此差不多结束了，下面是客户端的一些配置

#### Windows:

[shadowsocks-windows](<https://github.com/shadowsocks/shadowsocks-windows>)需要安装[obfs-local](<https://github.com/shadowsocks/simple-obfs/releases>)插件，并引入配置中

```json
{
      "server": "your domain",
      "server_port": 80,
      "password": "your password",
      "method": "chacha20-ietf-poly1305",
      "plugin": "obfs-local",
      "plugin_opts": "obfs=http;obfs-host=your domain",
      "plugin_args": "",
      "remarks": "",
      "timeout": 5
}
```

#### Android：

[GitHub](https://github.com/shadowsocks/simple-obfs-android/releases) | [Google Play](https://play.google.com/store/apps/details?id=com.github.shadowsocks.plugin.obfs_local)

下载混淆插件，Shadowsocks Android 客户端配置文件下选取 `simple-obfs` 插件，配置项：

```json
Obfuscation wrappper: http
Obfuscation hostname: your domain
```

#### Padavan:

如果你的路由器和我一样使用Padavan，可以在ss配置页面设置如下

- `插件名称`：`obfs-local`
- `插件参数`：`obfs=http;obfs-host=your domain`

好了，到此整个从客户端到服务端的流量混淆加密完成，开始体验新的模式的吧，个人感觉还是挺稳的。

### shadowsocks-libev相关命令

```shell
启动：/etc/init.d/shadowsocks-libev start
停止：/etc/init.d/shadowsocks-libev stop
重启：/etc/init.d/shadowsocks-libev restart
状态：/etc/init.d/shadowsocks-libev status
     systemctl status shadowsocks-libev
```

### 开启BBR加速

 BBR 是Google  TCP拥塞控制算法，能最大化利用网络上瓶颈链路的带宽，降低网络链路上的 buffer 占用率，从而降低延迟，实际测试对网速提升很大。从 4.9 开始，Linux 内核已经用上了该算法。

#### 安装：

推荐使用[一键脚本](<https://teddysun.com/489.html>)

```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```

安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。

重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令：

```bash
$ uname -r
4.9.0-3-amd64
```

查看内核版本，显示为最新版就表示 OK 了

#### 一些检测命令:

```bash
$ sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = bbr cubic reno
```
```bash
$ sysctl net.ipv4.tcp_congestion_control
net.ipv4.tcp_congestion_control = bbr
```

```bash
$ sysctl net.core.default_qdisc
net.core.default_qdisc = fq
```

```bash
$ lsmod | grep bbr
tcp_bbr                16384  28
```
