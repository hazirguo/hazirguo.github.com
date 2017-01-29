# Ubuntu 14.04 下安装 Docker

本文介绍如何在 Ubuntu 14.04(trusty) 下安装 Docker。

* 安装之前先更新一下安装包：

```
apt-get install update
```

* 通过安装 docker-io 包来安装 Docker：

```
apt-get install docker.io
```

* 创建软连接以及修正几个路径名：

```
ln -sf /usr/bin/docker.io /usr/local/bin/docker
sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker.io
```

* 最后也是可选的，配置 Docker 服务开机启动：

```
update-rc.d docker.io defaults
```
