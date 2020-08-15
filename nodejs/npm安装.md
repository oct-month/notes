## ubuntu 安装

```sh
sudo snap install node
# 或者
sudo apt-get install nodejs npm
```



## Windows 安装

https://nodejs.org/en/download 官网下载最新的 LTS 版本。然后安装到 D:\nodejs 目录下



## 换源

- 两种方式

1）修改 npm 的配置

```sh
npm config set registry https://registry.npm.taobao.org/
npm config set disturl https://npm.taobao.org/dist/
npm config set electron_mirror https://npm.taobao.org/mirrors/electron/
npm config set sass_binary_site https://npm.taobao.org/mirrors/node-sass/
npm config set phantomjs_cdnurl https://npm.taobao.org/mirrors/phantomjs/
npm config set chromedriver_cdnurl https://npm.taobao.org/mirrors/chromedriver/
npm config set operadriver_cdnurl https://npm.taobao.org/mirrors/operadriver/
npm config set python_mirror https://npm.taobao.org/mirrors/python/
npm config set electron_builder_binaries_mirror https://npm.taobao.org/mirrors/electron-builder-binaries/
npm config set node_sqlite3_binary_host_mirror https://npm.taobao.org/mirrors
```

2）使用淘宝提供的 cnpm 工具替换 npm

```sh
npm install -g cnpm --registry=https://registry.npm.taobao.org
```



## 注意

1、cnpm install 安装package默认不写入 package.json 文件，需要加 -S 选项：

```sh
cnpm install 依赖包@版本 -S
```

2、在 ubuntu 下，cnpm会默认把package安装到 /usr/local/lib/node_modules/ 目录下，而 npm 会把package安装到 /usr/lib/node_modules/ 目录下；cnpm 的缓存目录是 ~/.npm/.npm/，而 npm 的缓存目录是 ~/.npm。所以两者是互不影响的。

3、sudo npm 安装package失败时（提示没有权限），可以加上 --unsafe-perm 参数；例如：

```sh
sudo npm install electron -g -f --unsafe-perm
```



## 配置文件

npm 的配置文件是 .npmrc

cnpm 的配置文件是 .cnpmrc

**windows下的配置：**

.npmrc：

```properties
prefix=D:\nodejs\node_global
cache=D:\nodejs\node_cache
package-lock=false
registry=https://registry.npm.taobao.org/
disturl=https://npm.taobao.org/dist/
electron_mirror=https://npm.taobao.org/mirrors/electron/
sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
phantomjs_cdnurl=https://npm.taobao.org/mirrors/phantomjs/
chromedriver_cdnurl=https://npm.taobao.org/mirrors/chromedriver/
operadriver_cdnurl=https://npm.taobao.org/mirrors/operadriver/
python_mirror=https://npm.taobao.org/mirrors/python/
electron_builder_binaries_mirror=https://npm.taobao.org/mirrors/electron-builder-binaries/
node_sqlite3_binary_host_mirror=https://npm.taobao.org/mirrors
```

.cnpmrc

```properties
prefix=D:\nodejs\node_global
cache=D:\nodejs\node_cache
```

环境变量：

```properties
NODE_PATH=D:\nodejs\node_global
PATH=%PATH%;%NODE_PATH%
```

然后

```sh
mv D:\nodejs\node_modules D:\nodejs\node_global\
mv D:\nodejs\npm* D:\nodejs\node_blobal\
mv D:\nodejs\npx* D:\nodejs\node_global\
```

> ubuntu下无特殊需求不用配置，只是在安装时需要 root 权限。



