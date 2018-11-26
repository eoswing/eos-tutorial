---
layout: '[layout]'
title: （八）卡牌游戏第二课：存储状态和登录
date: 2018-11-5 22:20:02
tags: 手把手教你玩eos
---

# 手把手教你玩eos 
> 我是此系列教程作者，eoswing团队肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第八篇。本篇教程主要学习智能合约中如何创建多索引表以及更新表，来完成登录状态的存储。


## 0.2 学习内容
1. 相关准备
2. 智能合约代码编写
3. 前端代码讲解

## 0.3 机器环境

- cpu: 1核
- 内存: 8G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。

# 1 相关准备
## 1.1 课程目标

我们首先学习如何在智能合约中存储玩家信息，使用login动作。然后我们将开始构建Web前端，添加Login页面。

为了使Login页面与智能合约交互，我们将使用eosjs来连接到EOSIO区块链。我们使用eosjs库来调用智能合约中的登录操作。接下来，我们将React连接到Redux以存储应用程序状态。最后，我们将所有这些组合在一起并添加一个Game页面，以便我们有一个可用的登录屏幕。

在本课程结束时，玩家应该能够登录游戏并进入游戏中的首屏。

## 1.2 准备工作

### 进入开发环境容器

{% codeblock lang:Bash %}
	docker exec -it eosdev /bin/bash
{% endcodeblock %}

### 进入智能合约文件夹

{% codeblock lang:Bash %}
	cd /eos-work/contracts/cardgame
{% endcodeblock %}

# 2 智能合约代码编写

## 2.1 建立存储玩家信息的数据结构

打开cardgame.hpp

{% codeblock lang:Bash %}
	vi cardgame.hpp
{% endcodeblock %}

我们要在cardgame.hpp中创建一个user_info来存储以下玩家信息：

- 玩家的名字 name
- 胜利计数 win_count
- 失败计数 lost_count

同时我们还将定义一个login登录动作。

在原有代码基础上输入代码，更新如下：

{% codeblock lang:C %}	
	#include <eosiolib/eosio.hpp>

	using namespace std;
	class cardgame : public eosio::contract {
	
	  private:
	
	    // @abi table users
	    struct user_info {
	      account_name    name;
	      uint16_t        win_count = 0;
	      uint16_t        lost_count = 0;
	
	      auto primary_key() const { return name; }
	    };
	
	    typedef eosio::multi_index<N(users), user_info> users_table;
	
	    users_table _users;
	
	  public:
	
	    cardgame( account_name self ):contract(self),_users(self, self){}
	
	    void login(account_name username);
	
	};
{% endcodeblock %}

输入:wq 保存退出

## 2.2 编写login登录动作逻辑

打开cardgame.cpp

{% codeblock lang:Bash %}
	vi cardgame.cpp
{% endcodeblock %}

在这个代码文件里，我们要编写login动作的具体实现。

首先通过require_auth()来检查传入的用户名是否授权有效。
然后检查玩家信息在不在多索引表中。
如果是第一次加入游戏，将在多索引表中为该用户创建一条记录。

在原有代码基础上输入代码，更新如下：

{% codeblock lang:C %}
	#include "gameplay.cpp"
	
	void cardgame::login(account_name username) {
	  // Ensure this action is authorized by the player
	  require_auth(username);
	
	  // Create a record in the table if the player doesn't exist in our app yet
	  auto user_iterator = _users.find(username);
	  if (user_iterator == _users.end()) {
	    user_iterator = _users.emplace(username,  [&](auto& new_user) {
	      new_user.name = username;
	    });
	  }
	}
	
	EOSIO_ABI(cardgame, (login))
{% endcodeblock %}

输入:wq 保存退出

## 2.3 部署智能合约到麒麟测试网

### 在麒麟网上注册账号

麒麟测试网是对EOS开发者友好的类主网测试环境。

首先我们创建一个账号
在浏览器中输入
http://faucet.cryptokylin.io/create_account?new_account_name
new_account_name 就是你要创建的账号名。在这里我使用123123gogogo

{% asset_img eost08-04.png 创建一个账号 %}

记下账号的两个公私钥对。

再通过水龙头，给账号领一点测试币。(运行一次发100EOS,24小时内可以运行10次)
http://faucet.cryptokylin.io/get_token?your_account_name

{% asset_img eost08-05.png 领一点测试币 %}

在EOS区块浏览器中查看下麒麟网的账号
https://eospark.com/Kylin/account/your_account_name

{% asset_img eost08-06.png 查看下麒麟网的账号 %}

### 编译智能合约

#### 编译wast文件

{% codeblock lang:Bash %}
	cd /eos-work/contracts/cardgame

	eosiocpp -o cardgame.wast cardgame.cpp
	
	ll
{% endcodeblock %}

命令行输出如下：

{% asset_img eost08-07.png 编译wast文件 %}

