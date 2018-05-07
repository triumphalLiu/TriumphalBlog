---
title: SSH高延迟解决方案-Mosh的安装与使用
tags: 服务器
abbrlink: 64562
date: 2018-05-07 17:26:34
---

## 目的
由于远程服务器架设在国外，使用ssh连接的远程终端延迟极高、卡顿、失去连接。  

找到两种解决方案：
- 用putty(windows)的Local Echo和Local Line Editing功能，会导致tab无法使用，且vim的时候不正常。
- 用Mosh，能有效解决问题，且Tab之类的正常使用。

<!--More-->

## Mosh的安装与使用
**AWS用户需要先设置入站规则，允许自定义UDP端口或60001(默认端口)的UDP入站流量**
### 安装Mosh
``` bash
$ sudo apt-get update
$ sudo apt-get install mosh
$ mosh --version
mosh 1.2.5 [build mosh 1.2.5]
```
### 使用Mosh连接
#### 直接连接
``` bash
$ mosh User@IPAddress
```

#### SSH方式连接
``` bash
//用法 mosh --ssh="SSH认证命令" -p 指定端口（不填默认60001） 用户名@主机名
$ mosh --ssh="ssh -i aws.pem" -p 1234 ubuntu@ec2-1-1-1-1.us-west-2.compute.amazonaws.com
```
#### 一次性Key临时连接
服务器端
``` bash
ubuntu@ip-1-1-1-1:~$ mosh-server 

MOSH CONNECT 60001 uwApZWj7a+lH5lwVrfAPXA         <-------临时Key
ubuntu@ip-1-1-1-1:~$ 
mosh-server (mosh 1.2.5) [build mosh 1.2.5]
Copyright 2012 Keith Winstein <mosh-devel@mit.edu>
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

[mosh-server detached, pid = 17136]
```
本地主机
``` bash
$ sudo vi /etc/profile
添加一条：export MOSH_KEY=uwApZWj7a+lH5lwVrfAPXA
$ source /etc/profile
$ mosh-client Username@IPAddress 60001
```

#### Addition：偷懒的小方法
不想每次都输入命令可用：
``` bash
$ vi mosh-connect.sh    //在其中加入登录命令
$ chmod a+x mosh-connect.sh
$ ./mosh-connect.sh
```

## IPTABLES操作  
### 防火墙拦截时在服务器端增加防火墙规则
``` bash
$ sudo iptables -I INPUT -p udp --dport [指定端口] -j ACCEPT  (新建规则)
OR
$ sudo iptables -A INPUT -p udp --dport [指定端口] -j ACCEPT  (追加规则)
```
### 查看防火墙规则
``` bash
$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     udp  --  anywhere             anywhere             udp dpt:60001

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
### 删除防火墙规则
``` bash
$ sudo iptables -D INPUT 1 //删除input里第一条规则
```

