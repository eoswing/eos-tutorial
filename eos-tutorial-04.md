---
layout: '[layout]'
title: （四）编写第一个智能合约Hello_eos
date: 2018-10-08 21:56:02
tags: 手把手教你玩eos
---

# 手把手教你玩eos 
> 我是此系列教程作者，eoswing团队肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你学eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第四篇,主要是通过一个简单的HelloWorld来讲解如何编写EOS智能合约。


## 0.2 学习内容
1. 相关准备知识
2. 编写智能合约hello_eos.cpp
3. 编译和运行智能合约
4. 添加身份验证

## 0.3 机器环境

- cpu: 1核
- 内存: 2G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。


** 提示：以下命令行默认在root权限下执行。如遇权限问题，请在命令前加sudo。**


# 1.相关准备知识
## 1.1 C++语言经验
基于EOSIO的区块链使用WebAssembly（WASM）执行用户生成的应用程序和代码。  
C++是EOS官方推荐的主要智能合约开发语言。  

当然，如果你不会C++也没关系，毕竟入门学习智能合约开发，需要掌握的C++知识并不多。  
但是，最好要有一种编译型语言的学习经验。

## 1.2 相关术语规则

### 操作（action）和类型(type)定义

EOSIO智能合约由一组动作（action）和类型(type)定义组成。  

- action 动作定义指定并实现合同的行为。 

- type 类型定义指定所需的内容和结构。  

EOSIO操作主要在基于消息的通信体系结构中运行。 
客户端通过发送（推送）消息来调用操作nodeos。 
可以使用cleos命令完成，也可以使用EOSIO send方法之一（例如eosio::action::send）完成。

### 内联(Inline)和延迟(Deferred)
EOSIO支持两种基本通信模型，内联(Inline)和延迟(Deferred)。

- Inline 内联，意思就是直接采用内部函数体发起，调用其他函数的方式。这可以保证交易无阻碍执行，不必通知外部失败或者成功结果，同时内联也可保证交易始终处于同一作用域以及权限。

- Deferred 延迟，通过生产者的判定来决定延后按时执行，可能会发生timeout的问题。这种方式可以跨多个作用域工作，并且可以携带着发送给它的合约权限。

### 动作（action）和交易（transaction）

- action是一个动作，账户和合约交互是通过action，可以单独发送一个action。
- Transaction是一组动作。所有action都必须成功，该Transaction才会成功。

收到一个交易并不意味着这个交易已经被确认，它仅仅说明这个交易被一个BP节点接受并且没有错误，当然也意味着很有可能这个交易被其他bp接受了。当一个交易被包含在一个块当中的时候，它才是可以被确认执行的。

# 2.编写智能合约hello_eos.cpp

## 2.1 创建hello_eos智能合约目录框架

### 使用eosiocpp工具生成框架
进入/contracts文件夹用eosiocpp工具生成hello_eos智能合约目录框架。

> eosiocpp是eos自带的一个智能合约开发工具。

先后输入如下命令：

{% codeblock lang:Bash %}
	cd /contracts
	eosiocpp -n hello_eos
	cd hello_eos
	ll
{% endcodeblock %}

命令行输出如下：

{% asset_img eost04-01.png 创建目录框架 %}

### cpp和hpp文件
可以看到，在hello_eos文件夹中已经自动生成了两个文件hello_eos.hpp和hello_eos.cpp。

- hello_eos.hpp是头文件，包含文件引用的变量，常量和函数。这里主要引用了eosiolib/eosio.hpp这个库文件。
- hello_eos.cpp就是包含合约实现方法的源文件。

## 2.2 编辑hello_eos.cpp源文件

eosiocpp生成的hello_eos.cpp文件里已经生成了一些模板内容。  
里面有一个hi方法，主要是打印命令请求中传递的参数。  
下面我们修改其中打印语句。  
将 print("Hello, ",name{user});修改为print("Hello,eosfans: ",name{user});

{% codeblock lang:Bash %}
	vi hello_eos.cpp
{% endcodeblock %}

输入i进行编辑，编辑后的hello_eos.cpp文件内容如下：

