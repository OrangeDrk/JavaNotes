[TOC]

# 桥接

1. 优点：同一个局域网中的任意一台物理机想要访问虚拟机时，只要拥有账户和密码，就可以直接进行通信。
2. 缺点：如果宿主主机没有连接网络，那么虚拟机也就不存在与该真实网络环境中，换句话，虚拟机使用桥接模式的时候，它的网络依赖于宿主的网络环境。

![](https://note.youdao.com/yws/api/personal/file/F7AB1536CBB24D2EA4F1C21695E613AF?method=download&shareKey=43760dbe883a8ff8961a13312e7e91a3)



# NAT

1. 优点：可以无视物理机(宿主主机)网络环境。即便是物理机没有网络，也不影响本机和虚拟机进行通信，也不影响本机上的其他虚拟机之间互相通信。因为虚拟机真正通信网卡是VMNet8提供(网络环境)
2. 缺点：其他物理机想要访问NAT模式下的虚拟机时，比较麻烦。

![](https://note.youdao.com/yws/api/personal/file/73C84D2972C74F6AA0CDA79D5711E1B1?method=download&shareKey=6835f483ed7f65b7963c7c07ff7fa318)