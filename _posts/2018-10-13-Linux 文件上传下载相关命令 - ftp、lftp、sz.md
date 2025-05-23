---
layout: post
title:  "Linux 文件上传下载相关命令 - ftp、lftp、sz"
date:   2018-09-11 09:25:00 +0800
---
ftp

```
ftp
> open 118.244.227.117 21
> 自动弹出用户名和密码输入

ftp 118.244.227.117 21
> 自动弹出用户名和密码输入
```

sftp

```
sftp -oPort=40020 yinhouben@122.224.137.178
> 确认RSA key
> 自动弹出密码输入
```

lftp（可用于登录FTPS）

```
lftp
> set ftp:ssl-force true
> set ssl:verify-certificate no
> connect 180.186.40.193:10022
> login houben
> 自动弹出密码输入

// ssl:verify-certificate no 选项：if the server is making use of self signed
// certificates.
```

rz/sz（通过本地窗口上传下载文件）

XMODEM, YMODEM, ZMODEM - file transfer protocols over a modem. ZMODEM 是 YMODEM 的改进版，YMODEM 是 XMODEM 的改进版。

lrzsz is a unix communication package providing the XMODEM, YMODEM, ZMODEM file transfer protocols.

sx rx, sb rb and sz rz implement the xmodem, ymodem, and zmodem file transfer protocols respectively.

要使用 sz rz 命令，需要分别在服务端和客户端进行安装并配置。

服务端：
```
yum -y install lrzsz
```
客户端使用 Xshell 的话，默认支持使用这3种传输协议。
Xshell -> 文件 -> 传输也可以使用这3种协议来上传下载。

sz: s means server send 
rz: r means server receive

```
sz file
sz file1 file2 file3
rz


-e 选项，防乱码
```

sz 和 rz 不能传输文件夹，所以可以先打包，再传输。

rsync

```
- Transfer a file over SSH using a different port than the default (22) and show global progress:
rsync -e|--rsh 'ssh -p port' --info=progress2 host:path/to/source path/to/destination
```

