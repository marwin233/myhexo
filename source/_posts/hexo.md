---
title: 使用hexo搭建博客
date: 2020-02-05 20:13:41
tags:
---
## 背景

本来在自己的vps上自己写了一个博客，但是很多地方需要自己不断维护，自己写的前端也不太满意所以决定用框架搭一个静态博客部署在github page上，这样也省了在vps上到处迁移。

我选择了人气比较高的Hexo，记录一下自己搭建的过程，作为官方文档的整理和补充。

## 安装

首先确认电脑里有git和npm （安装node.js即可）的支持，以下为macOS上使用homebrew安装，其他安装方式参考搜索引擎。

```shell
# 安装git
$ brew install git
# 安装node.js
$ brew install node
```

### 安装Hexo

```shell
$ npm install -g hexo-cli
```

安装后可以使用hexo命令行，命令文档：[文档](https://hexo.io/zh-cn/docs/commands)。

### 新建项目

```shell
$ hexo init <folder>
$ cd <folder>
$ npm install
```

新建完成后，指定文件夹的目录如下：

```shell
.
├── _config.yml     //项目配置文件
├── package.json    //应用程序的信息
├── scaffolds       //模版文件夹
├── source          //存放用户资源和md文件的地方
|   ├── _drafts
|   └── _posts
└── themes          //主题文件夹
```

修改`_config.yml`为自己的信息，参考配置文档：[文档](https://hexo.io/zh-cn/docs/configuration)。

### 安装主题

建立项目后，第一件事可能是找一个喜欢的主题，在[主题市场](https://hexo.io/themes/)找到自己满意的主题进行接下来的安装，当然也可以自己写一个主题。

找到选好的主题的github项目，将其clone到你新建的项目的themes目录下，这里的默认的一个主题是landscape。修改项目根目录下的`_config.yml`（项目配置文件，一般主题目录下也有一个`_config.yml`，请勿混淆），theme的值改为刚才clone的主题名称。到这里就可以将主题应用到项目上，其他配置参考每个主题的文档。

## 部署

根据github的规定，仓库名为`<your github name>.github.io`时默认该仓库的master分支为github page部署分支，此时页面的url为`https://<your github name>.github.io/index.html`。而其他命名的仓库可以指定一个分支，如gh-pages作为github page部署分支，此时页面的url为`https://<your github name>.github.io/repositorie-name/index.html`。

由于这种差异，可以有两种部署方式。一是将hexo项目在本地渲染静态页面，然后push到远程；二是将项目代码同步到github仓库，使用CI将远程渲染后的结果发布到gh-pages分支。

### 本地渲染

由于是部署到github page，所以使用git发布到远程。按照文档，可以使用SFTP等方式发布到自己的服务器上，也可以设置多个发布地址，按顺序发布。

安装hexo-deployer-git：

```shell
$ npm install hexo-deployer-git --save
```

修改`_config.yml`的参数：

```shell
deploy:
  type: git
  repo: <repository url> #git@github.com:xxxx/xxxx.github.io.git
  branch: [branch]
  message: [message]
```

需要提前将本地ssh pubkey添加到github的`SSH keys`里，作为push命令的身份认证。然后只需一条命令就能将网站部署到服务器上：

```shell
$ hexo clean && hexo deploy # clean渲染，delpoy部署
```

### 远程渲染

参考官方文档：[github-pages](https://hexo.io/zh-cn/docs/github-pages)

