---
title: 使用Hexo搭建自己的博客
date: 2020-03-16 00:32:46
categories: 技术
tags: Hexo
---
一直想搭建一个自己的博客，把一些自己所学的，所思考的东西记在里面，为了不丢，也为了让自己有写作记录的习惯，同时也可以和朋友们讨论一下。

>搭建环境 \
>本地：WSL \
>服务器端：ubuntu 18.04 \
>服务器端: github 

# 搭建思路
为了不让初学者迷糊，这里引用一张图便于理解

![Hexo搭建博客框架](/images/hexo_blog/hexo_process.jpg)

1. hexo程序在本地计算机中，自己写好文章之后通过hexo程序生成静态的网页。
2. 通过hexo deploy服务将生成的静态网页push到远程仓库中 
3. 远程的服务器上运行着一个git的仓库，通过git hooks将git仓库中的文件checkout到网站的根目录中。
4. 远程的服务器中通过nginx运行网站。

# 搭建本地Hexo服务框架
Hexo程序需要Node环境，对于ubuntu系统来说，可以通过源来安装Node
```bash
sudo apt install nodejs
```
>这里不推荐这种方式，因为这样安装的node版本比较老，同时版本不好管理。
这里推荐用nvm管理node版本
## nvm安装方式
参考[github](https://github.com/nvm-sh/nvm#install--update-script)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```
>注意环境变量的设置。默认可能会将有关nvm的环境变量写在.bashrc中，如果你用的是zsh等，需要将相应的环境变量配置复制到相应的配置文件中，如.zshrc
## nvm更换源
由于众所周知的原因，直接访问国外的源速度比较慢，可以将下列配置添加到.zshrc中更换nvm源
```bash
export NVM_NODEJS_ORG_MIRROR="https://npm.taobao.org/mirrors/node"
```
## 使用nvm安装node
使用如下命令可以查看源中有哪些可安装的node版本
```bash
nvm list-remote
```
再通过下命令安装相应版本node
```bash
nvm install <version>
```
> 收到网络限制，可能加载不出来，可选择直接使用```nvm install stable```安装最新的stable版本
## 安装Hexo
参考如下命令
```bash
mkdir dirname //新建一个文件夹给hexo使用
npm install -g hexo-cli //安装hexo工具
hexo init
npm install //安装package.json中的工具
npm install hexo-deployer-git --save //远程部署插件
npm install hexo-server --save //本地服务器预览用
```

# 远程服务器配置
> 默认已安装Nginx 和 Git
## 博客网站文件夹
```bash
mkdir ~/www/hexo
sudo chown -R user:user ~/www/hexo //需要修改所有者为自己 
```
## 配置nginx的代理规则
```bash
cd /etc/nginx/conf.d  //这个文件夹中的配置被/etc/nginx/nginx.conf引用
sudo vim hexo.conf
```
在hexo.conf中输入下述配置
```
server{
    listen 80;
    server_name blog.com;
    location /{
        root /home/user/www/hexo;
        index index.jsp index.html index.htm;
    }
}
```
使用命令测试配置是否正确
```bash
sudo nginx -t -c /etc/nginx/nginx.conf
```
重启nginx
```bash 
sudo nginx -s reliad
```
## 配置远程git裸库
```bash
cd /home/user
git init --bare hexo_blog.git
sudo chown -R user:user hexo_blog.git //注意文件夹所有权和nginx部署的文件夹为同一个
```
## 配置HOOK
git hook的主要作用是监听到每一次push之后执行一段指令
```bash
cd /home/user/hexo_blog.git/hooks
cp ./post-update.sample ./post-update //注意文件名为post-update，不知道具体原因，猜测是根据文件夹名称决定在什么动作之后执行脚本
```
在post-update中输入下述指令
```
#!/bin/sh
git --work-tree=/home/user/www/hexo --git-dit=/home/user/hexo_blog.git checkout -f
```
修改脚本可执行权限
```bash
sudo chmod +x post-update
```
# 本地hexo文件配置
进入本地计算机的文件夹，修改_config.yml文件，修改其中的deploy
```
deploy:
  type: git
  repo: git@xxx.com:/home/user/hexo_blog.git
  branch: master
```
可以将文件夹初始化为git仓库，再任何一个地方pull下这个仓库即可重新写
## 第一次deploy
```bash 
hexo g -d
```
上述指令会自动编译部署

关于hexo具体的使用方法参考[hexo官网](https://hexo.io/docs/)

# Hexo + github pages
> 本地hexo环境配置和hexo文件配置如上


1. 在自己github中创建一个库，注意库的名字为github用户名.github.io，如desperado1999.github.io
2. 注意需要初始化README文件
3. 将github仓库输入到hexo的_config.yml文件中，再使用```hexo g -d```自动部署即可。如在_config.yml的deploy中写如下配置
```
deploy:
 type: git
 repo: git@github.com:desperado/desperado.github.io.git
 branch: master
```
4. 访问desperado.github.io即可访问自己博客。


