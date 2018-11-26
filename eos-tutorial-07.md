---
layout: '[layout]'
title: （七）卡牌游戏第一课：搭建前后端框架
date: 2018-10-29 20:45:02
tags: 手把手教你玩eos
---

# 手把手教你玩eos 
> 我是此系列教程作者，eoswing团队肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第七篇,从这一篇开始，我们将开始学习如何搭建一个EOS卡牌游戏。本篇教程主要是搭建前后端框架。


## 0.2 学习内容
1. 相关准备
2. 构建后端智能合约代码框架
3. 构建前端框架

## 0.3 机器环境

- cpu: 1核
- 内存: 8G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。

# 1 相关准备
## 1.1 项目技术栈

此项目来源于eosio官网:
https://battles.eos.io/main

{% asset_img eost07-00.png 官网截图 %}

本项目大体分为智能合约和前端两个部分。

EOSIO智能合约采用C++编写。不过，如我们在第4篇教程介绍智能合约时所说，即使你不会C++也没关系，毕竟入门学习智能合约开发，需要掌握的C++知识并不多。  

前端部分，采用React+Redux。

## 1.2 进入docker容器
### 下载容器

{% codeblock lang:Bash %}
	docker pull eosio/eos-dev:v1.4.1
{% endcodeblock %}

{% asset_img eost07-01.png 下载容器 %}

### 建立项目文件夹

{% codeblock lang:Bash %}
	mkdir /eosapp
	mkdir /eosapp/contracts
	mkdir /eosapp/contracts/cardgame
{% endcodeblock %}

命令行输出如下：

{% asset_img eost07-02.png 建立项目文件夹 %}

### 配置容器

{% codeblock lang:Bash %}
	docker run -it -d --net=host --rm --name eosdev -v /eosapp:/eos-work eosio/eos-dev:v1.4.1 /bin/bash
{% endcodeblock %}

命令行输出如下：

{% asset_img eost07-03.png 配置容器 %}

### 进入docker容器

{% codeblock lang:Bash %}
	docker exec -it eosdev /bin/bash
{% endcodeblock %}

命令行输出如下：

{% asset_img eost07-04.png 进入docker容器 %}

# 2 构建后端智能合约代码框架
## 创建智能合约的3个文件

{% codeblock lang:Bash %}
	cd /eos-work/contracts/cardgame

	touch cardgame.hpp
	touch cardgame.cpp
	touch gameplay.cpp
{% endcodeblock %}

命令行输出如下：

{% asset_img eost07-05.png 创建智能合约的3个文件 %}	

三个文件分别为：

- cardgame.hpp - 定义智能合约的C ++头文件。
- cardgame.cpp - 实现智能合约操作主体文件。
- gameplay.cpp - 智能合约中使用的内部函数。


## 编码cardgame.hpp

{% codeblock lang:Bash %}
	vi cardgame.hpp
{% endcodeblock %}

输入如下代码：

{% codeblock lang:C %}
	#include <eosiolib/eosio.hpp>

	using namespace std;
	class cardgame : public eosio::contract {
	
	  public:
	
	    cardgame( account_name self ):contract(self){}
	
	};
{% endcodeblock %}

输入:wq 保存退出

## 编码gameplay.cpp

{% codeblock lang:Bash %}
	vi gameplay.cpp
{% endcodeblock %}

输入如下代码：

{% codeblock lang:C %}
	#include "cardgame.hpp"
{% endcodeblock %}

输入:wq 保存退出

## 编码cardgame.cpp

{% codeblock lang:Bash %}
	vi cardgame.cpp
{% endcodeblock %}

输入如下代码：

{% codeblock lang:C %}
	#include "gameplay.cpp"

	EOSIO_ABI(cardgame, BOOST_PP_SEQ_NIL)
{% endcodeblock %}

输入:wq 保存退出

# 3 构建前端框架	

## 安装node
中间遇到[y/n]时，直接输入 y 即可

{% codeblock lang:Bash %}
	cd /eos-work
	curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
	apt-get install -y nodejs
	apt-get install npm
	npm install n -g
	n stable
	npm i -g pm2
{% endcodeblock %}

## 构建前端

{% codeblock lang:Bash %}	
	npm init react-app frontend
	cd frontend
	npm start
{% endcodeblock %}

在浏览器中输入服务器网址，我的是http://47.75.214.239:3000/，查看：

{% asset_img eost07-06.png 查看网址1 %}	


## 修改文件夹代码组织

### 清空src文件夹中的文件

{% codeblock lang:Bash %}
	cd src
	rm *
{% endcodeblock %}

### 添加components文件夹及相关代码	

{% codeblock lang:Bash %}	
	mkdir components
	mkdir ./components/App

	vi ./components/App/App.jsx
{% endcodeblock %}

输入如下代码：

{% codeblock lang:js %}	
	import React, { Component } from 'react';
	
	class App extends Component {
	
	  render() {
	    return (
	      <div className="App">
		EOS Game!
	      </div>
	    );
	  }
	
	}
	
	export default App;
{% endcodeblock %}
	
输入:wq 保存退出

{% codeblock lang:Bash %}
	vi ./components/App/index.js
{% endcodeblock %}

输入如下代码:

{% codeblock lang:js %}
	import App from './App';

	export default App;
{% endcodeblock %}

输入:wq 保存退出

{% codeblock lang:Bash %}
	vi ./components/index.js
{% endcodeblock %}

输入如下代码:

{% codeblock lang:js %}
	import App from './App';

	export {
	  App,
	}
{% endcodeblock %}

输入:wq 保存退出	


### 修改src/index.js文件

{% codeblock lang:Bash %}
	vi index.js
{% endcodeblock %}

删除默认代码，输入如下代码：

{% codeblock lang:js %}
	import React from 'react';
	import ReactDOM from 'react-dom';
	import { App } from './components';
	
	ReactDOM.render(
	  <App />,
	  document.getElementById('root')
	);
{% endcodeblock %}

输入:wq 保存退出	

#### 最终文件夹结构如下：

{% codeblock lang:Bash %}
	apt-get install tree
	tree
{% endcodeblock %}

命令行输出如下：

{% asset_img eost07-07.png 文件夹结构 %}	

### 再次运行前端

{% codeblock lang:Bash %}
	cd /eos-work/frontend
	npm start
{% endcodeblock %}

输入网址查看：

{% asset_img eost07-08.png 查看网址2 %}	

# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- EOS官方游戏开发第一课: https://battles.eos.io/tutorial/lesson1/chapter1	

# 下一篇：<a href="https://blog.eoswing.io/2018/11/05/eos-tutorial-08/" target="_blank">（八）卡牌游戏第二课：存储状态和登录</a>