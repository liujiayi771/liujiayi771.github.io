---
title: 使用Hexo与Github搭建个人博客
date: 2015-12-09 21:18:47
tags: hexo
---
#### 1. 安装Hexo
- 安装git直接使用指令：
```java
sudo apt-get install git
```

- 安装Node.js直接使用如下指令：
```java
sudo apt-get install -y python-software-properties software-properties-common
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
```

- 安装好Git和Node.js之后，使用如下指令安装Hexo：
```java
sudo npm install -g hexo-cli
```
<!--more-->
#### 2. 创建存放博文的文件夹
- 在任意位置创建文件夹，例如在"~/文档/"目录下创建文件夹hexo：
```java
sudo mkdir ~/文档/hexo
```
- 执行下列命令，Hexo会在指定文件夹中新建所需要的文件：
```java
sudo hexo init <folder>
cd <folder>
sudo npm install
```
- 新建完成后，制定文件夹的目录如下：
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
- _config.yml存放配置信息
- source/_posts/目录下存放博文markdown文件
- theme/目录用于存放主题文件夹

#### 3. 配置
- 在_config.yml中修改大部分配置
- title 网站标题
- subtitle 网站副标题
- author 你的名字
- language 语言(默认为英文，简体中文为zh-CN，繁体中文为zh-TW)
- 具体参照 https://hexo.io/zh-cn/docs/configuration.html

#### 4. 主题使用
- 在 https://hexo.io/themes/ 中找到想要使用的主题，进入其github主页，里面有主题的具体安装方法，一般是git到目录下的theme文件夹当中去，然后在配置文件中修改主题名称

#### 5. 部署到github
- 在github上new repository，名称为:[github用户名].github.io
- 在_config.yml添加如下代码：
```java
deploy:
  type: git
  repo: https://github.com/liujiayi771/liujiayi771.github.io.git
```
- repo后为仓库的https地址
- 使用命令创建名称为title的博文markdown文件：
```java
sudo hexo new <title>
```
- 在markdown文件中写好内容后，使用如下命令生成html文件：
```java
sudo hexo g
```
- 使用如下命令将生成的静态网页内容上传到github上去：
```java
sudo hexo d
```
- 可通过访问[github用户名].github.io来访问博客主页
