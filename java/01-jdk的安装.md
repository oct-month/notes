# jdk 的安装

## 1、ubuntu

安装 JDK：

```sh
sudo apt-get install -y openjdk-8-jdk openjfx
sudo apt-get install -y openjdk-11-jdk
sudo apt-get install -y openjdk-14-jdk
```

查看已安装的 JDK：

```sh
update-java-alternatives -l
```

切换 JDK 版本（两种方式）：

```sh
sudo update-java-alternatives -s java-1.14.0-openjdk-amd64
```

```sh
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

