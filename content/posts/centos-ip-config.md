---
title: "CentOS 新建虚拟机 IP 设置"
date: 2020-12-06T21:02:00+08:00
draft: false
tags: ["CentOS", "Linux"]
slug: "centos-ip-config"
---

VMware vSphere CentOS 7 虚拟机安装完成后，默认配置文件没有激活网络接口，且没有相关 IP 设置，若需要以静态 IP 方式联网，则需手动进行相关设置。

默认情况：

![ping baidu](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102144628.png)
![yum upgrade](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102144639.png)

目录：

- [设置方法](#设置方法)
- [参考链接](#参考链接)

## 设置方法

**Step 1.** 修改 /etc/sysconfig/network-scripts/ifcfg-ens... 文件，其中 ens 后面的数字编号可能不同。

 > \# 修改
 > BOOTPROTO=static
 > ONBOOT=yes
 >
 > \# 新增
 > IPADDR=10.245.150.192
 > GATEWAY=10.245.150.1
 > DNS1=202.102.154.3

 参数说明：

参数      | 值                                                                                                                                      | 说明
----------|-----------------------------------------------------------------------------------------------------------------------------------------|----------
BOOTPROTO | (a) none：不使用启动地址协议，设置为 none 禁止DHCP <br> (b) bootp：BOOTP 协议 <br> (c) dhcp：DHCP 动态地址协议 <br> (d) static：静态地址协议 | 系统启动地址协议
ONBOOT    | (a) yes：系统启动时激活该网络接口 <br> (b) no：系统启动时不激活该网络接口                                                                 | 系统启动时是否激活
IPADDR    | 10.245.150.192                                                                                                                          | IP地址
GATEWAY   | 10.245.150.1                                                                                                                            | 网关地址
DNS{1, 2} | (a) 202.102.154.3 <br> (b) 9.9.9.9 <br> (c) ......                                                                                      | DNS地址

**Step 2.** 重启服务

```bash
systemctl restart network
```

## 参考链接

* [ifcfg 配置文件](https://blog.csdn.net/u011857683/article/details/80950466)
