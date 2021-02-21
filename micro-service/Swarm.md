# Docker Swarm

## 集群搭建

三台主机，ubuntu1、ubuntu2、ubuntu3。

manager：ubuntu1

```sh
docker swarm init --advertise-addr 192.168.142.128
```

worker：ubuntu2、ubuntu3

```sh
# token 要看ubuntu1上的输出
docker swarm join --token SWMTKN-1-0pimswqdnijtt1o5etbql9w1zencb0viresoihjzweo5ezbxl1-2vn5jjbkcdsy489sxicbb6tjk 192.168.142.128:2377
```

```sh
# 提升ubuntu2作为manager（ubuntu2同时是manager和worker）
docker node promote ubuntu2
# 提升ubuntu3
docker node promote ubuntu3
# 查看node状态
docker node ls
```

## 服务创建

```sh
# --detach=false 可以看见服务的创建过程
# alpine是最小的linux image
 docker service create --name nginx --detach=false nginx
 # 日志查看
 docker service logs nginx
 # 查看服务配置
 docker service inspect nginx
```

## 服务更新

```sh
# 将主机的端口映射出去
docker service update --publish-add 8080:80 --detach=false nginx
```

## 服务扩缩容

```sh
# 扩展为3个
docker service scale nginx=3 --detach=false
```

## 服务间的访问

```sh
docker network create -d overlay test-overlay
docker service create --network test-overlay --name nginx -p 8080:80 --detach=false nginx
docker service create --network test-overlay --name alpine --detach=false alpine ping www.baidu.com
# 在alpine中可以通过nginx这个名字访问到nginx服务，ping nginx、wget nginx
```

```sh
docker service create --name nginx-b --endpoint-mode dnsrr --detach=false nginx
docker service update --network-add test-overlay --detach=false nginx-b
# 在alpine中可以通过nginx-b这个名字访问到服务
# dnsrr不能有端口映射，容器间调用时使用
```

## 服务组打包

service.yml

```yaml
version: "3.4"

services:
    alpine:
        image: alpine
        command:
            - "ping"
            - "www.baidu.com"
        networks:
            - "test-overlay"
        deploy:
            endpoint_mode: dnsrr
            replicas: 2
            restart_policy:
                condition: on-failure
            resources:
                limits:
                    cpus: "0.1"
                    memory: 50M
        depends_on:
            - nginx
    nginx:
        image: nginx
        networks:
            - "test-overlay"
        ports:
            - "8080:80"

networks:
    test-overlay:
        external: true
```

部署，alpine和nginx服务将带上test_前缀

```sh
docker stack deploy -c service.yml test
```

