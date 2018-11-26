
（七）卡牌游戏第一课：搭建前后端框架
===================================

# 手把手教你玩eos 
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

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

![](/images/eost07-00.png "官网截图")

本项目大体分为智能合约和前端两个部分。

EOSIO智能合约采用C++编写。不过，如我们在第4篇教程介绍智能合约时所说，即使你不会C++也没关系，毕竟入门学习智能合约开发，需要掌握的C++知识并不多。  

前端部分，采用React+Redux。

## 1.2 进入docker容器
### 下载容器

```Bash
	docker pull eosio/eos-dev:v1.4.1
```

![](/images/eost07-01.png "下载容器")

### 建立项目文件夹

```Bash
	mkdir /eosapp
	mkdir /eosapp/contracts
	mkdir /eosapp/contracts/cardgame
```

命令行输出如下：

![](/images/eost07-02.png "建立项目文件夹")

### 配置容器

```Bash
	docker run -it -d --net=host --rm --name eosdev -v /eosapp:/eos-work eosio/eos-dev:v1.4.1 /bin/bash
```

命令行输出如下：

![](/images/eost07-03.png "配置容器")

### 进入docker容器

```Bash
	docker exec -it eosdev /bin/bash
```

命令行输出如下：

![](/images/eost07-04.png "进入docker容器")

# 2 构建后端智能合约代码框架
## 创建智能合约的3个文件

```Bash
	cd /eos-work/contracts/cardgame

	touch cardgame.hpp
	touch cardgame.cpp
	touch gameplay.cpp
```

命令行输出如下：

![](/images/eost07-05.png "创建智能合约的3个文件")	

三个文件分别为：

- cardgame.hpp - 定义智能合约的C ++头文件。
- cardgame.cpp - 实现智能合约操作主体文件。
- gameplay.cpp - 智能合约中使用的内部函数。


## 编码cardgame.hpp

```Bash
	vi cardgame.hpp
```

输入如下代码：

```C
	#include <eosiolib/eosio.hpp>

	using namespace std;
	class cardgame : public eosio::contract {
	
	  public:
	
	    cardgame( account_name self ):contract(self){}
	
	};
```

输入:wq 保存退出

## 编码gameplay.cpp

```Bash
	vi gameplay.cpp
```

输入如下代码：

```C
	#include "cardgame.hpp"
```

输入:wq 保存退出

## 编码cardgame.cpp

```Bash
	vi cardgame.cpp
```

输入如下代码：

```C
	#include "gameplay.cpp"

	EOSIO_ABI(cardgame, BOOST_PP_SEQ_NIL)
```

输入:wq 保存退出

# 3 构建前端框架	

## 安装node
中间遇到[y/n]时，直接输入 y 即可

```Bash
	cd /eos-work
	curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
	apt-get install -y nodejs
	apt-get install npm
	npm install n -g
	n stable
	npm i -g pm2
```

## 构建前端

```Bash	
	npm init react-app frontend
	cd frontend
	npm start
```

在浏览器中输入服务器网址，我的是http://47.75.214.239:3000/，查看：

![](/images/eost07-06.png "查看网址1")	


## 修改文件夹代码组织

### 清空src文件夹中的文件

```Bash
	cd src
	rm *
```

### 添加components文件夹及相关代码	

```Bash	
	mkdir components
	mkdir ./components/App

	vi ./components/App/App.jsx
```

输入如下代码：

```js")	
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
```
	
输入:wq 保存退出

```Bash
	vi ./components/App/index.js
```

输入如下代码:

```js")
	import App from './App';

	export default App;
```

输入:wq 保存退出

```Bash
	vi ./components/index.js
```

输入如下代码:

```js")
	import App from './App';

	export {
	  App,
	}
```

输入:wq 保存退出	


### 修改src/index.js文件

```Bash
	vi index.js
```

删除默认代码，输入如下代码：

```js")
	import React from 'react';
	import ReactDOM from 'react-dom';
	import { App } from './components';
	
	ReactDOM.render(
	  <App />,
	  document.getElementById('root')
	);
```

输入:wq 保存退出	

#### 最终文件夹结构如下：

```Bash
	apt-get install tree
	tree
```

命令行输出如下：

![](/images/eost07-07.png "文件夹结构")	

### 再次运行前端

```Bash
	cd /eos-work/frontend
	npm start
```

输入网址查看：

![](/images/eost07-08.png "查看网址2")	

# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- EOS官方游戏开发第一课: https://battles.eos.io/tutorial/lesson1/chapter1	

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。

# 下一篇：<a href="https://github.com/eoswing/eos-tutorial/blob/master/eos-tutorial-08.md" target="_blank">（八）卡牌游戏第二课：存储状态和登录</a>
