## 安装

```sh
npm install hexo-cli -g
```

```sh
cnpm install hexo-cli -g
```



## 使用

- 初始化一个工作mul

```sh
hexo init hexoblog
cd hexoblog
npm install --save
```

md文件的存放位置：source/_posts/

- 生成静态的web文件

```sh
hexo g
```

- 打开服务，预览效果

```sh
hexo s
```

- 清除静态文件

```sh
hexo clean
```

- 发布hexo到github（或其它仓库）上

在_config.yml文件中配置远程github仓库

```yaml
deploy:
  type: git
  repository: # 你的仓库git地址
  branch: master
```

发布命令：

```sh
hexo d
```

