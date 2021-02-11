# Mesos

## 编译安装

> 并未成功。。。

### 下载地址

https://mirrors.tuna.tsinghua.edu.cn/apache/mesos/1.11.0/mesos-1.11.0.tar.gz

### 依赖

```sh
# gcc；g++；python >2.6，<3；openjdk8；maven
sudo apt-get -y install gcc g++ python openjdk-8-jdk maven
sudo apt-get -y install autoconf libtool
sudo apt-get -y install build-essential python-dev python-six libcurl4-nss-dev libsasl2-dev libsasl2-modules libapr1-dev libsvn-dev zlib1g-dev iputils-ping
```

### 编译

```sh
tar -xzvf mesos-1.11.0.tar.gz
cd mesos-1.11.0

vim src/Makefile.am
# 将 MESOS_CPPFLAGS += @WERROR@ 注释。

mkdir build && cd build
../configure
make -j6
```

**报错：**

```sh
vim 3rdparty/grpc-1.10.0/src/core/lib/gpr/log_linux.cc
# 将文件中的 gettid 函数声明及调用的地方重命名为 sys_gettid。
make -j6 #继续make
```

这边编译依旧报错，没有成功。

## Docker容器

> 最后决定采用docker来运行。
>
> 但还是没成功，marathon卡在loading。

1. Docker安装：

   https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/

2. Docker hub镜像加速：

   `阿里云->容器镜像服务->镜像加速器`

3. 配置普通用户运行：

```sh
sudo usermod -a -G docker $USER
reboot
```

4. 运行Mesos

运行mesos-master：

```sh
docker run -d --net=host \
        --restart=on-failure \
        --name=mesos-master \
        -e MESOS_PORT=5050 \
        -e MESOS_ZK=zk://192.168.10.9:2181/mesos \
        -e MESOS_QUORUM=1 \
        -e MESOS_REGISTRY=in_memory \
        -e MESOS_LOG_DIR=/var/log/mesos \
        -e MESOS_WORK_DIR=/var/tmp/mesos \
        -v "$(pwd)/log/mesos:/var/log/mesos" \
        -v "$(pwd)/work/mesos:/var/tmp/mesos" \
        mesosphere/mesos-master:1.7.1 \
        --no-hostname_lookup --ip=192.168.142.129
```

运行mesos-slave：

```sh
docker run -d --net=host --privileged \
        --restart=on-failure \
        --name=mesos-slave \
        -e MESOS_PORT=5051 \
        -e MESOS_MASTER=zk://192.168.10.9:2181/mesos \
        -e MESOS_SWITCH_USER=0 \
        -e MESOS_CONTAINERIZERS=docker,mesos \
        -e MESOS_LOG_DIR=/var/log/mesos \
        -e MESOS_WORK_DIR=/var/tmp/mesos \
        -v "$(pwd)/log/mesos:/var/log/mesos" \
        -v "$(pwd)/work/mesos:/var/tmp/mesos" \
        -v /var/run/docker.sock:/var/run/docker.sock \
        -v /sys:/sys \
        -v /usr/bin/docker:/usr/local/bin/docker \
        mesosphere/mesos-slave:1.7.1 \
        --no-systemd_enable_support \
        --no-hostname_lookup --ip=192.168.142.128
```

运行marathon：

```sh
docker run -d --net=host \
        --restart=on-failure \
        --name=marathon \
        mesosphere/marathon:v1.11.24 \
        --master zk://192.168.10.9:2181/mesos \
        --zk zk://192.168.10.9:2181/marathon
```

