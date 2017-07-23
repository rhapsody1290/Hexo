---
title: Gitlab安装配置

date: 2017-05-16 20:28:00

categories:
- Git

tags:
- Gitlab

---

## 安装

之前安装的时候忘记了，是通过官网的教程，遇到问题再去网上找的方式安装成功的，可以参考一下文档

http://skyao.github.io/2015/02/16/git-gitlab-setup/
https://segmentfault.com/a/1190000002722631

## 配置文件

配置文件位置：

	/etc/gitlab/gitlab.rb

配置生效：

	sudo gitlab-ctl reconfigure

Nginx 运行目录：

	/opt/gitlab/embedded/sbin/nginx

Nginx 配置文件(会被覆盖)：

	/var/opt/gitlab/nginx/etc/nginx.conf

在/var/opt/gitlab/nginx/etc/nginx.conf 开头处有这样的内容：

	# This file is managed by gitlab-ctl. Manual changes will be
	# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
	# and run `sudo gitlab-ctl reconfigure`.

配置文件引用文件：

	/var/opt/gitlab/nginx/etc/gitlab-http.conf

原来可以通过修改**/etc/gitlab/gitlab.rb来配置**，我再折腾一下。