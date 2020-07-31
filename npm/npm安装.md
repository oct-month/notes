## 安装

```sh
sudo apt-get install npm
```



## 换源

- 两种方式

1）改配置文件

```sh
npm config set registry https://registry.npm.taobao.org
```

2）使用淘宝提供的 cnpm 工具替换 npm

```sh
npm install -g cnpm --registry=https://registry.npm.taobao.org
```



## 注意

cnpm install 安装依赖默认不写入 package.json 文件，需要加 -S 选项：

```sh
cnpm install 依赖包@版本 -S
```

