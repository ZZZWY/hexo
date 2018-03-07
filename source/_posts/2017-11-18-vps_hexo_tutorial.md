---
layout:     post
title:      "在个人服务器上搭建hexo个人博客"
date:       2017-11-18 12:00:00
author:     "ZWY"
catalog: true
tags:
    - 建站
    - 教程
    - 原创
---
今年的2月份第一次有了写博客的想法,于是利用github_page搭建了一个个人博客,但是体验太差,连接会很慢(不知道是不是和GFW有关),但是因为工作太忙的原因,一直也没有管他,最近稍微轻松了些于是决定重新搭建一个,正好前段时间双十一阿里云的ECS还挺便宜,果断入手了一个.
hexo很强大,容易上手,丰富的主题,支持后台命令行这些都是选择的理由,所以hexo+vps就是最后我的选择.
<!-- more -->
# 本机配置hexo
安装hexo并没有什么难点,推荐查看[hexo官方教程](https://hexo.io/zh-cn/docs/index.html),唯一需要注意的是,请一定将npm切换为国内的源,比如cnpm,不然安装进度会慢到你怀疑人生.

# vps上部署git服务器
因为我们并没有使用github的仓库而是使用自己的vps,所以我们需要将自己的vps配置成一台类似于github一样的git服务器来作为网站的仓库.

### 安装
`sudo apt-get install git`

### 创建'git'用户
输入密码后一路通过就可以
`sudo adduser git`

### 创建仓库
选取一个文件夹作为仓库的地址
`sudo git init --bare /var/repo/blog.git #此处替换为你的地址`

### 配置hooks
配置 git hooks，关于 hooks 的详情内容可以参考[Pro Git](https://www.git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)
我们这里要使用的是 post-receive 的 hook，这个 hook 会在整个 git 操作过程完结以后被运行。

在 blog.git/hooks 目录下新建一个 post-receive 文件
`sudo touch /var/repo/blog.git/hooks/post-receive`
之后在文件中写入:
```
#!/bin/bash
GIT_REPO=/var/repo/hexo.git
TMP_GIT_CLONE=/tmp/hexo
PUBLIC_WWW=/var/www/hexo_mine
rm -rf ${TMP_GIT_CLONE}
git clone $GIT_REPO $TMP_GIT_CLONE
rm -rf ${PUBLIC_WWW}/*
cp -rf ${TMP_GIT_CLONE}/* ${PUBLIC_WWW}
```
可以看到命令是在我们每次push后将仓库更新到`PUBLIC_WWW`,这个路径也就是你web服务器的位置,稍后会讲到
文件写入后修改文件的权限:
`chmod +x post-receive`

### 改变 blog.git 目录的拥有者为 git 用户：
`sudo chown -R git:git blog.git`

### 添加公钥
创建证书登录，把自己电脑的公钥，也就是 `~/.ssh/id_rsa.pub` 文件里的内容添加到服务器的 `/home/git/.ssh/authorized_keys` 文件中，添加公钥之后可以防止每次 push 都输入密码。
关于git公钥的生成可以查看[github help](https://help.github.com/articles/connecting-to-github-with-ssh/)

### 本机配置
回到本机,修改hexo根目录下的`_config.yml` 配置文件:
```
deploy:
  type: git
  repo: git@1.1.1.1:/var/repo/blog.git
  branch: master
```
其中`repo`替换为你的仓库地址

至此,我们git服务器就已经配置完成

# 配置web服务器
这里我们选择Nginx

### 安装
`sudo apt-get install nginx`

### 配置
创建一份配置文件
`sudo touch /etc/nginx/conf.d/hexo.conf`
在文件中写入:
```
server {
    listen         80 ;
    root /var/www/hexo_mine;//这里可以改成你的网站目录地址
    server_name www.example.com;//这里输入你的域名或IP地址
    access_log  /var/log/nginx/hexo_access.log;
    error_log   /var/log/nginx/hexo_error.log;
    location ~* ^.+\.(ico|gif|jpg|jpeg|png)$ {
            root /var/www/hexo_mine;
            access_log   off;
            expires      1d;
    }
    location ~* ^.+\.(css|js|txt|xml|swf|wav)$ {
        root /var/www/hexo_mine;
        access_log   off;
        expires      10m;
    }
    location / {
        root /var/www/hexo_mine;
        if (-f $request_filename) {
            rewrite ^/(.*)$  /$1 break;
        }
    }
}
```
### 重启服务
重启nginx使配置生效
`sudo service nginx restart`

# 使用
查看[官方文档](https://hexo.io/zh-cn/docs/commands.html)
```
hexo new post_1
hexo g -d
```
以上命令就一键生成静态页面并且部署到你的服务器上了,如果你的服务器不是大陆的不用备案,那么你现在直接访问就可以了.[hexo官方主题列表](https://hexo.io/themes/)有很多开源的主题,顺便推荐我现在用的主题:[even](https://github.com/ahonn/hexo-theme-even),安装方法在他的readme里很详细.

希望玩的开心,专注于写作.
