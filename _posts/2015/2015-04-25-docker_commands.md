


## tag

**命令格式：**

`docker tag` 用来标记已存在的镜像，命令格式如下：

```
docker@boot2docker:~$ docker tag --help

Usage: docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]

Tag an image into a repository

  -f, --force=false    Force
  --help=false         Print usage
```

**命令详解：**

* 可选择带 [OPTIONS]，主要是 `-f`，当存在某个标记的时候，可以使用该选项强制替换原标记
* 将已存在的镜像 IMAGE[:TAG] 打个标记，其中 [:TAG] 可以选，默认为 latest 标签
* 标记的名字为 [REGISTRYHOST/][USERNAME/]NAME[:TAG]，其中可以指定注册服务器的主机名 [REGISTRYHOST/]，默认为 Docker Hub 官方服务器，还可以指定到用户名 [USERNAME/]，同时可以加上标签 [:TAG]

**命令示例：**

将官方的 `hello-world` 库 push 到

```
docker@boot2docker:~$ docker login
Username: hazirguo
Password:
Email: *****@*****.com
Login Succeeded
docker@boot2docker:~$ sudo docker push hello-world
FATA[0000] You cannot push a "root" repository. Please rename your repository to <user>/<repo> (ex: <user>/hello-world)
```


```
docker@boot2docker:~$ docker tag hello-world hazirguo/hello-world
docker@boot2docker:~$ docker push hazirguo/hello-world
The push refers to a repository [hazirguo/hello-world] (len: 1)
Sending image list
Pushing repository hazirguo/hello-world (1 tags)
a8219747be10: Image successfully pushed
91c95931e552: Image successfully pushed
Pushing tag for rev [91c95931e552] on {https://cdn-registry-1.docker.io/v1/repositories/hazirguo/hello-world/tags/latest}
```


## 与注册服务器相关的命令

与注册服务器相关的命令主要有五个，分别为：

* `docker login` —— 注册或登陆 Docker 注册服务器
* `docker logout` —— 登出 Docker 注册服务器
* `docker pull` —— 从注册服务器 pull 一个镜像或一个资源库到本地
* `docker push` —— 将一个镜像或一个资源库 push 到注册服务器上
* `docker search` —— 从注册服务器上搜索镜像

下面分别解释这些命令：

### login

使用用户名和密码登陆到

```
Usage: docker login [OPTIONS] [SERVER]

Register or log in to a Docker registry server, if no server is specified "https://index.docker.io/v1/" is the default.

  -e, --email=""       Email
  --help=false         Print usage
  -p, --password=""    Password
  -u, --username=""    Username
```








