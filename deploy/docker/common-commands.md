# 常用命令

| 命令 | 简介 |
| :--- | :--- |
| `docker version` | 获取docker版本 |
| `docker info` | 获取docker详细信息 |
| `docker build` | 构建docker镜像 |
| `docker run` | 运行docker容器 |
| `docker rm` | 删除docker容器 |
| `docker container` | 容器相关命令 |
|  |  |



## `docker container`

**`docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]`**

| option | 作用 |
| :--- | :--- |
| `--detach , -d` | 让容器在后台运行 |
| `--env, -e` | 为容器设置环境变量 |
| `--interactive , -i` | 保持STDIN打开 |
| `--mount` | 给容器附上一个filesystem mount |
| `--name` | 给容器起一个名字 |
| `--publish , -p` | 给容器分配一个接口 |
| `-tty, -t` | Allocate a pseudo-TTY |



**`docker container ls [OPTIONS]`**

| option | 作用 |
| :--- | :--- |
|  `` | 列出正在运行的容器 |
| `-a` | 列出所有容器 |



**`docker container logs [OPTIONS] CONTAINER`**

| option | 作用 |
| :--- | :--- |
|  `` | 查询容器的日志 |



**`docker container rm [OPTIONS] CONTAINER [CONTAINER...]`**

| option | 作用 |
| :--- | :--- |
|  `` | 删除指定已暂停运行的容器 |
| `-f` | 删除指定容器，无论是否在运行 |



**`docker container top CONTAINER [ps OPTIONS]`**

| option | 作用 |
| :--- | :--- |
|  `` | 获取一个容器的进程列表 |



**`docker container inspect [OPTIONS] CONTAINER [CONTAINERS...]`**

| option | 作用 |
| :--- | :--- |
|  `` | 获取一个容器的配置细节 |
| `--format` | 使用go模版规定输出格式 |



**`docker container stats [OPTIONS] [CONTAINER...]`**

| option | 作用 |
| :--- | :--- |
|  `` | 获取所有容器的性能数据 |



**`docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]`**

| option | 作用 |
| :--- | :--- |
|  `` | 在一个容器中使用命令行 |
| `-it` | 在一个已存在的容器中使用命令行 |



**`docker container start [OPTIONS] CONTAINER [CONTAINERS...]`**

| `option` | 作用 |
| :--- | :--- |
|  `` | 启动一个或多个容器 |



## `docker image`

**`docker image prume [OPTIONS]`**

| option | 作用 |
| :--- | :--- |
|  `` | 删除所有未使用的镜像 |
| `--all, -a` | 删除所有镜像 |
| `--force, -f` | 删除镜像时不进行确认 |



## `docker network`

**`docker network ls [OPTIONS]`**

| option | 作用 |
| :--- | :--- |
|  `` | 列出所有网络 |



**`docker network inspect [OPTIONS] NETWORK [NETWORK...]`**

| option | 作用 |
| :--- | :--- |
|  `` | 显示一个或多个网络的详细信息 |



**`docker network create [OPTIONS] NETWORK`**

| option | 作用 |
| :--- | :--- |
|  `` | 创建一个新网络 |



**`docker network connect [OPTIONS] NETWORK CONTAINER`**

| option | 作用 |
| :--- | :--- |
|  `` | 将一个容器连到一个网络 |



**`docker network disconnect [OPTIONS] NETWORK CONTAINER`**

| option | 作用 |
| :--- | :--- |
|  `` | 将一个容器从一个网络移除 |









