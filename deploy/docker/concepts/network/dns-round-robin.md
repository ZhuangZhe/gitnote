# DNS Round Robin

用Docker实现简单DNS Round Robin功能。

## 新建一个新network

```bash
zhes-mbp:~ zhezhuang$ docker network create dude
63709bc60a3cb3f92dee784a7b952e13f48b2a7f1e6385e18696aa2fe458fc1c
```



## 使用同一个镜像新建两个相同的容器，并使用相同的Alias

```bash
zhes-mbp:~ zhezhuang$ docker container run -d --net dude --net-alias search elasticsearch:2
7accc2b4404d388a130061fe3d1d03c90835cca7d6c7d173f24e03b4c40413a8
zhes-mbp:~ zhezhuang$ docker container run -d --net dude --net-alias search elasticsearch:2
63b96d1ed759e0cebd727ebf96936effa8ba03f441760fe8825698983c344072
```



## 验证可以使用同一个名分别访问到两个容器

```bash
zhes-mbp:~ zhezhuang$ docker container run --rm --net dude alpine nslookup search
Server:		127.0.0.11
Address:	127.0.0.11:53

Non-authoritative answer:
Non-authoritative answer:
Name:	search
Address: 172.19.0.2
Name:	search
Address: 172.19.0.3
```

```bash
zhes-mbp:~ zhezhuang$ docker container run --rm --net dude centos curl -s search:9200
{
  "name" : "Alexander Goodwin Pierce",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "iG7exzZGTDSPpDhNXMWmEg",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}
zhes-mbp:~ zhezhuang$ docker container run --rm --net dude centos curl -s search:9200
{
  "name" : "Ultimo",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "qmsxLIASRbS7FfTvdP0EcQ",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}
```









