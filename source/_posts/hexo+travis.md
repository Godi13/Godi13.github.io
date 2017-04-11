---
title: 使用 Travis 自动部署 Hexo 到 Github 与 自己的服务器
date: 2017-04-09 14:07:35
categories:
- travis
- hexo
---

> 该文写给爱折腾的人，对纯新手不友善，没有辣么详细地说明如何安装node、git或者是hexo，所以关于这些问题可先行`google`，如有其它问题请在这里或者我的[简书](http://www.jianshu.com/p/1226d159d514)上留言

使用`hexo`而不使用`jekyll`在github上搭blog最大的问题就是，每次提交都需要先`hexo g`，然后再`push`生成的文件们，这样哪怕是改一个小的地方都需要重新编译全部blog，想想都蛋疼（虽然我还没有多少文章），所以决定使用[Travis](https://travis-ci.org/)来自动持续集成提交到github以后的操作

网上一搜有很多的教程，看了好多然后东拼西凑才最终成功，遂决定记录下来最终成功的这个方法，以免忘记

本文中实现的最终效果是：
- 写完blog后，直接`push`到github的`source`分支，其它的就可以不用管了
- 由于我的`.travis.yml`配置文件里设置监听的就是`source`分支，所以会触发`webhook`
- `Travis`则会将该项目`clone`过去，然后按照`.travis.yml`的设置执行接下来的命令
- 执行完成后，再将编译好的文件们发送到自己的服务器，顺便`push`回`master`分支上来
- 这样就可以在[blog.godi13.com](http://blog.godi13.com)和[Godi13.github.io](http://godi13.github.io/)上都访问blog了

# Github

首先，按规定名称`XXXXX.github.io`，其中`XXXXX`为你的用户名，如下图中的`Godi13.github.io`创建项目

![创建github项目](http://images.godi13.com/2017-04-10-githubio.png-total)

为了使`travis`能够将编译好的文件们`push`回咱们的github，我们需要生成`token`，步骤如下：

1) 点击右上方头像，然后点`setting`

![githubsetting](http://images.godi13.com/2017-04-10-setting.png-small
)

2) 点击`Personal access tokens`

![](http://images.godi13.com/2017-04-10-token.png-small
)

3) 点击`Generate new token`

![](http://images.godi13.com/2017-04-10-token2.png
-total)

4) 为`token`起一个名字，勾选`repo`，然后点击生成

![](http://images.godi13.com/2017-04-10-repo.png
-total)

<div class="tip">生成`token`以后，一定要复制好，因为只显示一次，如果丢失只能再次生成了</div>

![](http://images.godi13.com/2017-04-10-copytoken.png
)

# Travis

1) 使用`github`帐号登录[Travis](https://travis-ci.org/)，右上方按钮点击同步项目，下方打开需要集成的项目，最后点击齿轮进入项目配置页面

![在travis同步](http://images.godi13.com/2017-04-10-travisfirst.png
)

2) 打开`Build only if .travis.yml is present`，右下角的那个其实也可以关了，然后往下移动页面到环境变量

![](http://images.godi13.com/2017-04-10-present.png
)

3) 在这里我将变量名称名为`REPO_TOKEN`，放上`token`，点击`Add`按钮

![](http://images.godi13.com/2017-04-10-value.png
)

# Terminal

回到终端，进入`blog`所在的文件夹下，新建`.travis.yml`文件，并添加以下内容

```yml
# 使用语言
language: node_js
# node版本
node_js: stable
# 设置只监听哪个分支
branches:
  only:
  - source
# 缓存，可以节省集成的时间，这里我用了yarn，如果不用可以删除
cache:
  apt: true
  yarn: true
  directories:
    - node_modules
# tarvis生命周期执行顺序详见官网文档
before_install:
- git config --global user.name "Godi13"
- git config --global user.email "mqzq9388@gmail.com"
# 由于使用了yarn，所以需要下载，如不用yarn这两行可以删除
- curl -o- -L https://yarnpkg.com/install.sh | bash
- export PATH=$HOME/.yarn/bin:$PATH
- npm install -g hexo-cli
install:
# 不用yarn的话这里改成 npm i 即可
- yarn
script:
- hexo clean
- hexo generate
after_success:
- cd ./public
- git init
- git add --all .
- git commit -m "Travis CI Auto Builder"
# 这里的 REPO_TOKEN 即之前在 travis 项目的环境变量里添加的
- git push --quiet --force https://$REPO_TOKEN@github.com/Godi13/Godi13.github.io.git
  master
```