虽然编译提示使用eosio.cdt工具，但是官方的源文件还是基于传统的eosiocpp。所以还是采用了eosiocpp编译。

#### 编译abi文件

{% codeblock lang:Bash %}
	eosiocpp -g cardgame.abi cardgame.cpp
{% endcodeblock %}

命令行输出如下：

{% asset_img eost08-08.png 编译abi文件 %}

### 部署智能合约

#### 创建钱包

{% codeblock lang:Bash %}
	cleos wallet create -n gamewallet --to-console
{% endcodeblock %}

命令行输出如下：

{% asset_img eost08-09.png 创建钱包 %}

#### 导入账号私钥

{% codeblock lang:Bash %}
	cleos wallet import -n gamewallet
{% endcodeblock %}

回车后，再输入active权限私钥。
命令行输出如下：

{% asset_img eost08-10.png 导入账号私钥 %}


### 购买点资源

{% codeblock lang:Bash %}
	cleos -u https://api-kylin.eosasia.one system delegatebw 123123gogogo 123123gogogo "40.0000 EOS" "40.0000 EOS"
{% endcodeblock %}

命令行输出如下：

{% asset_img eost08-11.png 购买点资源1 %}

{% codeblock lang:Bash %}
	cleos -u https://api-kylin.eosasia.one system buyram 123123gogogo 123123gogogo "900 EOS"
{% endcodeblock %}

命令行输出如下：

{% asset_img eost08-12.png 购买点资源2 %}

### 部署智能合约

{% codeblock lang:Bash %}
	cleos -u https://api-kylin.eosasia.one set contract 123123gogogo /eos-work/contracts/cardgame -p 123123gogogo@active
{% endcodeblock %}

命令行输出如下：

{% asset_img eost08-13.png 部署智能合约 %}

### 在区块浏览器中查看合约

输入网址：https://eospark.com/Kylin/contract/123123gogogo 查看合约情况。

{% asset_img eost08-14.png 查看合约 %}

至此，后端的合约部署完成。

# 3 前端代码讲解

## 3.1 相关准备
### 清理前端文件夹

{% codeblock lang:Bash %}
	cd /eos-work
	rm -rf ./frontend
{% endcodeblock %}

###下载源文件

{% codeblock lang:Bash %}	
	apt-get install subversion

	svn checkout https://github.com/EOSIO/eosio-card-game-repo/branches/lesson-2/frontend
{% endcodeblock %}

### 安装相关库文件

{% codeblock lang:Bash %}	
	cd frontend
	npm update
{% endcodeblock %}

### 测试一下

{% codeblock lang:Bash %}
	npm start
{% endcodeblock %}

在浏览器中输入服务器网址查看：

{% asset_img eost08-01.png 网址查看 %}

现在还只能看看，里面还有配置没处理好。
在命令行中输入ctrl+c,退出。

## 3.2 代码讲解

因为涉及到代码比较多，很多是基础性的react知识。

不做深入拓展理解，跟着教程继续一步步实现也没有问题。

如果读者想知其然后，再知其所以然，可以多谷歌下。

下面主要对主要框架结构和关键代码逻辑进行讲解

### 代码框架结构

{% codeblock lang:Bash %}
	cd src
	tree -d
{% endcodeblock %}

可以看到代码结构如下：

{% asset_img eost08-02.png 代码结构 %}

其中，在components文件夹下，Login和Game分别是登录页面和游戏主页面。

这个运行的流程如下图：

{% asset_img eost08-03.png 代码流程 %}

### 修改配置参数

{% codeblock lang:Bash %}
	 cd /eos-work/frontend
	 vi .env
{% endcodeblock %}

修改参数如下：

{% codeblock lang:Bash %}
	REACT_APP_EOS_CONTRACT_NAME="123123gogogo"
	REACT_APP_EOS_HTTP_ENDPOINT="https://api-kylin.eosasia.one"
{% endcodeblock %}

:wq保存退出

## 3.3 运行测试

{% codeblock lang:Bash %}
	npm start
{% endcodeblock %}

在浏览器中输入网址查看。
同时在麒麟网上按照上面的步骤申请一个玩家账号。

我申请了一个cardgametest账号

{% asset_img eost08-15.png 网址查看1 %}

输入正确的账号名和私钥key后，点击按钮，成功进入游戏主页面，看到welcome!

{% asset_img eost08-16.png 网址查看2 %}

输入错误的账号名或者key后，点击按钮，进入失败并反馈相关信息。

{% asset_img eost08-17.png 网址查看3 %}
	
# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- EOS官方游戏开发第二课: https://battles.eos.io/tutorial/lesson2/chapter1	

# 下一篇：<a href="https://blog.eoswing.io/2018/11/12/eos-tutorial-09/" target="_blank">（九）卡牌游戏第三课：从区块链中读取状态</a>


