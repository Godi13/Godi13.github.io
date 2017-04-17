---
title: 阿里云服务器从购买到配置全攻略
date: 2017-04-14 18:37:05
categories:
- server
---

我用的MAC，终端是`iTerm2`，从本地链接到服务器的最终效果图如下

![](http://images.godi13.com/2017-04-14-final.png-total)

> `ssh_godi13`是我在`.zshrc`里设置的`alias`，实际指令是`ssh -p PORT root@IP`

好，现在就从购买开始讲起（购买的步骤可能会跟我的略有差异，有可能因为阿里又更新了UI）

# 购买阿里云

登录[阿里云](https://www.aliyun.com/)，注册一个帐号，进入控制台(如已登录进去可忽略)

![](http://images.godi13.com/2017-04-14-login.png-total)

如出现该页面选择一个对应的然后点确定

![](http://images.godi13.com/2017-04-15-login2.png)

选择`云服务器ECS`，点击那个小购物车进入购买页面

![](http://images.godi13.com/2017-04-15-buy.png)

我选择的参数如下

![](http://images.godi13.com/2017-04-15-choose1.png-total)

我都选的最低配置

![](http://images.godi13.com/2017-04-15-choose2.png-total)

按量的相对便宜，带宽也高，我没什么访问量所以选的这个

![](http://images.godi13.com/2017-04-15-choose3.png-total)

感觉一次买3年的比较合适，我买的时候是3年800，而且当时有用100的优惠卷，现在不知道多少钱了，不过阿里经常搞活动，买之前可以留意一下

![](http://images.godi13.com/2017-04-15-choose4.png)

最后别忘了设置一个登录密码

![](http://images.godi13.com/2017-04-15-choose5.png-total)

最后支付完成，购买服务器的部分就到这里，接下来去整一个域名

# 购买域名与设置DNS解析

进入[阿里万网](https://wanwang.aliyun.com/domain/)选择一个自己喜欢的域名购买，购买完域名不要忘了去备案，在阿里云控制面板的这个位置有

![](http://images.godi13.com/2017-04-15-beian.png)

我当时是需要用阿里指定的背景照个半身像跟身份证正反面发过去即可，现在不知道是不是这样了，有可能需要去当地指定地点拍照

点击`运行中`进入查看所购买的实例

![](http://images.godi13.com/2017-04-15-ecs.png)

将IP地址记录下来，一会需要用到

![](http://images.godi13.com/2017-04-15-ip.png)

点击云解析，然后点击刚才购买的域名进入DNS解析页面

![](http://images.godi13.com/2017-04-15-dns1.png)

如图添加主机记录`www`与刚才记录的IP地址，还可以添加个`test`主机记录的，一会测试用

![](http://images.godi13.com/2017-04-15-dns2.png)

接下来该去登录服务器配置一下了

# 服务器

## 服务器免密登录

我是`Mac`用户，为了以后每次登录服务器不需要输入密码，我们需要使用`ssh`协议来登录

首先在客户端终端输入`ssh-keygen`，一路回车即可。然后你会发现在用户根目录下多了`.ssh`目录，进去看一下`cd ~/.ssh`，里面有这3个文件

![](http://images.godi13.com/2017-04-15-ssh.png-small)

把`id_rsa.pub`里的内容，手动复制到服务器的`~/.ssh/authorized_keys`中去即可

```bash
# 登录服务器
ssh root@服务器的IP地址
```

还有一种方法是使用`ssh-copy-id root@IP`命令，`Mac`用户可能需要用[brew](https://brew.sh/)安装一下`ssh-copy-id`，`ubuntu`用户应该是自带的这个命令，实现的效果与上面的一样，更多ssh使用方法请参考[介绍 ssh 的日常使用](http://haoduoshipin.com/v/62.html)

完成以后，再登录服务器就不需要输入密码了，接下来我们进行一下简单的安全配置，你也可以忽略这些步骤

## 简单的安全配置

### 修改默认端口号，取消密码登录

登录到服务器以后，`vim /etc/ssh/sshd_config`修改一下ssh的配合

```
Port 22 //默认是22，修改为自定义端口号
...
...
...
PasswordAuthentication no // 一般在最后一行,改为 no，不允许密码登录
```

`service ssh restart` 重启生效

<div class="tip">如果发生手残在服务器端删除了`.ssh`文件或者类似的情况，可以到阿里云上使用远程管理来拯救。远程登录后，把`PasswordAuthentication`值改回`yes`即可密码登录
![](http://images.godi13.com/2017-04-15-remote.png)</div>

### 配置防火墙

1. `ufw enable` 开启防火墙
2. `ufw default deny` 禁止所有端口访问
3. `ufw allow 80/tcp` 允许80端口tcp协议链接
4. `ufw allow 443/tcp` 443 https
5. `ufw allow 修改的sshd_config的Port的端口号/tcp`
6. `ufw status` 查看防火墙状态
7. `ufw reload` 重启防火墙

## Nginx

### 安装 nginx

1. `apt-get update`
2. `apt-get install nginx`
3. `service nginx status` 查看状态
4. 如果成功，浏览器中输入IP即可显示nginx默认页面

### 配置 nginx

1. nginx默认会把`/etc/nginx/conf.d`目录下的配置全部引入，下图是`nginx.conf`里默认配置

![](http://images.godi13.com/2017-04-15-nginxdefault.png-small)

2. `cd /etc/nginx/conf.d`，创建`自己起个名字.conf`，输入以下内容

```
server {
  listen 80;

  server_name  www.域名.com;
  # server_name  *.域名.com;
  # server_name  www.域名.com test.域名.com;

  location / {
    # 路径自己定，不过不能放到/root目录下
    root   /usr/local/src;
    index  index.html index.htm;
  }
}
```

3. `/etc/init.d/nginx restart` 重启nginx
4. 在`/usr/local/src`创建一个`index.html`
5. 输入域名登录，如果成功则显示`index.html`里面的内容

你要可以在`/etc/nginx/conf.d`目录下，多写几个不同的配置，分开管理二级域名

## 一些好的工具

为了统一客户端与服务端的操作习惯，我在服务器端也安装了`oh-my-zsh`和`z`，并把界面调整一致，想把服务器玩的更6的可以安装`tmux`，这里我没有安装就先不讲了

### zsh

如何安装可以参考[Ubuntu 下安装oh-my-zsh](http://www.jianshu.com/p/9a5c4cb0452d)，我记得阿里云的ubuntu里好像默认有`zsh`

我皮肤用的也是`oh-my-zsh`里的`agnoster`主题，如果想把`user@hostname`信息隐藏跟客户端的设施略有不同。客户端想隐藏只需要在`~/.zshrc`文件中添加即可

    DEFAULT_USER=`whoami`

但服务器默认是`root`用户，此法不通，需要在`cd ~/.oh-my-zsh/themes/agnoster.zsh-theme`里把最下面的`context`注释掉即可

```
...
## Main prompt
build_prompt() {
  RETVAL=$?
  prompt_status
  prompt_virtualenv
  # prompt_context
  prompt_dir
  prompt_git
  prompt_bzr
  prompt_hg
  prompt_end
}
```

### z

z是类似autojump的文件跳转工具，会记录你的目录习惯，就像我开篇贴的那个图一样，只需要输入`z conf`，就直接跳转到我常去的nginx配置目录了，非常方便

1. 首先需要安装git，`apt-get install git`
2. 我是在`/usr/share/`目录下，`git clone --depth=1 https://github.com/rupa/z`，你有可以在其它目录安装，但是要记住路径
3. `vim ~/.zshrc`，添加`. /usr/share/z/z.sh`
4. `source ~/.zshrc`

# 结语

至此所有的服务器相关的初始设置都已完成，希望本文对大家有所帮助，哪里有错误请告诉我，好及时修改以免误导他人，谢谢
