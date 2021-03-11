---
layout:     post
title:      "github上拒绝连接push&pull"
subtitle:   " \" \""
date:       2021-03-11 12:00:00
author:     "paulikarl"
header-img: "img/post-bg-2015.jpg"
catalog:    true
tags:
    - git
    - connect
---

>问题描述gnutls_handshake() failed: The TLS connection was non-properly terminated.


##### 解决记录
原因是将http错误配置了https的代理

1.正确的为git配置代理的方法如下：
```
git config --global http.https://github.com.proxy http://127.0.0.1:7890（此处填写自己的ip,下面也是一样）
git config --global https.https://github.com.proxy https://127.0.0.1:7890
```

然后push报错
>Recv failure: Connection reset by peer

2.需要重新配置远程仓库
```
git config remote.origin.url git@github.com:用户名/项目名.git
```

>enticity of host 'github.com (xx.xxx.xxx.xxx)' can't be established.
RSA key fingerprint is SHA256:n...8.
Are you sure you want to continue connecting (yes/no)? yes

>Warning: Permanently added 'github.com,xx.xxx.xxx.xxx' (RSA) to the list of known hosts.
Permission denied (publickey).
fatal: Could not read from remote repository.Please make sure you have the correct access rights
and the repository exists.

3.此时建议直接重新建立ssh连接
>$ls -al ~/.ssh

>ssh-keygen -t rsa -C "PauliKarl"

>cat ~/.ssh/id_rsa.pub

4.登陆github,点击头像-settings-new SSH,复制新生成的SSH

5.ok，正常push