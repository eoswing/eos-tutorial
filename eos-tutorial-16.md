（十六）使用EOSFactory与EOS交互
===================================

# 手把手教你玩eos 
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第十六篇。本篇教程讲解如何使用EOSFactory及其Python CLI轻松直观地与EOS进行交互。


## 0.2 学习内容
1. 启动本地测试节点
2. 管理账户和智能合约
3. 运行测试合约
4. 清理环境

## 0.3 机器环境

- 笔记本电脑
- 操作系统：Windows 10

# 1 启动本地测试节点

## 1.1 进入环境

首先进入WSL中的ubuntu的命令行，然后运行Python CLI。

```Bash
	Python3
```

![启动Python3](/images/eost16-01.png "启动Python3")

进入Python shell后，导入EOSFactory库。

```Bash
	from eosfactory.eosf import *
```
	
## 1.2 启动本地测试节点(testnet)

启动单节点本地testnet。

```Bash
	reset()
```

![启动单节点](/images/eost16-02.png "启动单节点")

获取testnet当前状态的信息。

```Bash
	info()
```

![当前状态的信息](/images/eost16-03.png "当前状态的信息")

## 1.3 管理本地testnet

要停止当前的testnet。

```Bash
	stop()
```

![停止当前](/images/eost16-04.png "停止当前")

要继续运行当前的testnet。

```Bash
	resume()
```

![继续运行](/images/eost16-05.png "继续运行")

停止当前的testnet并启动一个新的。

```Bash
	reset()
```

![重启](/images/eost16-06.png "重启")

# 2 管理账户和智能合约

## 2.1 创建主账户

首先，确保本地testnet正在运行。
info()查看，没有就reset()启动一个。

创建一个可以创建其他帐户的主帐户，名称为master。当然，你也可以取其他名称:

```Bash
	create_master_account("master")
```

![创建主账户](/images/eost16-07.png "创建主账户")

注意：名称master只是全局变量的名称，而不是区块链上创建的帐户的实际名称。

您无需担心锁定或解锁钱包，管理其密码或将私钥导入其中。所有这一切都由EOSFactory来处理。

查看master主账户信息。

```Bash
	master.info()
```

![查看master主账号信息](/images/eost16-08.png "查看master主账号信息")

## 2.2 创建子账户

使用master创建测试帐户:

```Bash
	create_account("charlie", master)
```

![创建测试帐户1](/images/eost16-09.png "创建测试帐户1")

查看账户信息：

```Bash
	charlie.info()
```

![查看账户信息1](/images/eost16-10.png "查看账户信息1")

和上面的master一样，名称charlie只是全局变量的名称，而不是区块链上创建的帐户的实际名称。

如果你想命名EOS链上的账号名，可以使用参数account_name。

下面创建一个命名链上账号名的账户charlie2。

```Bash
	create_account("charlie2", master, account_name="charlie22eos")
```

![创建测试帐户2](/images/eost16-11.png "创建测试帐户2")

查看charlie2的账户信息：

```Bash
	charlie2.info()
```

![查看账户信息2](/images/eost16-12.png "查看账户信息2")

## 2.3 管理智能合约

### 定义contract

创建一个部署智能合约用的账户host

```Bash
	create_account("host", master)
```

![创建合同账户](/images/eost16-13.png "创建合同账户")

定义contract,将账户和智能合约所在文件夹绑定。

```Bash
	contract = Contract(host, "/mnt/f/EOS/eosfactory/contracts/02_eosio_token")
```

### 构建合约

```Bash
	contract.build()
```

![构建合约](/images/eost16-14.png "构建合约")

### 部署合约

可以使用code()来检查合约。首先我们看下没有部署合约的code()提示。

```Bash
	contract.code()
```

![查看合约](/images/eost16-15.png "查看合约")

部署合约：

```Bash
	contract.deploy()
```

![部署合约](/images/eost16-16.png "部署合约")

再次查看合约hash:

![查看合约hash](/images/eost16-17.png "查看合约hash")


# 3 运行测试合约

## 3.1 创建代币

首先创建10亿的EOS代币。

```Bash
	host.push_action(
    "create", 
    {
        "issuer": master,
        "maximum_supply": "1000000000.0000 EOS",
        "can_freeze": "0",
        "can_recall": "0",
        "can_whitelist": "0"
    }, [master, host])
```

![创建代币](/images/eost16-18.png "创建代币")

注意：该push_action方法有三个参数：操作的名称，JSON格式的操作参数以及需要权限的帐户。

注意：如果您想要在不广播的情况下查看实际交易，请使用show_action方法代替push_action。

## 3.2 发放代币

发给charlie代币100EOS。

```Bash
	host.push_action(
    "issue",
    {
        "to": charlie, "quantity": "100.0000 EOS", "memo": ""
    },
    master)
```

![发放代币](/images/eost16-19.png "发放代币")

查看下charlie当前账户信息

```Bash
	host.table("accounts", charlie)
```

![账户信息](/images/eost16-20.png "账户信息")

## 3.3 代币转账

现在charlie的代币转账25EOS给charlie2。

```Bash
	host.push_action(
    "transfer",
    {
        "from": charlie, "to": charlie2,
        "quantity": "25.0000 EOS", "memo":""
    },
    charlie)
```

![代币转账](/images/eost16-21.png "代币转账")

再次查看charlie账户信息

```Bash
	host.table("accounts", charlie)
```

![账户信息](/images/eost16-22.png "账户信息")

查看charlie2账户信息

```Bash
	host.table("accounts", charlie2)
```

![账户信息](/images/eost16-23.png "账户信息")

可以看出，已经转账到位。

# 4 清理环境

## 4.1 关闭本地testnet

```Bash
	stop()
```
	
![关闭本地](/images/eost16-24.png "关闭本地")

## 4.2 退出Python CLI

```Bash
	exit()
```

![退出Python](/images/eost16-25.png "退出Python")

# 5 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- 使用EOSFactory与EOS交互: http://eosfactory.io/build/html/tutorials/02.InteractingWithEOSContractsInEOSFactory.html

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。