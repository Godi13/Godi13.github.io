---
title: 使用 Travis 将 GitHub 文件上传传至服务器
date: 2017-04-17 13:19:55
categories:
- travis
- server
---

同一天两个同学问我一样的问题，所以我决定详细的记录一下如何使用[Travis](https://travis-ci.org)把`GitHub`里的文件传至自己的服务器

# server

首先要确保自己可以通过`ssh`命令免密登录自己的服务器，这就需要先生成密钥对

```bash
ssh-keygen
```

输入上面的指令以后一路回车即可，你会发现在用户根目录下多了`.ssh`目录，进去看一下`cd ~/.ssh`，里面有这3个文件

![](http://images.godi13.com/2017-04-15-ssh.png-small)

接下来把`id_rsa.pub`里的内容，手动复制到服务器的`~/.ssh/authorized_keys`(如没有可自行创建)中去即可

![](http://images.godi13.com/2017-04-17-serverrsa.png)

> 还有一种方法是使用`ssh-copy-id root@IP`命令，Mac用户可能需要用`brew`安装一下`ssh-copy-id`，ubuntu用户应该是自带的这个命令，实现的效果与上面手动的一样，更多ssh使用方法请参考[介绍 ssh 的日常使用](http://haoduoshipin.com/v/62.html)

登录服务器的指令如下，如果不需要密码便进入则表示成功

```bash
ssh root@服务器的IP地址
# 如改变过端口则为
ssh -p 端口号 root@服务器的IP地址
```

# GitHub

在GitHub上新建个项目，名字自拟

![](http://images.godi13.com/2017-04-17-createrepo.png-total)

进入你所要上传的项目，输入一下指令

```bash
git init
git remote add origin https://github.com/Godi13/TravisSendToServer.git
```

# Travis

登录[Travis](https://travis-ci.org)，如果没有找到新建项目的话，点击`Sync account`

![](http://images.godi13.com/travishome.png)

先点开开关，然后点击齿轮，进入`Setting`页面

![](http://images.godi13.com/2017-04-17-travissetting.png)

这里可以这么设置

![](http://images.godi13.com/2017-04-17-travisgeneral.png)

# 配置文件

在本地项目中添加`.travis.yml`文件，先只加入一下配置

```yml
language: node_js
node_js: stable
```

接下来使用`Travis`命令行工具将`id_rsa.pub`加密，同时将环境变量传至`Travis`

```
# 安装travis命令行工具，如无法使用gem指令须先安装ruby
gem install travis
# --auto自动登录github帐号
travis login --auto
# 此处的--add参数表示自动添加脚本到.travis.yml文件中
travis encrypt-file ~/.ssh/id_rsa --add
# 这个命令会自动把 id_rsa 加密传送到 .git 指定的仓库对应的 travis 中去
```

<div class="tip">本次执行完`travis encrypt-file ~/.ssh/id_rsa --add`报了下图这个错误，是网路问题，终端翻墙即可
![](http://images.godi13.com/2017-04-17-traviserror.png)</div>

执行完以后会发现在`Travis`网站项目里面的`Setting`中的环境变量里多两个参数

![](http://images.godi13.com/2017-04-10-sshkey.png)

并且在`.travis.yml`里的`before_install`周期中自动多了下面这2行

```bash
- openssl aes-256-cbc -K $encrypted_97d432d3ed20_key -iv $encrypted_97d432d3ed20_iv
  -in id_rsa.enc -out ~\/.ssh/id_rsa -d
```

默认生成的命令可能会在`/`前面带转义符`\`，我们不需要这些转义符，手动删掉所有的转义符，否则可能在后面引发莫名的错误

之后为了保证命令的顺利运行，我们还需要正确地设置权限和认证，注意第三行`主机IP地址`那里写自己服务器的IP

```bash
before_install
- openssl aes-256-cbc -K $encrypted_97d432d3ed20_key -iv $encrypted_97d432d3ed20_iv
  -in id_rsa.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
- echo -e "Host 主机IP地址\n\tStrictHostKeyChecking no\n" >> ~/.ssh/config
```

最后，就是在`after_success`周期中，添加上传服务器的指令即可，在这里要注意，如果没有`stricthostkeychecking=no`参数，将构建失败，详细原因请参考[通过travis部署代码到远程服务器](http://blog.csdn.net/qq8427003/article/details/64921238)

```bash
# 没有修改过端口的，可以用这个，上传目录要加 -r 参数
- scp -o stricthostkeychecking=no -r 要上传的文件或目录 用户@域名或IP:/路径
# 由于我修改了默认的port，所以在这里也进行了加密处理
- scp -o stricthostkeychecking=no -P $PORT -r 要上传的文件或目录 用户@域名或IP:/路径
```

最最后上传所有文件到GitHub

```bash
git add .
git commit -m "Travis Send to Server"
git push
```

在`Travis`查看log，显示成功上传

![](http://images.godi13.com/2017-04-17-travisdone.png)

在这里可以看到我的配置文件[项目地址](https://github.com/Godi13/TravisSendToServer)，欢迎大家的反馈，希望能有所帮助
