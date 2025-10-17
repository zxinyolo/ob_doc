```shell
docker pull bitnami/etcd

# 配置文件

#成员名称
name: etcd01
# 数据保存路径
data-dir: /data
#对外提供服务的地址
listen-client-urls: http://0.0.0.0:2379
#成员之间通信地址
listen-peer-urls: http://0.0.0.0:2380
#此成员的客户端URL列表，用于通告群集的其余部分。这些URL可以包含域名
advertise-client-urls: http://etcd01:2379


docker run -d --name etcd -p 2379:2379 -p 2380:2380 -e ALLOW_NONE_AUTHENTICATION=yes -e ETCD_CONFIG_FILE=/opt/bitnami/etcd/conf/etcd.yml -v f:\docker\etcd\etcd01\data:/data -v f:\docker\etcd\etcd01\conf:/opt/bitnami/etcd/conf bitnami/etcd:latest

```

