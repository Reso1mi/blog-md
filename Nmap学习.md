---
title: Nmap 学习
tags:
  - Kali
  - Nmap
date: 2018/6/17
categories:
  - 渗透
abbrlink: d5f81032
---

## Nmap 简介
   Nmap 是一个开源免费的网络发现和安全审计工具 ， 全称是 NetWork  Mapper , 采用 c++语言编写。纯命令行界面，Nmap 官方提供了一个 Zenmap 的图形界面。

- 关于端口
	·	默认情况下，Namp 会扫描 1000 个最有可能开放的 TCP 端口
·   open：开放
·   closed：关闭
·   filtered：被屏蔽
·   unfiltered：没有屏蔽还需要确认
·   open | filtered : 开放或者屏蔽
·   close | filtered : 关闭或屏蔽
# 使用
 - 主机扫描
 1. nmap  192.168.1.100 直接扫描，会扫描常用前 1000 个端口 扫描我的宿主机
2. nmap -F 192.168.1.100  快速扫描，会扫描常用的前 100 个端口
3. -v  显示扫描过程
root@kali:~# nmap -v -F  192.168.1.100
Starting Nmap 7.70 ( https://nmap.org ) at 2018-06-17 00:55 EDT
Initiating Ping Scan at 00:55
Scanning 192.168.1.100 [4 ports]
Completed Ping Scan at 00:55, 0.05s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 00:55
Completed Parallel DNS resolution of 1 host. at 00:55, 0.01s elapsed
Initiating SYN Stealth Scan at 00:55
Scanning 192.168.1.100 [100 ports]
Discovered open port 135/tcp on 192.168.1.100
Discovered open port 139/tcp on 192.168.1.100
Discovered open port 445/tcp on 192.168.1.100
Discovered open port 3306/tcp on 192.168.1.100
Discovered open port 443/tcp on 192.168.1.100
Increasing send delay for 192.168.1.100 from 0 to 5 due to 43 out of 142 dropped probes since last increase.
Completed SYN Stealth Scan at 00:55, 7.68s elapsed (100 total ports)
Nmap scan report for 192.168.1.100
Host is up (1.8s latency).
Not shown: 94 closed ports
PORT     STATE    SERVICE
135/tcp  open     msrpc
139/tcp  open     netbios-ssn
443/tcp  open     https
445/tcp  open     microsoft-ds
514/tcp  filtered shell
3306/tcp open     mysql
Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 7.88 seconds
Raw packets sent: 166 (7.280KB) | Rcvd: 163 (6.688KB)
4. -sV  返回端口对应的服务信息（好慢）
5. -O  返回对应服务的系统信息
- 主机发现
	我们可以通过 namp 来嗅探整个局域网，扫描出局域网中在线的主机，通过 ICMP ECHO 扫描出在线的主机 （ping 的底层原理） 
  1. nmap -sS 192.168.1.100 隐蔽扫描 -sS ，也就是 SYN 扫描，只管发送数据包
  2. 查看存活主机
nmap -sP 192.168.239.* 或者 192.168.239.0/24
  3. 扫描主机的所有端口
nmap -p 1-65535 192.168.239.133
  4. 扫描主机的操作系统
nmap -O 192.168.239.133
  5. 查看主机个服务的版本详细信息
nmap -sV 192.168.239.133
- 常见的 Nmap 扫描类型参数
 -sT :TCP connect 扫描 ，类似 Metasploit 中的 tcp 扫描
 -sS : TCP SYN 扫描，类似于 Metasploit 中的 syn 扫描模块
 -sF/-sX/-sN : 这些扫描通过发送一些特殊的标志位以避开设备或软件的监测
 -sP : 通过发送 ICMP echo 请求探测主机是否存货原理同 Ping.
 -sU : 探测主机开放了那些 UDP 端口。
 -sA : TCP ACK 扫描，类似与 Metasploit 里的 ack 扫描模块
 - 常见的 Nmap 扫描选项
-Pn : 在扫描之前不发送 ICMP echo 请求测试目标是否活跃
-O ：启用对于 TCP/IP 协议栈的特征扫描以获取远程主机的操作系统
-F ：快速扫描
-p ：端口范围
