[TOC]

# 启动网卡

> 安装完成之后，系统启动时，默认不会启动网络连接，需要手动开启。

## 查看网卡信息

> eth0为网卡，第一次启动会发现，`addr：`是空的，没有ip地址，需要启动网卡

```shell
[root@Demo01 ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:16:72:6A  
          inet addr:  Bcast:192.168.72.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe16:726a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3660 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2774 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:577799 (564.2 KiB)  TX bytes:298332 (291.3 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:12 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:720 (720.0 b)  TX bytes:720 (720.0 b)
```



## 启动网卡

> 表示启动CentOS系统中名为【eth0】的网卡。

```shell
[root@Demo01 ~]# ifup eth0
```





# 设置网卡开机自启

> 修改`/etc/sysconfig/network-scripts/ifcfg-eth0`文件
>
> - `ONBOOT=no` ==> `ONBOOT=yes`
>
> 注意：
>
> - 按键盘`i`进入编辑模式
> - 按键盘`ESC`，退出编辑模式。输入`:wq`保存退出；`q!`不保存强制退出
>
> 介绍：
>
> | `vi`              | linux系统内核自带的文本编辑器                                |
> | ----------------- | ------------------------------------------------------------ |
> | `etc`             | Linux系统中所有的配置文件存放目录                            |
> | `sysconfig`       | 系统配置文件的存放目录                                       |
> | `network-scripts` | 网络配置文件的存放目录                                       |
> | `ifcfg-eth0`      | 具体的网卡配置文件<br />`ifconfig`:用来查看当前系统的网络连接，类似于Windows的`ipconfig` |

```shell
[root@Demo01 ~]# vi /etc/sysconfig/network-scripts/ifcfg-eth0
```



# 设置静态IP

> 为什么要设置一个静态IP：
>
> - 服务器拥有一个静态IP是因为方便客户端的访问和提供服务，企业中的所有服务器都是一个固定IP(静态IP)地址。

## 通过`setup`命令便捷设置

```shell
[root@Demo01 ~]# setup
```

## 操作步骤

> 1. 使用方向键将光标移动到"使用DHCP"，敲击"空格"取消它的勾选。
> 2. 静态IP：与服务器当前所使用的IP地址保持相同（192.168.89.128）
> 3. 子网掩码：255.255.255.0
> 4. 网关IP：192.168.89.2 ==与静态IP地址的前三段保持一致==，最后一位改为2。
> 5. 两个DNS：
>    - 114.114.114.114
>    - 8.8.8.8
> 6. 一路保存到最后

![](https://gitee.com/sxhDrk/images/raw/master/imgs/设置静态IP流程图.png)



## 重启网络服务

> 正常重启网络服务会出现如下内容
>
> ```shell
> [root@Demo01 ~]# service network restart
> 正在关闭接口 eth0： 设备状态：3 (断开连接)				    [确定]
> 关闭环回接口：                                             [确定]
> 弹出环回接口：                                             [确定]
> 弹出界面 eth0： 活跃连接状态：激活的
> 活跃连接路径：/org/freedesktop/NetworkManager/ActiveConnection/1
>                                                            [确定]
> ```

```shell
[root@Demo01 ~]# service network restart

##或者：
[root@Demo01 ~]# /etc/init.d/network restart
```