{% asset_img eost04-02.png  cpp文件内容 %}

按esc键退出编辑模式后，输入 :wq 后保存退出。

为了极简演示一个合约的编写、编译和运行流程，这个cpp文件只是简单的日志打印，没有引入包括权限，签名等复杂度。

# 3 编译和运行智能合约

## 3.1 编译wast文件

wast文件是WASM适用的由cpp文件编译后的文件格式，这是区块链接收的唯一格式。

{% codeblock lang:Bash %}
	eosiocpp -o /contracts/hello_eos/hello_eos.wast /contracts/hello_eos/hello_eos.cpp
{% endcodeblock %}

命令行输出如下：

{% asset_img eost04-03.png  编译wast文件  %}

## 3.2 编译abi文件
abi是一个json格式的，用来描述智能合约如何在action和二进制程序中进行转变的方法，也用来描述数据库状态。有了abi来描述你的智能合约，开发者和用户都可以通过JSON无缝地与合约进行交互。

{% codeblock lang:Bash %}
	eosiocpp -g /contracts/hello_eos/hello_eos.abi /contracts/hello_eos/hello_eos.cpp
{% endcodeblock %}
	
命令行输出如下：

{% asset_img eost04-04.png  编译abi文件 %}

查看hello_eos目录

{% asset_img eost04-05.png 查看hello_eos目录 %}

## 3.3 创建helloaccount账户

创建公钥私钥对：
{% asset_img eost04-06.png  创建公钥秘钥对 %}

添加秘钥到xiao钱包：

{% asset_img eost04-07.png  添加秘钥到xiao钱包 %}

建立helloaccount账户：

{% asset_img eost04-08.png  建立helloaccount账户 %}

> 以上相关命令，可以查看第3篇的 2.1 创建合约用账户 

## 3.4 上传合同到账户

{% codeblock lang:Bash %}
	cleos set contract helloaccount /contracts/hello_eos -p helloaccount@active
{% endcodeblock %}

命令行输出如下：

{% asset_img eost04-09.png  上传合同到账户 %}

## 3.5 运行合同

输入一个模拟用户名 xiao 使用 helloaccount签名授权，测试通过。

{% codeblock lang:Bash %}
	cleos push action helloaccount hi '["hello"]' -p helloaccount@active
{% endcodeblock %}

命令行输出如下：

{% asset_img eost04-10.png  上传合同到账户 %}

输入一个模拟用户名 xiao 使用 xiaoaccount签名授权，测试通过。

{% codeblock lang:Bash %}
	cleos push action helloaccount hi '["hello"]' -p xiaoaccount@active
{% endcodeblock %}
	
命令行输出如下：

{% asset_img eost04-11.png  上传合同到账户 %}

目前我们的hello_eos合约是不限制hi参数的，也就是说其实我们是没有“xiao”这个签名人的。
现在无论这个参数如何输入账户名，无论用哪个账户签名授权，都可以输出。

下面我们在hello_eos中添加身份验证，要求输入的账户名必须要对应的账户来签名授权。

# 4 添加身份验证
## 4.1 修改cpp文件
在hello_eos.cpp文件中添加 require_auth(user); 这段代码：

{% asset_img eost04-12.png 修改cpp文件 %}

## 4.2 重新编译和部署合同

重复上述编译和部署过程。全部命令行输出如下：

{% asset_img eost04-13.png  编译和部署 %}

## 4.3 再次运行合同

输入一个模拟用户名 xiao 使用 helloaccount签名授权，测试发现身份验证不通过。

输入账户名 helloaccount 使用 helloaccount签名授权，身份验证通过。

输入账户名 xiaoaccount 使用 xiaoaccount签名授权，身份验证通过。

命令行输出如下：

{% asset_img eost04-14.png  再次运行合同 %}

hello_eos的身份验证生效。

# 5 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- Hello World合同: https://developers.eos.io/eosio-cpp/docs/hello-world

# 下一篇：<a href="https://blog.eoswing.io/2018/10/15/eos-tutorial-05/" target="_blank">（五）编写智能合约游戏：三连棋</a>
