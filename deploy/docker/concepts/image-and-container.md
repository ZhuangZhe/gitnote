# 镜像与容器

一个镜像是一个你想跑的程序。

一个容器是一个镜像的实例。

可以多个容器跑同一个镜像。

## 创建一个nginx容器

```bash
docker container run --publish 80:80 nginx
```

1. 从Docker Hub下载一个nginx的镜像。
2. 用这个nginx的镜像启动一个容器。
3. 向主机开放80端口。
4. 主机将通往80端口的流量路由到容器的80端口。

```bash
docker container run --publish 80:80 --detach nginx
```

* `detach`使容器的后台运行。

```bash
zhes-mbp:~ zhezhuang$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
ed08550712aa        nginx               "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes        0.0.0.0:80->80/tcp   blissful_keller
```

* 停止运行容器。

```bash
zhes-mbp:~ zhezhuang$ docker container stop ed08550712aa
ed08550712aa
zhes-mbp:~ zhezhuang$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
zhes-mbp:~ zhezhuang$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
ed08550712aa        nginx               "nginx -g 'daemon of…"   5 minutes ago       Exited (0) 13 seconds ago                       blissful_keller
89fca093db31        nginx               "nginx -g 'daemon of…"   12 minutes ago      Exited (0) 6 minutes ago                        admiring_wescoff
```

* 如果未提供容器的名字，docker会自动生成一个名字，可用通过使用`--name`提供名字。

```bash
zhes-mbp:~ zhezhuang$ docker container run --publish 80:80 --detach --name webhost nginx
c583a8de98dbdf9b060adf69c44351094cfa039d619fef6af0348af0e10327b3
```

* 查看容器的日志

```bash
zhes-mbp:~ zhezhuang$ docker container logs webhost
172.17.0.1 - - [10/Sep/2020:22:50:37 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36" "-"
172.17.0.1 - - [10/Sep/2020:22:50:37 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36" "-"
```

* 查看正在运行的容器

```bash
zhes-mbp:~ zhezhuang$ docker container top webhost
PID                 USER                TIME                COMMAND
89879               root                0:00                nginx: master process nginx -g daemon off;
89916               101                 0:00                nginx: worker process
```

* 同时删除多个容器

```bash
zhes-mbp:~ zhezhuang$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                NAMES
c583a8de98db        nginx               "nginx -g 'daemon of…"   8 minutes ago       Up 8 minutes                0.0.0.0:80->80/tcp   webhost
ed08550712aa        nginx               "nginx -g 'daemon of…"   20 minutes ago      Exited (0) 15 minutes ago                        blissful_keller
89fca093db31        nginx               "nginx -g 'daemon of…"   27 minutes ago      Exited (0) 21 minutes ago                        admiring_wescoff
zhes-mbp:~ zhezhuang$ docker container rm ed08550712aa 89fca093db31
ed08550712aa
89fca093db31
zhes-mbp:~ zhezhuang$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
c583a8de98db        nginx               "nginx -g 'daemon of…"   8 minutes ago       Up 8 minutes        0.0.0.0:80->80/tcp   webhost
zhes-mbp:~ zhezhuang$ 
```

* 删除正在运行的容器，使用`-f`

```bash
zhes-mbp:~ zhezhuang$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
c583a8de98db        nginx               "nginx -g 'daemon of…"   8 minutes ago       Up 8 minutes        0.0.0.0:80->80/tcp   webhost
zhes-mbp:~ zhezhuang$ docker container rm -f c583a8de98db
c583a8de98db
zhes-mbp:~ zhezhuang$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## 当使用`docker container run`之后会发生什么

1. 在本地镜像缓存中寻找需要的镜像
2. 如果在缓存找不到想要的镜像，会去远程的镜像仓库找
3. 现在最新版本的镜像
4. 基于获取到镜像构建一个容器并准备启动
5. 在容器引擎的私有网络中赋予这个新容器一个虚拟ip地址
6. 将通往主机某个端口的流量设定为路由到指定的容器端口
7. 使用dockerfile中的命令启动容器

## 使用容器的shell

一个新建可交互的容器

```bash
docker container run -it
```

新建一个容器并使用shell

```bash
zhes-mbp:~ zhezhuang$ docker container run -it --name proxy nginx bash
root@aa14a36a9afc:/# 
```











