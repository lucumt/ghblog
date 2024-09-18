---
title: "[杂谈]记一次个人GitHub账号的找回过程"
date: 2024-04-03T10:12:47+08:00
lastmod: 2024-04-03T10:12:47+08:00
draft: false
keywords: ["github","2FA","ssh","recovery codes"]
description: "记录自己由于手机格式化导致的GitHub 2FA认证失效而无法登录后恢复GitHub账号的过程"
tags: ["github","git","ssh"]
categories: ["个人博客","编程杂想"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: false
borderImage: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

简要记录下由于自己手机格式化刷机后，`GitHub`两阶段认证失败导致最终无法完整登录，尝试过多种方式后最终通过`github-recovery-codes.txt`恢复自己`GitHub`账号的过程。只是单纯的作为记录，对其他遇到类似的问题的人不具有参考价值。

<!--more-->

## 问题背景

自己的`GitHub`账号注册了有些年头，之前一直是通过账号密码直接登录即可，但`GitHub`官网[^1]从2023年3月份到2023年末对提交代码的用户逐步开启两阶段认证(`2FA`,全称为`two-factor authentication`)，由于那段时间每次登录时`GitHub`都会强制提示开启`2FA`功能，影响正常的使用，且`2FA`是大趋势，为了减少麻烦自己准备`2FA`功能。



`GitHub`官网提供了包括`TOTP`、`SMS`、`passkey`、`security key`、`GitHub Mobile`这5种方式来实现`2FA`，自己一开始想用的是`SMS`，但实际操作时发现其不支持中国大陆的手机号码(哎，又是一个简中特供)，而`GitHub Mobile`不能单独使用，只能在配置了其它`2FA`的前提下使用，可选的范围变小。综合考虑后自己决定采用[**Authy**](https://authy.com/)作为`TOTP`实现，按照官网的描述很快就配置完成，且正常的使用了半年多。

由于`Authy`是安装在个人手机上，而个人手机用了几年之后频繁提示手机存储空间不足(此处必须吐槽下微信)，严重影响使用，于是自己在春节期间将相关的文件备份后对自己的安卓手机进行了格式化与刷机(但是没有对`GitHub`与`Authy`进行备份设置)，以确保手机中有足够的存储空间。

之后自己通过`git`依旧能正常的给`GitHub`提交并推送代码，让我误以为一切正常，知道前段时间自己想登录网页版的`GitHub`,当我输入完正确的邮箱和密码后，`GitHub`提示要求输入`2FA`相关的认证码

!["GitHub要求输入2FA认证码"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-login-require-2fa.png "GitHub要求输入2FA认证码") 

此时我的手机已经格式化重刷系统有一段时间了，`Authy`和`GitHub Mobile`随着手机刷机已经消失不见了! 一开始我以为只要把`Authy`在手机上安装好，然后重新获取对应的认证码即可，但事实证明，我大意了！

## Authy尝试

由于国内特殊的网络环境，自己首先确保了手机可以正常科学上网。

之后便迫不及待的在手机上安装了`Authy`，不过打开该软件之后，其首先要求我通过短信或电话输入验证码，之后才能登录。

一开始我尝试通过发短信的方式获取验证码，但一直提示无法识别手机号，开启科学上网也是如此。

接下来选择给我拨打语音电话，但无论是否开启科学上网，手机一直都无法接听到对应呼叫。

为了排除是否是自己手机问题，用其他同事的苹果手机安装`Authy`并进行测试，依旧无法获取到认证码！

## GitHub尝试

通过`Authy`获取验证码这条路暂时无法走通，接下来尝试绕过`Authy`，看看`GitHub`上有没有其它方式。

在前述的`GitHub`登录页面上点击 **Having problems?** 下面的[**Use a recovery code or begin 2FA account recovery**](https://github.com/sessions/two-factor/recovery)链接，出现如下界面

!["GitHub 2FA恢复页面"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-2fa-recovery-page.png "GitHub 2FA恢复页面") 

上图中提供了三种方式：

1. 使用`authenticator app`，我之前的设置为`Authy`，目前无法登录

2. 使用`recovery code`,官方文档上说该文件的名称为`github-recovery-codes.txt`，但多次全局搜索自己的笔记本电脑，均为找到该文件，此路也不通

3. 点击下方的 **Try 2FA account recovery, or unlink your account email address(es)** 链接，出现类似如下界面

   !["GitHub 2FA恢复确认页面"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-2fa-recovery-begin-confirm-page.png "GitHub 2FA恢复确认页面") 

   点击确认按钮之后，出现如下界面

   !["GitHub账号恢复界面"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-account-recovery-page.png "GitHub账号恢复界面") 

   该界面上虽然有多个选项，但具体到我自己的账号时只有 **Personal access token** 和 **Start unlinking email** 这2个选项，其中`Start unlinking email`表示在尝试各种方法之后都无法找回账号时，将注册邮箱与当前账号解除绑定关系，使用此种方式意味着已经放弃治疗，只能保留邮箱，而账号无法恢复，只能`fork`非`private`的仓库，不到万不得已，此种方式不考虑。

   

   翻看自己的电脑磁盘和博客记录，在编写[**利用GitHub Action实现Hugo博客在GitHub Pages自动部署**](/post/hugo/using-github-action-to-auto-build-deploy/)这篇文章时，自己记录过相关的`personal access token`，然而自己之前为了方便在博客中展示，将对应信息进行了马赛克处理，而我本地又没保存相关的`token`值，此条路也走不通！

   !["GitHub历史token截图"](/blog_img/github/recovery-github-code-when-2fa-not-working/generate-new-token-result.png "GitHub历史token截图") 

## Authy邮件

在`GitHub`上折腾了一圈后还是没能找到有效的解决办法，只能给`Authy` 官方发邮件咨询为啥在手机上无法收到短信或电话，给`Authy`发邮件时，需要在其官网上先注册登录之后才能进行，奇怪的是在其网页版的官网上登录时，同样的手机号能收到验证码，就是用手机APP时死活收不到验证码。

相对于下文要说的`GitHub`官方的技术支持邮件回复的慢吞吞，`Authy`官方的技术支持邮件回复还算及时，我发了邮件之后等了2天就有回复了，在我告知必要的信息之后告知我是由于个人的`Authy`账号被它们的系统判定有异常，给`blocked`了，才导致自己手机上的`Authy`一直无法正常登录。

!["Authy解释账号被锁的原因"](/blog_img/github/recovery-github-code-when-2fa-not-working/authy-reply-for-account-block.png "Authy解释账号被锁的原因") 

在收到`Authy`账号恢复的邮件后，我立马在手机上登录`Authy`期望能出现自己预想中的`Authentication code`，然而现实又给了我一次重击，虽然成功登陆了`Authy`，但是里面空空如也，没有地方可以获取`Authentication code`！

可是我自己之前明明在`Authy`中关联过`GitHub`相关的认证，最开始配置的时候还用过里面生成的`Authentication code`，为何现在啥都没有了？无奈之下只好继续给`Authy`技术支持人员写邮件寻求帮助，而得到的结果却让我更加郁闷，我之前没有在`Authy`中开启备份功能，所以重新安装`Authy`后需要重新关联`GitHub`认证配置！

!["Authy解释账号无法恢复的原因"](/blog_img/github/recovery-github-code-when-2fa-not-working/authy-reply-for-account-backup.png "Authy解释账号无法恢复的原因") 

## GitHub邮件

为了登录`GitHub`需要获取`Authy`中的`Authentication code`，而为了获取`Authy`中的`Authentication code`需要登录`GitHub`后重新配置，陷入死循环了！

此时的我已经感到一丝紧张，难道我的`GitHub`账号真的要丢失？

虽然里面没啥有价值的东西，关注和被关注人也不多，重新创建一个账号后`fork`迁移也不是很麻烦，但是我的`GitHub`开启了`GitHub Pages`并且绑定了个人域名，而个人域名不久前才续费了5年，牵一发而动全身。

不到万不得已，我是不会走到`Start unlinking email`这一步的。



查看网上的资料，发现其他遇到类似问题的人可以通过`ssh -vT git@github.com verify`来获取之前设置好的`token`，之后让`GitHub`官方人员协助解决，前提是`git`能正常连接访问`GitHub`。

我测试了下自己的电脑，发现在`git`中可以正常的`commit`和`push`代码，但是`ssh -vT git@github.com verify`却不好使。

!["git能正常commit和push"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-commit-push-success.png "git能正常commit和push") 

!["ssh认证不生效"](/blog_img/github/recovery-github-code-when-2fa-not-working/ssh-verify-not-working.png "ssh认证不生效") 

既然`git`能正常的`commit`和`push`，那我本地肯定有相关的`token`或`key`信息，事情还有一丝转机。



一开始自己想找`GitHub Support`的人工客服，找了一会发现并没有，只有如下所示的一个虚拟机器人

!["github support虚拟机客服"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-2fa-virtual-agent.png "github support虚拟机客服") 

其回答问题只有简单的yes or no,要是都不满足条件，最后给你来一句there are unfortunately xxx,卵用没有！



只能给`GitHub Support`写邮件咨询，这个过程中又遇到很操蛋的事情：

**要给`GitHub Support`写邮件需要先登录`GitHub`，但我就是无法完全登录`GitHub`才想写邮件求助的，又陷入死循环了！**

What a fuck!

只能又在网上搜索相关资料，好不容易找了一个可以非登录发邮件的入库，结果还要给自己绑定的邮箱发临时验证码，不得不说`GitHub`这块确实挺操蛋的，还好我自己密码没忘记，总算找到了一个反馈入口。

为了避免像之前咨询`Authy`时反复咨询回复邮件的问题，我给`GitHub`官方的技术支持人员写了一封很详细的邮件，添加了各种截图以及之前`Authy`官方回复的附件，重点是说明我本地通过`git`可以正常的`commit`和`push`，同时说明了我本地有打马赛克的`Personal access token`，看看`GitHub Support`能否基于这些判断我是账号的拥有者并协助登录。



在我邮件发送之后`GitHub Support`立马给我自动回复了一封很机械的邮件，重点是说我们的业务量很大，所以你要多等一段时间。

!["github support默认邮件回复"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-support-email-reply-1.png "github support默认邮件回复") 



没关系，等就等呗。

让我想不到的是，这一等就等了十多天，即使我中途又补发了两封邮件来更详细的描述和催促，依旧没有响应，没办法只能等了。

等了十多天之后，终于收到邮件回复，但其内容又是毫无营养的官样回复。

!["github support初次回复"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-support-email-reply-2.png "github support初次回复") 

虽然我的邮件里面写的特别详细，并提出了不少问题，但这封邮件只回复了两个问题：

1. 我们记录到你在2023年8月份下载过`github-recovery-codes.txt`文件，建议你再仔细找找该文件，但我的电脑已经搜索过N次，都没找到
2. 虽然你提供了`Personal access token`，但打了马赛克无法使用，需要提供完整的，同时根据已有的截图显示，该`token`的后缀不正常，就算有完整的，也不能作为恢复凭据。



看起来通过`GitHub Support`邮件咨询的这条路也走不通，此时我已经做好账号丢失的心理准备了。

## 找回恢复码

在解除`GitHub`邮箱绑定之前，我有些不死心，于是又给`GitHub Support`回复了如下的邮件，主要内容有2点：

1. 既然你们记录到我已经下载过`github-recovery-codes.txt`文件，但是我本地并没有找到，请说下具体日期，方便我进一步排查
2. 想具体了解下为啥通过`git`可以正常的`commit`和`push`，但是`ssh -vT git@github.com verify`不好使。

!["github support寻求进一步协助"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-support-ask-further-help.png "github support寻求进一步协助") 

还好，这次发完邮件后第2天他们就给我回复了

!["github support再次回复"](/blog_img/github/recovery-github-code-when-2fa-not-working/github-support-email-reply-3.png "github support再次回复") 

虽然依旧是机械回复，并没有解决我的实际问题，但`GitHub Support`的支持人员这么肯定我下载过`github-recovery-codes.txt`文件并且精确到了分钟，那就表示我肯定下载过。

于是在笔记本上再次全局搜索，无果后，去自己不常用的台式机上搜索，废了九牛二虎之力在回收站里面找到了那个文件，赶紧还原，打开该文件一看，数据都在，然后就是按照`GitHub`官方的说明基于`recovery code`进行恢复。

至此，`GitHub`账号恢复的事情告一段落。

## 总结&教训

1. 遇到事情一定要尝试各种可能的解决方案，不到最后一步，千万不要放弃
2. 做人做事要仔细认证，不要走马观花，如果一开始设置完`2FA`后就仔细阅读`GitHub`官方发的邮件，及时做好备份，也不会这么折腾
3. `AI`时代，人工客户越来也少,机器人客服越来越多，对资本家是好事，对普通用户不一定是好事，要学会适应潮流与大趋势。

[^1]: https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/about-mandatory-two-factor-authentication