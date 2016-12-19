---
layout:     post
title:      "ssh连接超时问题"
date:       2016-12-19 18:13:00 +0800
category:   "linux"
excerpt:    "敬请期待"
tags:       "ssh"
---

使用`ssh -vv`查看ssh连接时的debug信息，分析耗时具体在哪个环节，然后去修改`/etc/ssh/sshd_config`中的相关配置。

## Authentications方式冗余

如果连接在尝试Authentications(认证方式)时耗时太多，在下面这个地方停留时间过长。

````
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic,password
````

我们可以避免ssh去尝试不必要的认证方式，根据你自己的使用情况去禁用部分配置选项。例如：

````
# 密码验证
PasswordAuthentication yes
# 质疑-应答认证(ssh的时候会返回challenge，然后根据challenge生成response，用response登录)，可以禁用
ChallengeResponseAuthentication no
# 使用基于GSSAPI的用户认证，可以禁用
GSSAPIAuthentication no
# 用户退出登录后自动销毁用户凭证缓存
GSSAPICleanupCredentials no
# 非对称密钥验证是否开启。建议开启，关闭之后可能会password登陆失败
UsePAM yes
# 采用公/密钥的方式进行身份验证，这个可以保留。
PubkeyAuthentication yes
# X11转发，一般ssh都不会使用远程的X11图形，也可以禁用
X11Forwarding no
````

## 地址解析时间过长

如果减少了尝试的认证方式之后，ssh连接仍然在这个地方耗时过长，那就有可能是认证过程中的DNS解析太慢。
如果我们使用的ip地址去访问，不需要域名解析，大可省略DNS解析这个步骤。

````
# 禁用dns解析
UseDNS no
````

使用了上面两种措施之后，ssh连接速度都会明显加快。如果有另外的情况，需要另作分析。
