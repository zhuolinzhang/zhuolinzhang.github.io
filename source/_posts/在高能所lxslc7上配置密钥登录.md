---
title: 在高能所lxslc7上配置密钥登录
date: 2021-02-26 23:28:50
tags:
- Linux
---

随着工作量的增大，集群的使用也越来越频繁。可是每次登录都要输密码让我很烦。于是我参照网上的教程和高能所集群使用手册设置密钥登录。

## 生成密钥

首先在我自己的电脑上生成密钥

```bash
ssh-keygen -t rsa -f lxslc
```

因为我本地的`~/.ssh`中已经有了Github私钥，为了不覆盖GitHub私钥，需要对文件命名。如果没有其他私钥则不需要指定文件名`-f`。然后，在`~/.ssh`中可以看到有`lxslc`和`lxslc.pub`两个文件。前者为私钥，后者为公钥。随后将公钥上传到lxslc集群`~/.ssh`中。

```bash
scp lxslc.pub username@lxslc7.ihep.ac.cn:~/.ssh/authorized_keys
```

随后登录到集群，确保`authorized_keys`文件权限为600。

修改本地的`~/.ssh/config`设置，在`Host`下添加

```
IdentityFile ~/.ssh/lxslc
```

下次登录就是密钥登录了。

## Troubleshooting

可实际上往往没有这么顺利。在我登录集群之后遇到了`$HOME`目录没有读写权限和X11错误的问题。登录后会出现以下提示

```
/usr/bin/xauth:  timeout in locking authority file /afs/ihep.ac.cn/users/z/username/.Xauthority
```

按照高能所计算环境使用手册的FAQ，采取以下操作：

```bash
-bash-4.2$ kinit
Password for username@IHEPKRB5:
-bash-4.2$ aklog
-bash-4.2$ rm -f ~/.Xauthority
```

随后退出重新登录即可。

## 题外话

我还想对CERN的lxplus集群做类似操作，但根据CERN的指南，不建议使用密钥登录，因为使用这种方法登录后无法访问afs文件系统，只好作罢。CERN推荐使用Kerberos实现免密登录。

## Reference

SSH FAQ for CERN：https://twiki.cern.ch/twiki/bin/view/LinuxSupport/SSHatCERNFAQ

高能所计算环境使用手册：http://afsapply.ihep.ac.cn/cchelp/zh/

