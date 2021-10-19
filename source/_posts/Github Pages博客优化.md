---
title: Github Pages 博客优化
date: 2021-09-03
tags:
---

# 使用Github Action自动部署

Github Action是Github提供的免费CI系统，利用Git Hooks可以在提交代码时触发自动化构建流程。对于我使用hexo搭建的博客来说，在我写完博客后push到Github就能触发自动hexo deploy发布博客。

<!-- more -->

在项目的`.github/workflows/`创建一个yaml配置文件，也可以在Github Action页面创建该文件：

![截屏2021-09-03 21.39.39](https://i.loli.net/2021/09/03/2mF7YHMbaug39d4.png)

然后在配置文件中定义自动化的`steps`：hexo博客需要安装Node.js环境，然后npm install下载依赖，执行hexo deploy即可。我这里使用了actions-gh-pages这个插件，可以定义部署的Git仓库和分支等信息。

我之前的博客是在`marwincn/blog`代码仓的`gh-page`分支部署的Github Pages，但是Github有个限制就是非username.github.io仓库部署的Github Pages发布的地址会带上仓库名的相对路径，如`https://marwincn.github.io/blog/`。为了去掉`/blog/`这个相对路径，我还是决定将博客发布到`marwincn/marwincn.github.io`代码仓。

actions-gh-pages插件部署的背后就是将渲染后的静态页面push到代码仓，要成功push就要获取代码仓的push权限，对于自动化脚本来说最简单且安全的方法是使用token。首先在Github设置页面生成一个token：

![截屏2021-09-03 21.56.18](https://i.loli.net/2021/09/03/8G9LHgOE4uwfVvQ.png)

然后将此token设置到配置了Github Action的代码仓的环境变量中，在ymal配置中可以通过`${{ secrets.PERSONAL_TOKEN }}`访问该环境变量：

![截屏2021-09-03 21.59.49](https://i.loli.net/2021/09/03/qjGPfV8l5LeJNvH.png)

最后给出我的yaml配置：

```yaml
name: Node.js CI
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14.x'
        cache: 'npm'
    - name: Install dependencies
      run: npm install
    - name: Build
      run: npm run build
    - name: Deploy
      uses: peaceiris/actions-gh-pages@v3
      with:
        personal_token: ${{ secrets.PERSONAL_TOKEN }}
        external_repository: marwincn/marwincn.github.io
        publish_branch: main
        publish_dir: ./public
        cname: marwin.cn
        commit_message: ${{ github.event.head_commit.message }}

```

最后实现的效果，写完博客直接git push，然后Github Action自动部署静态页面到`marwincn/marwincn.github.io`代码仓，然后更新了博客页面：

![截屏2021-09-03 22.07.23](https://i.loli.net/2021/09/03/HZVg8YOrvGB5Mwk.png)

# CNAME到自定义域名

如果你正好有个域名，那么可以将该域名CNAME到Github Page实现通过自定义域名访问博客。

首先在拥有域名的服务商对域名进行CNAME解析：

![截屏2021-09-03 22.12.23](https://i.loli.net/2021/09/03/JMwtRoPNl6UmCSH.png)

想要实现CNAME单方面的解析还不够，需要在被解析的域名下放置一个CNAME文件，可以在hexo渲染时就带上CNAME文件，我这里使用了actions-gh-pages插件，添加`cname: marwin.cn`就可以同时发布一个CNAME文件：

![截屏2021-09-03 22.17.53](https://i.loli.net/2021/09/03/n3NsZ1WECXm6BUb.png)

最终就可以使用自己的域名访问你的博客了。

# Google Analytics博客访客分析

Google Analytics提供免费的网站访客分析，可以在官网https://analytics.google.com/创建媒体资源和应用，根据提示创建跟踪网站的代码：

![截屏2021-09-03 22.26.34](https://i.loli.net/2021/09/03/Kf4XBSNAYokr6ud.png)

将这段JavaScript标签放到hexo生成静态页面的模板中，hexo渲染生成的每个页面就会带上这段标签，当用户访问你的博客时就会给Google发送请求记录这次访问：

![截屏2021-09-03 22.33.04](https://i.loli.net/2021/09/03/znxVoOrXEmatbP2.png)

使用Google分析的缺点就是国内用户可能无法访问Google而无法统计到，当然也可以用百度统计来统计访客，但是百度统计的体验并不好，而且Github Pages屏蔽了百度的爬虫，对于程序员群体来说使用Google应该是比较基本的操作，所以使用Google分析挺好的，最终效果如下：

![截屏2021-09-03 22.40.09](https://i.loli.net/2021/09/03/XmG3zKDEae64O9g.png)
