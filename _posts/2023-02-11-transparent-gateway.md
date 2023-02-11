---
layout: post
title: Transparent Gateway
date: '2023-02-11T20:43:00+08:00'
tags: [iptables,docker]
categories: network
comments: true
---

#### 软硬件
N1,NDM(aarch64 GNU/Linux)
#### 创建网络,网卡混杂模式
```bash
ip link set eth0 promisc on
docker network create -d macvlan --subnet=2.2.2.0/24 --gateway=2.2.2.1 -o parent=eth0 macnet
```
#### 转发用clash redir
```bash
docker run -d \
-it \
--name=clash \
--network macnet \
--ip 2.2.2.77 \
--mac-address 32:D8:E2:AE:93:77 \
--ulimit nofile=16384:65536  \
--cap-add=NET_ADMIN \
--privileged=true \
-e TZ=Asia/Shanghai \
-p 7892:7892 \
golang:latest \
/bin/sh
```
#### 安装,配置clash
```bash
go install github.com/Dreamacro/clash@latest
cd bin/
vi config.yaml
./clash -f config.yaml
```
#### iptables规则
```bash
#来自 https://github.com/shadowsocks/shadowsocks-libev#transparent-proxy

# Create chain
iptables -t nat -N WENDESDAY
iptables -t mangle -N WENDESDAY
# Ignore server's addresses
iptables -t nat -A WENDESDAY -d 54.250.148.164 -j RETURN
# Ignore LANs addresses,bypass the proxy
iptables -t nat -A WENDESDAY -d 0.0.0.0/8 -j RETURN
iptables -t nat -A WENDESDAY -d 10.0.0.0/8 -j RETURN
iptables -t nat -A WENDESDAY -d 127.0.0.0/8 -j RETURN
iptables -t nat -A WENDESDAY -d 169.254.0.0/16 -j RETURN
iptables -t nat -A WENDESDAY -d 172.16.0.0/12 -j RETURN
iptables -t nat -A WENDESDAY -d 192.168.0.0/16 -j RETURN
iptables -t nat -A WENDESDAY -d 224.0.0.0/4 -j RETURN
iptables -t nat -A WENDESDAY -d 240.0.0.0/4 -j RETURN
iptables -t nat -A WENDESDAY -d 2.2.2.0/24 -j RETURN
# TCP
iptables -t nat -A WENDESDAY -p tcp -j REDIRECT --to-ports 7892
# UDP
ip route add local default dev lo table 100
ip rule add fwmark 1 lookup 100
iptables -t mangle -A WENDESDAY -p udp --dport 53 -j TPROXY --on-port 7892 --tproxy-mark 0x01/0x01
# Apply the rules
iptables -t nat -A PREROUTING -p tcp -j WENDESDAY
iptables -t mangle -A PREROUTING -j WENDESDAY
# NAT
iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
```
Others that may be used
```bash
# Packages
apt update
apt install -y vim iptables iproute2 iputils-ping curl net-tools ca-certificates
update-ca-certificates

# Temporary proxy server
export http_proxy=http://2.2.2.77:7892
export https_proxy=http://2.2.2.77:7892
unset http_proxy
unset https_proxy

# Test Gateway
curl https://ipinfo.io/
curl https://ifconfig.co/
```