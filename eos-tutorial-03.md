
（三）使用智能合约创建和发放代币
===================================

# 手把手教你玩eos
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第三篇,主要是讲解了如何使用系统自带的eos.token合约来创建代币，发放代币和账户转账。


## 0.2 学习内容
1. 理解相关概念
2. 创建账户和导入智能合约
3. 创建、发放代币和账户转账

## 0.3 机器环境

- cpu: 1核
- 内存: 2G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。


**提示：以下命令行默认在root权限下执行。如遇权限问题，请在命令前加sudo。**


# 1.理解相关概念
## 1.1 智能合约概念
简单地说，**智能合约就是传统合约的数字化版本**。
![智能合约概念](/images/eost03-00.png "智能合约概念")

智能合约是运行在计算机里面的，用于保证让参与方执行承诺的代码。它们是在区块链数据库上运行的计算机程序，可以在满足其源代码中写入的条件时自行执行。智能合约一旦编写好就可以被用户信赖，合约条款不能被改变，因此合约是不可更改的。

## 1.2 EOS的智能合约实现

EOSIO智能合约是在区块链上注册并在EOSIO节点上执行的软件。智能合约定义了接口（命令，参数，数据结构）和实现接口的代码。  
代码被编译成规范的字节码格式，节点可以检索和执行。区块链存储合同的交易（例如，资产合法转移，游戏数据变更）。每份智能合约都必须附有一份李嘉图合同，该合同定义了合同中具有法律约束力的条款和条件。

# 2 创建账户和导入智能合约
## 2.1 创建合约用账户
> 如果你在学习此讲前，已经退出了运行环境。请参考第二讲中的*2.1 进入eos运行环境*部分。  
> 如果xiao钱包没有解锁，请先解锁。

### 生成公钥-私钥对
```Bash
    cleos create key
```

命令行输出如下:
![生成秘钥](/images/eost03-01.png "生成秘钥")

> 为简化操作，这里只生成了一对公钥-私钥对。对合约用账户eos.toekn的owner权限和active权限都使用同一个公钥-私钥对。

### xiao钱包导入公钥-私钥对
```Bash
	cleos wallet import -n xiao --private-key 5JitNtAj18S3q31L3XpEVmd1aPeNo35TWDk3SqTkwzAo9xxPxg7
```

命令行输出如下:
![导入钥对](/images/eost03-02.png "导入钥对")

### 创建eos.token账户
```Bash
	cleos create account eosio eosio.token EOS6PLbqkWQey7JSeoS9GXAwdp2Nu7o3rKCiaFEpA92Luhzkiixrm EOS6PLbqkWQey7JSeoS9GXAwdp2Nu7o3rKCiaFEpA92Luhzkiixrm
```

命令行输出如下:
![导入钥对](/images/eost03-03.png "导入钥对")

## 2.2 eos.token导入智能合约
eos系统自带有eosio.token合约。此合约允许创建许多不同的令牌。  
合约位置为 /contracts/eosio.token

### 导入智能合约
```Bash
	cleos set contract eosio.token /contracts/eosio.token -p eosio.token@active
```

命令行输出如下:
![导入合约](/images/eost03-04.png "导入合约")

### eosio.token支持的命令接口

查看下abi文件：
```Bash
	cat /contracts/eosio.token/eosio.token.abi
```

可以在输出中看到actions：
![接口定义](/images/eost03-0402.png "接口定义")

其中，create是创建代币（或者是令牌、token等不同叫法），issue是发放代币，而transfer是账户转帐。

# 3 创建、发放代币和账户转账
## 3.1 创建代币
先实现1个亿的小目标吧，我们创建1亿个EOS的代币：
```Bash
	cleos push action eosio.token create '[ "eosio", "100000000.0000 EOS"]' -p eosio.token@active
```

命令行输出如下：
![创建代币](/images/eost03-05.png "创建代币")

## 3.2 发放代币
给xiaoaccount帐户发放100个EOS:
```Bash
	cleos push action eosio.token issue '[ "xiaoaccount", "100.0000 EOS", "memo" ]' -p eosio@active
```

命令行输出如下：
![发放代币](/images/eost03-06.png "发放代币")

查看下xiaoaccount现在的资产：
```Bash
	cleos get currency balance eosio.token xiaoaccount
```

命令行输出如下：
![xiao代币资产](/images/eost03-07.png "xiao代币资产")

可以看到，xiaoaccount已经收到了100个EOS。

## 3.3 账户转账
现在xiaoaccount给eosio账户转账25个EOS:
```Bash
	cleos push action eosio.token transfer '[ "xiaoaccount", "eosio", "25.0000 EOS", "m" ]' -p xiaoaccount@active
```

命令行输出如下：
![账户转账](/images/eost03-08.png "账户转账")

查看下eosio现在的资产：
```Bash
	cleos get currency balance eosio.token eosio
```

命令行输出如下：
![xiao代币资产1](/images/eost03-10.png "xiao代币资产1")

可以看到，eosio账户上已经有25个EOS。

再查看下xiaoaccount现在的资产：
```Bash
	cleos get currency balance eosio.token xiaoaccount
```

命令行输出如下：
![xiao代币资产2](/images/eost03-09.png "xiao代币资产2")

可以看到，xiaoaccount账户上已经扣除了25个EOS，只有75个EOS。
账户转账成功。

# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- EOSIO令牌合同简介: https://developers.eos.io/eosio-cpp/docs/token-tutorial

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。

# 下一篇：<a href="https://github.com/eoswing/eos-tutorial/blob/master/eos-tutorial-04.md" target="_blank">（四）编写第一个智能合约Hello_eos</a>