（十七）使用EOSFactory构建和部署智能合约
===================================

# 手把手教你玩eos 
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第十七篇。本篇教程讲解如何使用EOSFactory执行最简单的开发周期：创建新合约，编辑代码，构建合约，部署合约并与之交互。


## 0.2 学习内容
1. 创建新合约
2. 编译和部署合约
3. 测试合约
4. 修改代码，重新编译部署

## 0.3 机器环境

- 笔记本电脑
- 操作系统：Windows 10

# 1 创建新合约

## 1.1 进入环境

首先进入WSL中的ubuntu的命令行，然后运行Python CLI。

```Bash
	python3
```

进入Python shell后，导入EOSFactory库。

```Bash
	from eosfactory.eosf import *
```

## 1.2 使用模板创建新合约

从预定义模板创建新合约，第一个参数为合约名称，第二个参数为模板名称。

```Bash
	contract = ContractBuilder(project_from_template("foo_bar", template="01_hello_world"))
```

![创建新合约](/images/eost17-01.png "创建新合约")

查看新建合约所在路径

```Bash
	contract.path()
```

![所在路径](/images/eost17-02.png "所在路径")

## 1.3 编辑源代码

使用你常用的编辑器打开合约路径下的src/foo_bar.cpp。

我使用的是Notepad++编辑器。

这里我们简单修改一下。把权限验证的第16行代码注释掉。

![编辑源代码](/images/eost17-03.png "编辑源代码")

# 2 编译和部署合约

## 2.1 编译合约

可以逐个编译生成ABI文件和WAST文件。

也可以用contract.build()一次编译两个文件。

这里我们使用逐个编译。

```Bash
	contract.build_abi()
	contract.build_wast()
```

![编译合约](/images/eost17-04.png "编译合约")

## 2.2 部署合约上链

初始化本地testnet

```Bash
	reset()
```

![编译合约](/images/eost17-05.png "编译合约")

创建主账户master

```Bash
	create_master_account("master")
```

![创建主账户](/images/eost17-06.png "创建主账户")

使用master主账号创建合约账户host

```Bash
	create_account("host", master)
```

![创建合约账户](/images/eost17-07.png "创建合约账户")

将账户host和合约绑定。

```Bash
	contract = Contract(host, contract.path())
```

部署合约。

```Bash
	contract.deploy()
```

![部署合约](/images/eost17-08.png "部署合约")

# 3 测试合约

## 3.1 创建测试账号

```Bash
	create_account("alice", master)
	create_account("carol", master)
```

![部署合约](/images/eost17-09.png "部署合约")

## 3.2 调用合约

```Bash	
	contract.push_action("hi", {"user":alice}, permission=alice)
	contract.push_action("hi", {"user":alice}, permission=carol)
```

![调用合约](/images/eost17-10.png "调用合约")

因为我们注释掉了权限验证代码行。

所以，用alice签名还是用carol签名User为alice，都能顺利通过。

# 4 修改代码，重新编译部署

## 4.1 修改代码

打开合约路径下的src/foo_bar.cpp，这次将第16行的权限验证代码取消注释，使之生效。

![修改代码](/images/eost17-11.png "修改代码")

## 4.2 重新编译合约

```Bash
	contract.build()
```

## 4.3 重新部署合约

```Bash
	contract.deploy()
```

## 4.4 测试合约

再次调用合约

```Bash
	contract.push_action("hi", {"user":alice}, permission=alice)
	contract.push_action("hi", {"user":alice}, permission=carol)
```

![测试合约](/images/eost17-12.png "测试合约")

会发现用alice签名User为alice通过。

而用carol签名User为alice，提示没有权限。

说明权限代码生效。

## 4.5 清理环境

关闭本地testnet

```Bash
	stop()
```
	
退出Python CLI

```Bash
	exit()
```

# 5 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- 使用EOSFactory构建和部署智能合约: http://eosfactory.io/build/html/tutorials/03.BuildingAndDeployingEOSContractsInEOSFactory.html

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。

# 下一篇：<a href="https://github.com/eoswing/eos-tutorial/blob/master/eos-tutorial-18.md" target="_blank">（十八）Visual Studio Code和EOSFactory的结合使用</a>	