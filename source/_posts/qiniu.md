---
title: 使用七牛 + GitHub + Hexo + Travis搭建静态blog
date: 2017-04-15 05:32:26
categories:
- 七牛
- hexo
- travis
---

其实如果你只是想搭个静态Blog，完全不需要买服务器，直接使用GitHub即可，网上有很多的方法。但是并不是所以地区访问GitHub都那么快和稳定，而且据我亲测国内有的地方还真上不了GitHub。那如果想搭个自己域名并且访问速度既快又稳定的还几乎免费的Blog，该怎么办？直到我在七牛看到了这个

![](http://images.godi13.com/2017-04-15-indexsetting.png)

七牛不但有以上所说优点，而且处理图片大小、加个水印之类的着实方便。而且七牛还提供了命令行工具[qshell](https://developer.qiniu.com/kodo/tools/1302/qshell)，那接下来的事情就简单了。我的构想是，写完Blog提交到GitHub，触发hook，Travis将项目生成，上传到七牛即可

# 准备

- 自己的域名
- GitHub帐号
- 七牛帐号，并且往里面冲10元钱(七牛规定启用CDN加速最少充值10元)，所以前面说是几乎免费

# 七牛

首先创建一个储存空间，步骤如下

![](http://images.godi13.com/2017-04-15-createbucket.png)

在七牛冲完值以后，就可以新建加速域名了

![](http://images.godi13.com/2017-04-15-cdn.png)

我选择的是范域名，这样只需要设置一次CNAME，其它的泛子域名就可以自己随意添加了，很方便，你也可以根据自己的情况只加速一个普通域名

设置完之后需要等一阵子，具体是多久我忘记了，这个位置会有提示，我的这个是已经完成范域名的加速了

![](http://images.godi13.com/2017-04-15-status.png)

鼠标点击那个CNAME，会出现CNAME地址，复制一下，然后去DNS解析设置

![](http://images.godi13.com/2017-04-15-cname.png-total)

我的域名是在阿里万网购买的，大家自行在自己的DNS服务商那添加

![](http://images.godi13.com/2017-04-15-dns-qiniu.png)

主机记录我写的是`qiniu`，跟我下图中的泛子域名`qiniu.godi13.com`是对应的，你也可以自行设置，CNAME记录值就是复制的那个

![](http://images.godi13.com/2017-04-15-test.png)

<div class="tip">最后，别忘了在`空间设置`里面将`默认首页设置`开启，就是这篇Blog第一张图那里，然后你可以上传一个`index.html`测试用你设置的域名登录一下，看是否有效，如果没有问题，继续下一步</div>

# Travis

> 如果对hexo配合travis不了解，建议先看一下[使用 TRAVIS 自动部署 HEXO 到 GITHUB 与 自己的服务器](http://blog.godi13.com/2017/04/09/hexo+travis/)

首先在GitHub上创建一个项目

![](http://images.godi13.com/2017-04-16-createrepository.png-total)

在本地项目中下载`qsehll`，将来一起上传到GitHub上

```bash
curl -O http://devtools.qiniu.com/qshell-v2.0.5.zip
# 如果没安装zip请先安装或者手动解压
unzip qshell-v2.0.5.zip
# 留下linux版本
mv qshell_linux_amd64 qshell
# 删除其它
rm qshell_* qshell-*
# 确保有执行权限
chmod +x qshell
```

添加上传七牛的配置文件，名称自拟，为了配合七牛的指令[qupload](https://github.com/qiniu/qshell/blob/master/docs/qupload.md?ref=developer.qiniu.com)，具体参数含义请看文档

```
{
  "src_dir"            :   "/home/travis/build/GitHub帐号名/创建的项目名称/public",
  "bucket"             :   "你要上传的七牛空间名称",
  "ignore_dir"         :   false,
  "overwrite"          :   true,
  "check_exists"       :   true,
  "check_hash"         :   true,
  "check_size"         :   true,
  "rescan_local"       :   true,
  "skip_file_prefixes" :   "test,demo,",
  "skip_path_prefixes" :   "hello/,temp/",
  "skip_fixed_strings" :   ".svn,.git",
  "skip_suffixes"      :   ".DS_Store,.exe",
  "log_file"           :   "upload.log",
  "log_level"          :   "info",
  "log_rotate"         :   1,
  "log_stdout"         :   false
}
```

在项目中添加`.tarvis.yml`文件，具体内容如下

```yml
language: node_js
node_js: stable
branches:
  only:
  - master
cache:
  apt: true
  yarn: true
  directories:
  - node_modules
before_install:
# AK SK将来会设置到travis的环境变量中
- ./qshell account $AK $SK
- curl -o- -L https://yarnpkg.com/install.sh | bash
- export PATH=$HOME/.yarn/bin:$PATH
- npm install -g hexo-cli
install:
- yarn
script:
- hexo clean
- hexo generate
after_success:
# 为上传七牛的配置文件名称，这里我起名为config
-  ./qshell qupload config
```

点击七牛右上角的个人面板，找到自己的密钥

![](http://images.godi13.com/2017-04-16-key.png-total)

将这两个密钥复制

![](http://images.godi13.com/2017-04-16-key2.png)

放到travis里这个项目的setting里的环境变量中去（如果在travis首页里没有看到自己新建的项目，就手动点击一下同步）

![](http://images.godi13.com/2017-04-16-travisvalue.png)

然后在终端进入你要上传的hexo项目中去，执行以下命令

```bash
git init
git remote add origin https://github.com/用户名/刚才起的项目名字.git
git add .
git push
```

<div class="tip">如果上传成功看到此图可能是因为没有打开空间设置里面的默认首页设置，如果开启了稍等一会刷新一下就应该可以了
![](http://images.godi13.com/2017-04-16-error.png)</div>

最后在这里放一下我的配置文件[config](https://github.com/Godi13/Qiniu-Hexo-Travis/blob/master/config)和[.travis.yml](https://github.com/Godi13/Qiniu-Hexo-Travis/blob/master/.travis.yml)

<div class="tip">提醒大家注意看`travis`的`log`，如果用实在不行，那就手动使用`hexo`跟`qshell`吧，此文只是提供一个思路，感谢大家的阅读~</div>
