---
title: 基于hexo搭建博客
date: 2017-06-15 10:42:18
tags: blog
---
## 安装hexo
### 安装nodejs环境
```
sudo apt-get install -y nodejs
sudo apt install nodejs-legacy
```
### 安装hexo
```
sudo npm install hexo-cli -g
```
### 初始化blog目录
```
hexo init name.github.io/
```
<!-- more -->
### 生成静态页面，需要进入blog目录
```
hexo generate
```
### 预览效果
```
hexo server
```
### 安装git 插件
```
sudo npm install hexo-deployer-git --save
```
### 安装搜索
```
sudo npm install hexo-generator-searchdb --save
```
### 配置RSS
```
sudo npm install hexo-generator-feed --save
```
### 添加文章的步骤
```
hexo new "第一篇博客"
#edit source/_posts/第一篇博客.md
hexo clean
hexo generate
hexo deploy
```
### 管理hexo的生成文件的目录，注意在push之前需要把文件都提交
```
git init
git remote add origin https://github.com/name/blog.git
git push -u origin master
```
### 修改_config.yml添加配置
```
# 添加发布的路径
deploy:
  type: git
  repo: https://github.com/name/name.github.io.git
  branch: master
# 添加多说和百度统计
# 多说停了。。
# duoshuo_shortname: name
# duoshuo_share: true
baidu_analytics: ec30a044f2f69aacc83c0b70ea53e01a

search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
## 安装主题
### 安装nexT主题
```
cd hexo_blog_dir
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
### 添加tag
```
hexo new page tags
```
### 添加分类
```
hexo new page categories
```
### 添加关于
```
hexo new page about
```
### 文章内容的手工截断
```
<!-- more -->
```
