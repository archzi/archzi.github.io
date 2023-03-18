---
title: REHL Linux 服务器设置免密登录
date: 2020-06-11 17:00:00
categories: [Linux]
tags: [笔记]     # TAG names should always be lowercase
---

## 环境
- Red Hat Enterprise Linux 8
- Red Hat Enterprise Linux 7
- Red Hat Enterprise Linux 6
- Red Hat Enterprise Linux 5

## 方法步骤
1. 生成 SSH key
如果你还没有生成过 ssh-key，参照下面的步骤生成一对 ssh-key，如果已经生成直接跳过
```bash
[user@ssh-client.example.com ~]$ ssh-keygen -t ecdsa
Generating public/private ecdsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_ecdsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/user/.ssh/id_ecdsa.
Your public key has been saved in /home/user/.ssh/id_ecdsa.pub.
The key fingerprint is:
SHA256:Q/x+qms4j7PCQ0qFd09iZEFHA+SqwBKRNaU72oZfaCI user@ssh-client.example.com
The key's randomart image is:
+---[ECDSA 256]---+
|.oo..o=++        |
|.. o .oo .       |
|. .. o. o        |
|....o.+...       |
|o.oo.o +S .      |
|.=.+.   .o       |
|E.*+.  .  . .    |
|.=..+ +..  o     |
|  .  oo*+o.      |
+----[SHA256]-----+
```

2. 复制`公钥(Public Key)` 到服务器的用户账户中
选择下面几种方法的任意一种即可。
> 注意： 公钥文件名称可能和你的不一样。

    - 用 `ssh-copy-id` linux 环境
    前提是你已经有服务器 ssh 登录的密码了，并且`openssh-clients` package 已经按照到服务器了，可以通过以下方式将上一步生成的`公钥`安装到服务器中。
    ```bash 
    [user@ssh-client.example.com ~]$ ssh-copy-id -i ~/.ssh/id_ecdsa.pub user@ssh-server.example.com
    user@ssh-server's password:
    ```
    - 用`ssh` 和 `cat`命令 linxu 环境和 windows 环境都可以用。
    ```bash
    [user@ssh-client.example.com ~]$ cat ~/.ssh/id_ecdsa.pub | ssh user@ssh-server.example.com "cat >> ~/.ssh/authorized_keys"
    ```
    以上代码访问了你本地的 `.ssh`目录，并且复制到了服务器的 `authorized_keys` 文件中。

    - 手动复制 不推荐

        如果你可以登录服务器的 terminal，你可以直接用编辑器编辑 `~/.ssh/authorized_keys` 文件，并且把你的本地公钥内容添加到文件中。
        比如以下这样：

        **没有添加之前**

        Host(ssh-client.example.com)
        ```
        [user@ssh-client.example.com ~]# cat ~/.ssh/authorized_keys
        ecdsa-sha2-nistp256 BBBBBE2VjZHNhLXNoYTItbmlzdHAyNTYBBBBIbmlzdH1111AAABBBFIv/yAbGAnT1qi2MEsLTAAB8v+YJfJoarEV8uUuKaVEnKyR/FblcI/lbwZ3pqxfalqNuqxQJHhAaJuJkE0jlnI= user@ssh-client.example.com
        ```
        Host(ssh-server.example.com)
        ```
        [user@ssh-server.example.com ~]# cat ~/.ssh/authorized_keys
        ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdH1111AAABBBFIv/yAbGAnT1qi2MEsLTAAB8v+YJfJoarEV8uUuKaVEnKyR/FblcI/lbwZ3pqxfalqNuqxQJHhAaJuJkE0jlnI= user1@example-1.com
        ```

        **添加之后**

        Host (ssh-server.example.com)
        ```
        [root@ssh-server.example.com ~]# cat ~/.ssh/authorized_keys
        ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdH1111AAABBBFIv/yAbGAnT1qi2MEsLTAAB8v+YJfJoarEV8uUuKaVEnKyR/FblcI/lbwZ3pqxfalqNuqxQJHhAaJuJkE0jlnI= user1@example-1.com
        ecdsa-sha2-nistp256 BBBBBE2VjZHNhLXNoYTItbmlzdHAyNTYBBBBIbmlzdH1111AAABBBFIv/yAbGAnT1qi2MEsLTAAB8v+YJfJoarEV8uUuKaVEnKyR/FblcI/lbwZ3pqxfalqNuqxQJHhAaJuJkE0jlnI= user@ssh-client.example.com
        ```
3. 尝试登录
现在尝试登录一下服务器，检查一下是否还需要密码
```
[user@ssh-client.example.com ~]$ ssh user@ssh-server.example.com
```

## 问题解决
1. 文件权限问题
确保文件权限是以下权限
```
[user@ssh-server.example.com ~]$ ls -ld ~/{,.ssh,.ssh/authorized_keys*}
drwx------. 25 user user 4096 Aug 21 11:01 /home/user/
drwx------.  2 user user 4096 Aug 17 13:13 /home/user/.ssh
-rw-------.  1 user user  420 Aug 17 13:13 /home/user/.ssh/authorized_keys
```
如果文件权限不对，用以下命令解决
```
[user@ssh-server.example.com ~]$ chmod 600 ~/.ssh/authorized_keys
[user@ssh-server.example.com ~]$ chmod 700 ~/.ssh/
```
另外一种替代办法是关闭 `sshd`服务的`StrictModes`,可以修改配置文件`/etc/ssh/sshd_config` 变成以下内容
```
StrictModes no
```
随后，通过`sudo systemctl restart sshd` 来重启 `sshd`服务，让配置文件生效。

2. SELinux 设置
SELinux 可以拒绝`sshd`服务访问 `~/.ssh`目录。通过以下命令解决
```
[user@ssh-server.example.com ~]$ restorecon -Rv ~/.ssh
```
3. 打开服务器 `sshd` 服务的`RSA`认证。
确保`/etc/ssh/sshd_config` 文件中的以下配置没有被注释
```
RSAAuthentication yes
PubkeyAuthentication yes
```
并且，关闭密码认证 `PasswordAuthentication no`,之后重新启动`sshd` 服务

经过以上步骤，应该可以免密登录服务器了。
