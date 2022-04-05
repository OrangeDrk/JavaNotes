[TOC]

# 1. head可视化页面

## 1.1 下载node.js

> 到node官网下载Linux版本的node

wget https://nodejs.org/dist/v14.16.0/node-v14.16.0-linux-x64.tar.xz

![image-20210317214700491](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210317215150341.png)

解压到`/usr/local`中

```shell
[root@localhost tarfile]# tar xf node-v14.16.0-linux-x64.tar.xz
[root@localhost tarfile]# mv node-v14.16.0-linux-x64 /usr/local/node
```

添加环境变量：`/etc/profile`

```shell
export NODE_HOME=/usr/local/node
export PATH=$PATH:$NODE_HOME/bin

# 刷新环境变量
source /etc/profile
```

查看node版本和npm版本

```shell
[root@localhost local]# node -v
v14.16.0
[root@localhost local]# npm -v
6.14.11
```

> 如果有版本号的出现，说明安装成功

## 1.2 安装cnpm

> 为了拉取依赖更快，安装淘宝的cnpm

```sh
[root@localhost local]# npm install -g cnpm -registry=https://registry.npm.taobao.org
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
npm WARN deprecated har-validator@5.1.5: this library is no longer supported
/usr/local/node/bin/cnpm -> /usr/local/node/lib/node_modules/cnpm/bin/cnpm
+ cnpm@6.1.1
added 689 packages from 973 contributors in 18.05s
```

## 1.3 下载head项目

- 通过wget下载：wget https://github.com/mobz/elasticsearch-head/archive/master.zip
- 或者通过GitHub克隆项目

![image-20210317215150341](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210317215619602.png)

> 解压项目

```shell
[root@localhost plugins]# unzip master.zip 
```

> 进入项目，cnpm安装依赖

```shell
[root@localhost elasticsearch-head-master]# cnpm install

[root@localhost elasticsearch-head-master]# cnpm install
✔ Installed 10 packages
✔ Linked 0 latest versions
✔ Run 0 scripts
✔ All packages installed (used 8ms(network 6ms), speed 0B/s, json 0(0B), tarball 0B)
```



## 1.4 启动项目并访问

确保通过cnpm将依赖安装正确

> 修改项目配置文件：Gruntfile.js

```js
connect: {
    server: {
        options: {
            hostname: '*',  // 添加该配置
            port: 9100,
                base: '.',
                    keepalive: true
        }
    }
}

```

![image-20210317215619602](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210317214700491.png)

> 启动项目

```shell
[root@localhost elasticsearch-head-master]# cnpm run start

> elasticsearch-head@0.0.0 start /home/software/es-cluster/plugins/elasticsearch-head-master
> grunt server

Running "connect:server" (connect) task
Waiting forever...
Started connect web server on http://localhost:9100
```

> 启动成功，通过IP:9100访问

![image-20210317215823222](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210317215823222.png)