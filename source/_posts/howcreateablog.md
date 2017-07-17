title: 使用hexo建立blog
date: 2017-07-15 21:42:02
tags:
---

# 如何建立自己的blog #

## 1. 需求分析 ##

分解问题：

1. 要建立的blog类型？动态 or 静态
2. blog的存放。 独立主机 or 其他
3. 建立自己的网页。 手写 or md格式文件转换

点上一根烟，默默的思考一下。从方便考虑，我需要的是一个静态(无数据库)，暂存在github上的，能够由md格式自动转换的blog。

----------

## 2. 工具的选择 ##
1. hexo负责md文件编辑和转换
2. git 用来上传文件至github

## 3. hexo的使用 ##
以下几篇blog可以大致了解hexo.

### 3.1 hexo主题 ###

theme主题可以从 [hexo-Themes](https://github.com/hexojs/hexo/wiki/Themes) 下载

1. 用于了解hexo-themes结构 --- [从零开始定制hexo主题](http://www.maintao.com/2014/hexo-theme-from-scratch/)

### 3.2 hexo使用 ###
1. [hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)

## 4. github ##
### 4.1 建立github page ###

### 4.2 SSH与GPG
SSH是Secure Shell的缩写，而[GPG](https://www.gnupg.org/)是GNU PG的缩写。 GNU PG是PGP的GNU实现，而[PGP](https://en.wikipedia.org/wiki/Pretty_Good_Privacy)为Pretty good privacy的缩写，一个非开源软件。

github项目的提交可以通过HTTPS方式发送用户名/密码登陆，也可以通过SSH和GPG方式登陆。SSH和GPG可以理解成一种网络协议，同HTTPS，保证数据传输安全。但SSH和GPG方式不像HTTPS协议存在根证书的发布单位，使用口令登陆方式无法防止[中间人攻击](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)。

发生上述攻击的原因是因为在没有根证书的情况下，客户端无法验证服务器端的真实性。SSH和GPG都是RSA算法的应用。RSA算法存在一个公钥和私钥对，通常情况下，私钥存在于服务器端，客户端登陆服务器时，服务器端会下发一个公钥给客户端端，客户端使用公钥加密自己的密码，发送给服务器端。服务器端使用私钥解密密码，验证用户密码的正确性。这种方式被称为**口令登陆**(服务器端保有用户密码)。

但上诉过程也可以反其道行之，服务器端再保存用户的密码，而保存用户的公钥，在用户登录时，服务器会向客户端随机发送一段数据。客户端使用私钥加密数据，并发送给服务器端。如果服务器端使用公钥解密，如果能够解密成功，则允许用户登陆。这种方式被成为**公钥登陆**(服务器端保有用户公钥)。公钥登陆的好处是可以防止中间人攻击。

关于SSH和GPG可以参考阮老师的文章。

-  [SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
-  [GPG入门教程](http://www.ruanyifeng.com/blog/2013/07/gpg.html)

Github即允许客户进行口令登陆，也允许客户进行公钥登陆。Github这样设计有一定道理，允许公钥登陆能够在保证安全的情况下，方便第三方程序的集成和扩展。比如hexo在进行deploy代码到github的动作时，如果采用口令登陆，那么登陆口令对于第三方软件变的透明。但是如果采用公钥登陆，在整个过程中，第三方软件只能接触到用户的私钥，口令不会显式的出现在登陆场景中。

在Github每个用户的key setting页面里，可以设置SSH和GPG的密钥。在安装了git客户端后，在git bash中可以产生ssh密钥。 过程如下：

    // 1. 使用ssh-keygen创建key
    $ ssh-keygen -t rsa -C "xxx@xxx.com"
    Generating public/private rsa key pair.
    ....
    Enter passphrase (empty for no passphrase): // 对私钥设置密码
    ....
    The key fingerprint is:
    SHA: .....
    The key's randomart image is:
    .......
    
    // 2. 由于对私钥设置了密码，每次使用私钥时，程序会要求用户输入私钥密码
    // 使用SSH agent程序缓存已解密的私钥，在需要的时候提供给您的SSH客户端。
    // 这样只要输入一遍私钥密码即可
    $ eval "ssh-agent -s"
    ....
    echo Agent pid 857996;
    
    // 3. 使用ssh-add，在登录shell的时候自动添加您的私钥到ssh-agent的缓存
    $ ssh-add ~/.ssh/id_rsa
    Could not open a connection to your authentication agent.
    $ ssh-agent bash
    $ ssh-add ~/.ssh/id_rsa
    $ Enter passphrase for /f/Gene/OpenSource/Psd/GithubSSH/id_rsa:
    $ Identity added: ~/.ssh/id_rsa (~/.ssh/id_rsa)

    // 4. 将key添加到github账户中去
    复制 ~/.ssh/id_rsa.pub到github SSH key中

    // 5. 测试
    $ ssh -T git@gitub.com

关于ssh-keygen和ssh-agent的使用见[SSH keys](https://wiki.archlinux.org/index.php/SSH_keys_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))。

上面的步骤中，只有1，4步是必要的。

## 5. 部署网站 ##
部署非常简单。首先修改hexo目录下的配置文件_config.yml。

    type: git
    repo: git@github.com:YourRepository.git.git
    branch: master

然后 

    $ hexo deploy
    ERROR Deployer not found: git            // 有时候会出现,虽然以安装
    $ npm install hexo-deployer-git --save   // 安装即可
    $ hexo deploy                            // 重新部署
    .................
    .................                        // 可能弹出对话框,验证SSH key
    Warning: Permanently added 'github.com,192.30.255.113' (RSA) to the list of known hosts.
    Branch master set up to track remote branch master from: .....
    ............
    INFO  Deploy done: git                   // 成功


----------
question
1. 寻找一个可以寄存blog的站点
2. github上的hg-page分支解析md文件吗？
   解析md文件 -- 则直接上传md文件
   不能解析 -- 需要一个解析器生成html文件
3. github上传
   ssh的作用
4. 使用markdown
5. 使用hexo
