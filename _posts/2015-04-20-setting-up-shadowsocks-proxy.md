---
layout: post
title: "Setting up shadowsocks proxy to visit Google FB and etc"
date: 2015-04-20 20:10:33 +0800
categories: [Network]
tags: [shadowsocks]

---

***(Considering the Wall's ability to block keyword, this column will evade sensitive words and be writen in English.)***

You know, we are often stuck in the life, even in the Internet.
As a heavy Internet user, I need to visit Google, Android Developer and Github sites by high frequency. But these sites cannot be accessed normally in my country.
After comparing the advantages and disadvantages of many solutions, I choose to set up shadowsocks proxy on foreign server.
I will share my experience on passing each stage. Hope it works for you.

##	Catalogue
---

*	[Wiki](#Wiki)
*	[Server Side Install](#Server Side Install)
*	[Configure and start the service](#Configure and start the service)
*	[Optional: chacha20 encryption](#Optional: chacha20 encryption)
*	[Optional: Optimizing shadowsocks](#Optional: Optimizing shadowsocks)
*	[Optional: Optional: Zeta-TCP](#Optional: Optional: Zeta-TCP)
*	[Clients on windows](#Clients on windows)

## Wiki
---

A foreign server is required firstly.  I choosed a VPS which DigitalOcean server is located at San Francisco.
Recommended VPS:
[DigitalOcean](https://www.digitalocean.com/?refcode=4cac58a4d0a0) Provide cloud hosting in the US and Singapore. Default Support for TCP Hybla. Low-cost, take Paypal.
[Linode](https://www.linode.com/) Provide cloud hosting in Japan and the US, High Bandwidth.
[Bandwagonhost](https://shorturl.at/OiOTH) BandwagonHost is a popular VPS (Virtual Private Server) provider owned by the Canadian company IT7 Networks Inc.

**Shadowsocks-libev** is a lightweight secured socks5 proxy for embedded devices and low end boxes.
You can visit official site at [github.com/shadowsocks/shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev) to see more introduction.

**Cow** is HTTP proxy and written in Go. It can automatically identify blocked sites and use parent proxies to access.
Official website: [github.com/cyfdecyf/cow](https://github.com/cyfdecyf/cow)

**TCP Hybla** is a TCP enhancement for heterogeneous networks.

## Server Side Install
---

###	Debian & Ubuntu
Build package from source

	apt-get install build-essential autoconf libtool libssl-dev gawk debhelper git
	git clone https://github.com/madeye/shadowsocks-libev.git
	cd shadowsocks-libev
	dpkg-buildpackage -us -uc

Install `shadowsocks-libev`

	cd ..
	dpkg -i shadowsocks-libev*.deb

Install `supervisor`

	apt-get install supervisor

## Configure and start the service
---

Edit the `shadowsocks-libev` configuration

	vi /etc/shadowsocks-libev/config.json

With the following content:

```
{
     "server":"0.0.0.0",
     "server_port":443,
     "password":"password",
     "method":"aes-256-cfb",
     "timeout":300
}
```

Edit the `supervisor` configuration

	vi /etc/supervisor/conf.d/shadowsocks-libev.conf

With the following content:

```
[program:shadowsocks-libev]
command=ss-server -c /etc/shadowsocks-libev/config.json -u
autorestart=true
user=nobody
```

If  the `server_Port` < 1024 in `config.json`, change `user=nobody` to `user=root`.

Restart the `supervisor `

	service supervisor restart
	supervisorctl reload

After edit any files on `/etc/supervisor/*`, update `supervisor` configuration:

	supervisorctl update

Make sure if `shadowsocks-libev` is listening to the correct port:

	netstat -lnp

## Optional: chacha20 encryption
---

Modify `"method":"aes-256-cfb"` to `"method":"chacha20"` and save, like this:

```
{
     "server":"0.0.0.0",
     "server_port":443,
     "password":"password",
     "method":"chacha20",
     "timeout":300
}
```

Install required libs:

```bash
wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz

tar zxf LATEST.tar.gz
cd libsodium*
./configure
make && make install
```

Optimizing encryption  speed:

    apt-get install python-m2crypto

Reboot shadowsocks service.

## Optional: Optimizing shadowsocks
---

###	Debian & Ubuntu

Create /etc/sysctl.d/local.conf with the following content:

```
# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for high-latency network
net.ipv4.tcp_congestion_control = hybla

# for low-latency network, use cubic instead
# net.ipv4.tcp_congestion_control = cubic
```

Then:

	sysctl --system

**Warning: DO NOT ENABLE `net.ipv4.tcp_tw_recycle`!!!**

If you use `Supervisor`, make sure you have the following line in `/etc/default/supervisor`:

	ulimit -n 51200

Once you added that line, restart `Supervisor`

	service stop supervisor && service start supervisor

See more info at the Sourse: [Optimizing-Shadowsocks](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks)

## Optional: Zeta-TCP
---

Now, Serverspeeder provid a free Zeta-TCP service. We can deploy it on our server to  improve the end-to-end performance of TCP.

Install

	wget http://my.serverspeeder.com/d/ls/serverSpeederInstaller.tar.gz
	tar xzvf serverSpeederInstaller.tar.gz
	bash serverSpeederInstaller.sh

Input your Serverspeeder account and press `Enter` to keep default for others settings during the installation process.
![](/img/setting-up-ss-proxy/serverspeeder.png)

Some configuration

	vi /serverspeeder/etc/config


rsc="1", RSC 网卡驱动模式
advinacc="1", 流量方向加速
maxmode="1", 最大传输模式

Restart service

	/serverspeeder/bin/serverSpeeder.sh stop
	/serverspeeder/bin/serverSpeeder.sh start

## Clients on windows
---

Cow: [X32](http://dl.chenyufei.info/cow/cow-win32-0.9.4.zip),[X64](http://dl.chenyufei.info/cow/cow-win64-0.9.4.zip)

Edit `rc.txt (Windows)` with the following content:

```
# 本地 HTTP 代理地址
# 配置 HTTP 和 HTTPS 代理时请填入该地址
# 或者在自动代理配置中填入 http://127.0.0.1:7777/pac
listen = http://127.0.0.1:7777

# shadowsocks 二级代理
proxy = ss://aes-256-cfb:password@1.2.3.4:443
```

Then, open `Internet Explorer`, click on `Tools`, `Internet Options`.
After seeing the `Internet Options` window, click on `Connections`, `LAN settings`.
Checked `Use automatic configuration script`, fill in `Address` box with the following content:

	http://127.0.0.1:7777/pac

Peference picture
![](/img/setting-up-ss-proxy/IE-PAC-proxy.jpg)

Closed.
