# nvm 简介

Node Version Manager（Node版本管理器）,能灵活的切换node版本。



# nvm 安装

```sh
sudo pacman -S nvm
sudo echo 'source /usr/share/nvm/init-nvm.sh' >> ~/.bashrc
sudo echo 'export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node' >> ~/.xprofile
```



# nvm 使用

```sh
# 查看LTS版本
nvm ls-remote --lts
```

```sh
# 安装
nvm install v12.19.0
```

```sh
# 查看当前版本
nvm current
```

```sh
# 查看已安装版本
nvm ls
```

```sh
# 版本临时切换
nvm use lts/erbium
```

```sh
# 彻底的版本切换
nvm alias default v12.19.0
```

