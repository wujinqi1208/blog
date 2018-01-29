---
title: Ubuntu + Hexo博客搭建
---

# Hexo github 博客搭建

## 搭建nodejs

``` bash
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo apt-get install -y npm
```

## 搭建hexo

``` bash
npm install hexo-cli -g
hexo init blog
cd blog
npm install
```

## hexo常用命令

``` bash
hexo g    #生成静态网页
nohup hexo server -p 80 &    #运行本地服务器
hexo clean    #
hexo d    #部署
```

## 安装Git

``` bash
sudo apt-get install -y git

git config --global user.name "mark"
git config --global user.email "w_jinqi1208@163.com"
```
## 更换主题

```
git clone https://github.com/iissnan/hexo-theme-next themes/next

cd themes/next
git pull
```
