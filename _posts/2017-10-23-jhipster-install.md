---
layout: post
title: Jhipster安装
categories: [jhipster, java, spring]
description: Jhipster安装
keywords: jhipster, java, spring
---

# JHipster的安装

JHipster GitHub地址：https://jhipster.github.io/


### 安装步骤:

* 1、安装java 8（http://www.oracle.com/technetwork/java/javase/downloads/index.html）

* 2、安装Maven

　　Maven地址（http://maven.apache.org/） 

* 3、安装Git（https://git-scm.com/）

　　如果需要你也可以选择安装Git客户端SourceTree（https://www.sourcetreeapp.com/）

* 4、安装NodeJs，推荐LTS版本（http://nodejs.org/）安装的同时它也会帮你装上npm包管理器

* 5、安装Yeoman：npm install -g yo

* 6、安装Bower：npm install -g bower

* 7、安装gulp：npm install -g gulp-cli

* 8、安装JHipster：npm install -g generator-jhipster

至此JHipster安装完成，接下来我们创建一个Application

### 创建过程：

* 1、创建一个空目录：mkdir myapplication

* 2、进入该目录：cd myapplication/

* 3、生成应用：yo jhipster

在生成过程中有一些配置及选项需要填写，比如名称、包名、NoSql等，按需填写即可。

完成以上步骤之后，属于你的myapplication应用就算创建成功了。