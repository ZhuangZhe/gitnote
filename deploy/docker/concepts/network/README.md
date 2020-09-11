# 网络



## 创建一个network

```bash
zhes-mbp:~ zhezhuang$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
5761d0ff5592        bridge              bridge              local
bce8d73a72e1        host                host                local
23c2627d8bc9        none                null                local
zhes-mbp:~ zhezhuang$ docker network create my_app_net
70e5b19fbb7a75c310495b67d99d60c221b50eceef7a8cc96a27bdcc26454b6a
zhes-mbp:~ zhezhuang$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
5761d0ff5592        bridge              bridge              local
bce8d73a72e1        host                host                local
70e5b19fbb7a        my_app_net          bridge              local
23c2627d8bc9        none                null                local
```



## 创建一个容器并将其连接到指定network

```bash
zhes-mbp:~ zhezhuang$ docker container run -d --name new_nginx --network my_app_net nginx
cc8af6d67cb47ac5e501a879a494eb717aa84088beaeb8121823de369d03ac92
zhes-mbp:~ zhezhuang$ docker network inspect my_app_net
[
    {
        "Name": "my_app_net",
        "Id": "70e5b19fbb7a75c310495b67d99d60c221b50eceef7a8cc96a27bdcc26454b6a",
        "Created": "2020-09-11T05:48:11.559620519Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "cc8af6d67cb47ac5e501a879a494eb717aa84088beaeb8121823de369d03ac92": {
                "Name": "new_nginx",
                "EndpointID": "5deb7ddee6265ece4c3919fbb8cd0ff704584d2ec47fd6f881cae41f5c6e73a6",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```



## 将已经创建的容器连接到指定network

```bash
zhes-mbp:~ zhezhuang$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
3d09eaef527b        nginx               "/docker-entrypoint.…"   3 minutes ago       Up 3 minutes        0.0.0.0:80->80/tcp    webhost
cc8af6d67cb4        nginx               "/docker-entrypoint.…"   6 minutes ago       Up 6 minutes        80/tcp                new_nginx
07a677cb31f3        mysql               "docker-entrypoint.s…"   6 hours ago         Up 6 hours          3306/tcp, 33060/tcp   mysql
1098663a9393        nginx               "/docker-entrypoint.…"   6 hours ago         Up 6 hours          80/tcp                nginx
zhes-mbp:~ zhezhuang$ docker network connect 70e5b19fbb7a 3d09eaef527b
```