然后，准备`push`该项目到github，看下是否成功，如果是新项目可参照下面的git指令

```bash
git init
# 添加自己的项目
git remote add origin git@github.com:Godi13/Godi13.github.io.git
# 新建并切换分支
git checkout --orphan source
git add -A
git commit -m "Travis CI"
git push
```

> 关于 `--orphan` 请参考 [如何建立一個沒有 Parent 的獨立 Git branch](https://ihower.tw/blog/archives/5691)

如最终成功则会看到

![](http://images.godi13.com/2017-04-10-success.png
)

到这里关于hexo和github的事情就先吿一段落，接下来就是服务器与tarvis

# 服务器

按照上面`.tarvis.yml`的设置，其实我只需要在`after_success`这个生命周期中，把`public`文件夹下的所有文件传送到服务器指定的路径即可，接下来就是解决如何让`tarvis`往我的服务器传东西的问题了

在此之前，你需要将本机生成的密钥传到服务器上

```bash
ssh-keygen -t rsa # 然后一路回车即可
# ssh-copy-id 可能需要另行安装
# 如果ssh默认端口是22，则不需要 -p
ssh-copy-id <登录部署服务器用户名>@<部署服务器地址> -p <部署服务器ssh端口>
ssh <登录部署服务器用户名>@<部署服务器地址> -p <部署服务器ssh端口>
```

如果登录服务器成功，接下来需要安装`travis`命令行工具

```bash
# 安装travis命令行工具，如无法使用gem指令须先安装ruby
gem install travis
# --auto自动登录github帐号
travis login --auto
# 此处的--add参数表示自动添加脚本到.travis.yml文件中
travis encrypt-file ~/.ssh/id_rsa --add
# 这个命令会自动把 id_rsa 加密传送到 .git 指定的仓库对应的 travis 中去
```

执行完以后会发现在`travis`网站项目里面的环境变量里多了两个参数

![](http://images.godi13.com/2017-04-10-sshkey.png
)

并且在`.travis.yml`里的`before_install`周期中多了下面这2行

```bash
- openssl aes-256-cbc -K $encrypted_97d432d3ed20_key -iv $encrypted_97d432d3ed20_iv
  -in id_rsa.enc -out ~\/.ssh/id_rsa -d
```

默认生成的命令可能会在`/`前面带转义符`\`，我们不需要这些转义符，手动删掉所有的转义符，否则可能在后面引发莫名的错误

之后为了保证命令的顺利运行，我们还需要正确地设置权限和认证

```bash
before_install
- openssl aes-256-cbc -K $encrypted_97d432d3ed20_key -iv $encrypted_97d432d3ed20_iv
  -in id_rsa.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
- echo -e "Host 主机IP地址\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
```

最后，就是在`after_success`周期中，添加上传服务器的指令即可，在这里要注意，如果没有`stricthostkeychecking=no`参数，将构建失败，详细原因请参考[通过travis部署代码到远程服务器](http://blog.csdn.net/qq8427003/article/details/64921238)

```bash
# 由于我修改了默认的port，所以在这里也进行了加密处理
- scp -o stricthostkeychecking=no -P $PORT -r public/* 用户@域名:/路径
```

但使用`scp`有很多问题，所以后来我决定改用`rsync`命令，缺点是端口号就不能用travis环境来加密了，如果哪个朋友有更好的方案希望能告诉我

```bash
# public 后面加上/即可将该目录下的文件都传送到服务器了
- rsync -rv --delete -e 'ssh -o stricthostkeychecking=no -p 端口号' public/ 用户@域名:/路径
```

最后在这里放上我最终的[.travis.yml](https://github.com/Godi13/Godi13.github.io/blob/source/.travis.yml)配置参数，希望本文对大家能有所帮助

# 参考资料

- [Hexo作者的.travis.yml配置](https://github.com/tommy351/tommy351.github.io/blob/source/.travis.yml)
- [用 Travis CI 自動部署網站到 GitHub](https://zespia.tw/blog/2015/01/21/continuous-deployment-to-github-with-travis/)
- [一点都不高大上，手把手教你使用Travis CI实现持续部署](https://zhuanlan.zhihu.com/p/25066056)
- [使用 Travis CI 自动更新 GitHub Pages](http://notes.iissnan.com/2016/publishing-github-pages-with-travis-ci/)
- [使用Travis CI自动构建hexo博客](http://magicse7en.github.io/2016/03/27/travis-ci-auto-deploy-hexo-github/)
