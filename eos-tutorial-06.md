---
layout: '[layout]'
title: （六）架设EOS区块浏览器
date: 2018-10-23 12:03:02
tags: 手把手教你玩eos
---

# 手把手教你玩eos 
> 我是此系列教程作者，eoswing团队肖南飞,区块链技术开发人员。


# 0.引言
## 0.1教程概况
手把手教你学eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第六篇,主要是讲解如何搭建一个EOS区块浏览器。


## 0.2 学习内容
1. 相关准备知识
2. 配置docker容器环境
3. 架设区块浏览器

## 0.3 机器环境

- cpu: 1核
- 内存: 8G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。


# 1 相关准备知识
## 1.1 硬件说明

本教程仅仅是演示如何一步步搭建一个区块浏览器，所以硬件配置相对较低。如果有持续运行区块浏览器的需求，请相应调高硬件配置。

## 1.2 架设目标

架设的目标演示网站为: https://eosnetworkmonitor.io/

{% asset_img eost06-00.png 演示网站 %}



对应的github源码地址为: https://github.com/CryptoLions/EOS-Network-monitor
	

# 2 配置docker容器环境
## 2.1 配置ubuntu容器
### 下载镜像

{% codeblock lang:Bash %}
	docker pull ubuntu:18.04
{% endcodeblock %}

命令行输出如下：

{% asset_img eost06-01.png 下载镜像 %}

### 配置容器

{% codeblock lang:Bash %}
	docker run -it -d --net=host --rm --name eosmonitor -v /data/monitor:/monitor-work ubuntu:18.04 /bin/bash
{% endcodeblock %}
    
命令行输出如下：

{% asset_img eost06-02.png 配置容器 %}


### 进入docker容器

{% codeblock lang:Bash %}
	docker exec -it eosmonitor /bin/bash
{% endcodeblock %}

### 更新源索引

{% codeblock lang:Bash %}
	apt-get update
{% endcodeblock %}

命令行输出如下：

{% asset_img eost06-03.png  更新源索引 %}

### 安装相关组件
 中间遇到[y/n]时，直接输入 y 即可

{% codeblock lang:Bash %}
	apt-get install sudo
	apt-get install curl
	apt-get install git
	apt-get install vim
{% endcodeblock %}
	
## 2.2 配置环境
### 安装Nodejs v10和mongodb

{% codeblock lang:Bash %}	
	curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
	sudo apt-get install -y nodejs
	sudo apt-get install npm
	sudo npm install n -g
	sudo n stable
	npm i -g pm2
	sudo apt-get install mongodb
{% endcodeblock %}

### 查看nodejs和mongodb安装版本

{% codeblock lang:Bash %}
	mongo --version
	node --version
{% endcodeblock %}

命令行输出如下：

{% asset_img eost06-04.png  查看nodejs和mongodb安装版本 %}


### 创建数据和日志文件目录

{% codeblock lang:Bash %}
	mkdir /monitor-work/mongo
	mkdir /monitor-work/mongo/data
	mkdir /monitor-work/mongo/logs
{% endcodeblock %}

### 运行mongo服务

{% codeblock lang:Bash %}
	mongod --dbpath=/monitor-work/mongo/data --fork --port 27017 --bind_ip localhost --logpath=/monitor-work/mongo/logs/work.log --logappend
{% endcodeblock %}

命令行输出如下：

{% asset_img eost06-05.png  运行mongo服务 %}

# 3 架设区块浏览器



## 3.1 配置后端

### git下载源码

{% codeblock lang:Bash %}	
	cd /monitor-work
	git clone https://github.com/CryptoLions/EOS-Network-monitor.git
{% endcodeblock %}

命令行输出如下：

{% asset_img eost06-06.png  git下载源码 %}

### 修改配置文件

{% codeblock lang:Bash %}
	cd /monitor-work/EOS-Network-monitor/netmon-backend/config/
	cp default.json default.json.back
	vi default.json
{% endcodeblock %}

这里目前有三个地方需要注意，分别用箭头标识出来了：
1.是你的服务器后台网址，这里我输入的是自己的服务器ip地址
2.可用的eosapi服务节点网址
3.是备用的eosapi服务节点网址

{% asset_img eost06-07.png 修改配置文件 %}


这个配置文件里还有很多区块浏览器运营需要的账号配置，在这里就不一一配置了。感兴趣的可以看源码注释。

### 启动后端

{% codeblock lang:Bash %}
	cd ..
	npm install
	pm2 start ecosystem.config.js
{% endcodeblock %}

## 3.2 配置前端

### 安装yarn

{% codeblock lang:Bash %}
	curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
	echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
	sudo apt-get update && sudo apt-get install yarn

	yarn --version
{% endcodeblock %}

### 修改配置文件

{% codeblock lang:Bash %}	
	vi /monitor-work/EOS-Network-monitor/netmon-frontend/app/constants.js
{% endcodeblock %}

这里主要是修改成你服务器的网址。
我这里修改为 http://47.75.214.239:3002

{% asset_img eost06-08.png 修改配置文件2 %}

### 启动前端

其中 yarn build 为可选，在运营环境时使用。	

{% codeblock lang:Bash %}	
	cd /monitor-work/EOS-Network-monitor/netmon-frontend/
	yarn

	yarn build

	yarn start
{% endcodeblock %}

最后在浏览器中输入网址查看

{% asset_img eost06-09.png 网址查看 %}

其中，因为后端mongo数据库在同步区块中。注意看以下面板：


{% asset_img eost06-10.png  网址查看2 %}

可以与http://eosnetworkmonitor.io 官方演示的对照。

#### 只有当区块数据同步到最新高度附近，区块数据的显示和查询才是完备的。


# 下一篇：<a href="https://blog.eoswing.io/2018/10/29/eos-tutorial-07/" target="_blank">（七）卡牌游戏第一课：搭建前后端框架</a>
    


