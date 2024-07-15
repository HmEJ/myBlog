---
title: 搭建git服务 
date: 2024-07-07 20:45:53
categories:
    - 运维
tags:
    - git
---



记录一下搭建git服务的过程（简单版）

# 步骤

## 新建用户

第一步，创建一个新的用户，是为了便于权限管理

```sh
adduser git
```

可以通过 `cat /etc/passwd` 查看当前系统的用户

## 添加ssh公钥

第二步，将客户端机器的ssh公钥（位于用户目录下的 .ssh 目录中）添加到服务器用户的 `.ssh/authorized_keys` 中。如果没有的话新建这个文件。

这一步的主要目的是为了能让客户端机器可以通过git访问服务器。

试着通过 `ssh git@<服务器ip或域名>` 来访问服务器，如果这里公钥添加成功了，是可以直接访问的。

```sh
cd ~
mkdir .ssh && cd .ssh
touch authorized_keys
```

## 初始化一个远端仓库

在 ~ 下新建一个远端仓库，通过 `git init --bare` 来初始化。

```sh
cd ~
mkdir test.git && cd test.git
git init --bare
```

## 推送文件到远端仓库

在客户端机器上克隆远端仓库：

```sh
git clone git@<ip/domain>:test.git
```

进行一些修改，然后推送。

---

这样就实现了一个简易的git服务器了。
