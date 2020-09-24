---
title: "多SSH设置"
date: 2020-09-24T16:58:39+08:00
draft: true
---

# abstract
经常会遇到这种情况，即一台电脑需要多个公钥来使得github的push pull流畅。

具体操作如下，不管是windows还是MacBook还是Ubuntu貌似都使用。
1. 首先进入 公钥和私钥的存储目录

```sh
cd ~/.ssh
```
2. 生成可用于github账号的公钥和私钥

```sh
ssh-keygen -t rsa -C "XXXXX@XXX.com"
```
`XXXXX@XXX.com`指的是自己的邮箱，比如我的github账号邮箱是gmail

3. 会出现让输入密钥的名称

```sh
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/mosesyang/.ssh/id_rsa):
```
如果不操作就会默认以`id_rsa`为名。咱们这里是为了多SSH，所以，多出来的github账号就填写新的名称（可以自定）。

4. 后面就是让输入密码

```sh
Enter passphrase (empty for no passphrase):

Enter same passphrase again:
```
这里不输入就行了，不需要密码=。=
于是，这里的`id_rsa.pub`文件就是用来复制到github的公钥

5. 要在这个目录（~/.ssh）下创建一个配置文件`config`文件

你没有看错，没有后缀，对于多个ssh的密钥，配置信息类似如下
```sh
# XXX
Host github.com
User XXXXX@XXX.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa

# xxxhhh
Host github.com
User xxxhhh@XXX.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/xxxhhh
```

6. 要把新的ssh密钥加入？？？
加入啥我也不知道，就黑箱操作吧
```sh
ssh-add ~/.ssh/xxxhhh
```

7. 在github中加入公钥
这一步就不介绍了