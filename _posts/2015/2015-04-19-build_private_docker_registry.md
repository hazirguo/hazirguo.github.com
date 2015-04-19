---
layout: post
title: "搭建私有 Docker 仓库服务器"
description: ""
category: docker
tags: [docker,registry]
---
{% include JB/setup %}

[Docker Hub](https://hub.docker.com) 是 Docker 官方的公共仓库服务器，用户在 DockerHub 上只能创建一个私有仓库，这对于有些用户是不够用的，而且 DockerHub 服务器的访问速度也是个很大问题，那么我们希望能在自己本地的服务器上创建一个类似于 DockerHub 仓库服务器供团队使用，这也是可以的。

我测试的环境是 Mac OSX 下，已经通过 Boot2Docker 工具安装好 Docker 的环境，通过 命令 `boot2docker ip` 可以查看虚拟机的 IP 是 `192.168.59.104`。那么下面我就在自己虚拟机上搭建一个私有的 Docker 仓库服务器：

## 安装运行 Docker-Registry

运行官方提供的 registry 镜像，将端口映射到主机的 5000 端口上，其它均使用默认配置：

```
guohl@ghl-MBP ⮀ ~ ⮀ docker run -d -p 5000:5000 registry
Unable to find image 'registry:latest' locally
6cfde7386ab2: Pull complete
9789d95d9fda: Pull complete
19443e64f223: Pull complete
b329371ab73c: Pull complete
f0daee9a4e8f: Pull complete
a66e50e56475: Pull complete
8ab3d2988df5: Pull complete
5f60fa7ea945: Pull complete
db22a140c899: Pull complete
5b2fff9306bd: Pull complete
511136ea3c5a: Already exists
f3c84ac3a053: Already exists
a1a958a24818: Already exists
9fec74352904: Already exists
d0955f21bf24: Already exists
registry:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Status: Downloaded newer image for registry:latest
8fb8e82e61822e593e10b59a4e7bbad18c789b34e3b38942d5b63dccb497ed09
```

## 上传镜像到私有仓库

创建好私有仓库之后，我们就可以向该仓库上传镜像，别人也可以从该仓库下载镜像了。

查看本地已有的镜像：

```
guohl@ghl-MBP ⮀ ~ ⮀ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
nginx                       latest              637d3b2f5fb5        4 days ago          93.44 MB
mysql                       latest              0feafece277d        11 days ago         282.9 MB
wordpress                   latest              f90659c8fdb9        2 weeks ago         451.5 MB
ubuntu                      latest              d0955f21bf24        4 weeks ago         188.3 MB
google/golang               latest              3cc1d7ae0e9c        11 weeks ago        611.3 MB
hello-world                 latest              e45a5af57b00        3 months ago        910 B
```

通过 `docker tage` 命令将 `hello-world` 这个镜像标记为 `192.168.59.104:5000/hello-world`之后，再 push 到该镜像到私有仓库：

```
guohl@ghl-MBP ⮀ ~ ⮀ docker tag hello-world 192.168.59.104:5000/hello-world
guohl@ghl-MBP ⮀ ~ ⮀ docker push 192.168.59.104:5000/hello-world
FATA[0000] Error: v1 ping attempt failed with error: Get https://192.168.59.104:5000/v1/_ping: dial tcp 192.168.59.104:5000: connection refused. If this private registry supports only HTTP or HTTPS with an unknown CA certificate, please add `--insecure-registry 192.168.59.104:5000` to the daemon's arguments. In the case of HTTPS, if you have access to the registry's CA certificate, no need for the flag; simply place the CA certificate at /etc/docker/certs.d/192.168.59.104:5000/ca.crt
```

发现报错，使用 [SO上类似问题](http://stackoverflow.com/questions/28712455/pushing-files-into-private-registry-in-docker) 的解决方法可以解决上面的错误：

>To use the `--insecure-registry` option, add it to the file `/var/lib/boot2docker/profile` inside the boot2docker VM. You can get into the VM with `boot2docker ssh`. The file contents should look like:
>
>`EXTRA_ARGS="--insecure-registry REGISTRY_IP:PORT"`
>You will then need to restart boot2docker (e.g. `boot2docker restart`).

步骤如下：

1. 使用 `boot2docker ssh` 登陆到 boot2docker 虚拟机
2. 修改 `/var/lib/boot2docker/profile` 文件，向该文件中增加一行： `EXTRA_ARGS="--insecure-registry 192.168.59.104:5000"`
3. 退出该虚拟机并使用命令 `boot2docker restart` 重启 boot2docker

完成重启之后，将私有仓库服务器运行起来，并 push hello-world 到该仓库：

```
guohl@ghl-MBP ⮀ ~ ⮀ docker run -d -p 5000:5000 registry
4935607095a22655da1ef91feb6f569264a50529cb8d594d520fe62da81250db
guohl@ghl-MBP ⮀ ~ ⮀ docker push 192.168.59.104:5000/test
The push refers to a repository [192.168.59.104:5000/test] (len: 1)
Sending image list
Pushing repository 192.168.59.104:5000/test (1 tags)
511136ea3c5a: Image successfully pushed
31cbccb51277: Image successfully pushed
e45a5af57b00: Image successfully pushed
Pushing tag for rev [e45a5af57b00] on {http://192.168.59.104:5000/v1/repositories/test/tags/latest}
```

使用 Docker 的 RESTful API 可以查看仓库服务器中的镜像：

```
guohl@ghl-MBP ⮀ ~ ⮀ curl http://192.168.59.104:5000/v1/search
{"num_results": 1, "query": "", "results": [{"description": "", "name": "library/hello-world"}]}%
```

表示 `hello-world` 镜像已成功长传至私有仓库服务器了。


## 从私有仓库中下载、搜索镜像

其他机器可以从私有仓库服务器上下载、搜索镜像等，与从 Docker Hub 上操作无异，只不过需要指出仓库的位置，如：

```
guohl@ghl-MBP ⮀ ~ ⮀ docker rmi -f 192.168.59.104:5000/hello-world
Untagged: 192.168.59.104:5000/hello-world:latest
Deleted: e45a5af57b00862e5ef5782a9925979a02ba2b12dff832fd0991335f4a11e5c5
Deleted: 31cbccb51277105ba3ae35ce33c22b69c9e3f1002e76e4c736a2e8ebff9d7b5d
guohl@ghl-MBP ⮀ ~ ⮀ docker search 192.168.59.104:5000/hello-world
NAME                  DESCRIPTION   STARS     OFFICIAL   AUTOMATED
library/hello-world                 0
guohl@ghl-MBP ⮀ ~ ⮀ docker pull 192.168.59.104:5000/hello-world
Pulling repository 192.168.59.104:5000/hello-world
e45a5af57b00: Download complete
511136ea3c5a: Download complete
31cbccb51277: Download complete
Status: Downloaded newer image for 192.168.59.104:5000/hello-world:latest
```

----

**参考资料：** http://dockerpool.com/static/books/docker_practice/repository/local_repo.html







