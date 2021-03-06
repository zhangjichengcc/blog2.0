title: Travis CI + github + hexo 自动化部署
categories:
  - 随笔
tags:
  - git
  - hexo
  - Travis
  - 部署
date: 2019-01-08 14:24:00
---
>[Travis CI](https://travis-ci.org/)是目前新兴的开源持续集成构建项目，采用`yaml`格式，简洁清新独树一帜。目前大多数的github项目都已经移入到Travis CI的构建队列中。Travis-CI会同步你在GitHub上托管的项目，每当你`commit` `push`成功之后，就可以根据配置文件进行项目的构建发布。

>本博客最开始采用手动部署，后来采用jenkins部署，但是配置步骤相对繁琐，后来发现了Travis CI，基于github刚好符合我的需求，并且配置起来十分方便，本文记录了配置Travis的全过程，构建步骤为：
>1. 本地开发完成，提交代码到github仓库;
2. github收到提交的更新，通知Travis;
3. Travis 收到github的提交通知，进行构建;

<!-- more -->

hexo 的安装使用本文就不做介绍了，可以参考之前的文章[hexo常用命令](https://blog.veigar.xyz/2018/02/05/hexo%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A4/),[hexo 创建文章 & 文章缩略图及banner & MarkDown
](https://blog.veigar.xyz/2018/02/27/hexo-%E5%88%9B%E5%BB%BA%E6%96%87%E7%AB%A0/)

## 注册配置 Travis
1. 打开[Travis CI](https://travis-ci.org/)官网，进行注册，这里就不做太多赘述，推荐用github账户注册；

![Travis CI注册](travis.jpg)

2. 绑定你的github账户，此时Travis会同步你的github仓库，将你要监听的github仓库名选中，此时Travis会监听该仓库的push操作，并执行指定的脚本文件；

![Travis CI设置](travis2.jpg)

## 添加 .travis.yml
当我们提交代码后执行的一系列操作都是在 .travis.yml文件中配置的；

``` bash
language: node_js  #設置語言
node_js: stable  #設置相應的版本
cache:
    apt: true
    directories:
        - node_modules # 緩存不經常更改的內容
install:
  # 执行安装操作
  ...
script:
  # 开始部署
  ...
after_script:
  # 部署后操作
  ...
branches:
  # 配置监听的分支
  only:
    - master #只監測master分支，master是我的博客源碼分支的名稱，可根據自己情況設置
env:
  # 环境变量配置
  global:
    - GH_REF: github.com/***/blog.git #設置GH_REF，注意更改***
```

更详细的参数配置可以参考官网
如下是我的配置信息:
``` bash
language: node_js  #設置語言

node_js: stable  #設置相應的版本

cache:
    apt: true
    directories:
        - node_modules # 緩存不經常更改的內容

before_install:
  - echo 安装hexo相关环境...

install:
  # - npm install -g cnpm --registry=https://registry.npm.taobao.org
  - cnpm install

before_script:
  - echo 正在清空缓存静态文件...
  - hexo clean                      # 清除缓存静态文件
  - echo 正在生成静态文件...
  - hexo g                          # 生成静态文件
  - cd ./public
  - ls -l

script:
  - echo 开始部署...
  - git init
  - git config --global user.name "${GH_username}"                          # 修改name
  - git config --global user.email "${GH_useremail}"                        # 修改email
  - git add ./
  - git commit -m "update"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master  # GH_TOKEN是在Travis中配置token的名稱

after_script:
  - echo 部署完成！

branches:
  only:
    - master                                                                # 只監測master分支，master是我的博客源碼分支的名稱，可根據自己情況設置
env:
  global:
    - GH_REF: github.com/<prourl>                                           # 设置 github 项目仓库地址
    - GH_username: <yourname>                                               # 设置 github 用户名           
    - GH_useremail: <youremail>                                             # 设置 github 绑定邮箱地址

```

---

## 生成github access Token 并 配置到 Travis 
>此步骤是为了使travis获得对github的操作权限，如`git push`等

### 生成github access Token

![github access Token](travis3.jpg)

1. 如上图所示，登陆github并打开该页面，并新建token

![github access Token](travis4.jpg)

2. 如上图进行对应操作，生成token，注意token只显示一次，要保存好备用。

### 配置 access token 到 Travis

打开Travis CI 找到setting页面，填写对应的token名及上面步骤生成的token值，如下图：

![Token配置](travis5.jpg)

细心的同学可能会发现我的 .travis.yml 文件中有下面这样一段配置

``` bash
after_script:
  - echo 开始部署...
  - cd ./public
  - git init
  - git config --global user.name "yourname"  #修改name
  - git config --global user.email "youremail"  #修改email
  - git add ./
  - git commit -m "update"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master  #GH_TOKEN是在Travis中配置token的名稱
  - echo 部署完成！
branches:
  only:
    - master #只監測master分支，master是我的博客源碼分支的名稱，可根據自己情況設置
env:
  global:
    - GH_REF: github.com/yourname/bolg.git #設置GH_REF，注意更改yourname
```
其中有两个变量，GH_REF是在env中配置的，而GH_TOKEN则是我们刚刚在设置中添加的github token，此时执行`git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master`命令就可以在不用输入用户名密码的情况下进行提交。

---

## 测试
1. 提交代码到github中

![代码提交](travis6.jpg)

2. 查看部署情况
![查看部署情况](travis7.jpg)

![查看部署情况](travis8.jpg)