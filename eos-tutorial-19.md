（十九）在公共Testnet上部署和测试智能合约
===================================

# 手把手教你玩eos 
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第十九篇。本篇教程演示如何使用EOSFactory在公共testnet上部署和测试EOS智能合约，例如Jungle Testnet或CryptoKylin Testnet。

## 0.2 学习内容
1. 注册账号
2. 部署和测试智能合约
3. 两则技巧

## 0.3 机器环境

- 笔记本电脑
- 操作系统：Windows 10

# 1. 注册账号

## 1.1 在丛林测试网上注册账号 

我们在eosfactory中注册账号：

```Bash
	cd /mnt/f/EOS/eosfactory
	python3 utils/register_testnet.py http://jungle2.cryptolions.io:80 myjungle
```

![注册账号](/images/eost19-01.png "注册账号")

将对应的账号名和公钥复制下来。

在浏览器中打开丛林测试网 Jungle Testnet 2.0：

https://monitor.jungletestnet.io/

点击“Create Account”，在弹出对话框中输入对应账户名和公钥（注意，不是私钥），完成创建。

![注册账号2](/images/eost19-02.png "注册账号2")

完成人机身份验证（注意，如果网络不好的，可能会一直刷不出来）
然后，点击Create，完成账户注册。

因为是公共测试网，而不是本地节点，为了方便合约开发。

我们要通过水龙头，给该账户申请一些EOS，然后抵押资源。

确保RAM,cpu,net等资源充足。

在https://monitor.jungletestnet.io/上点击“faucet”菜单获取。6小时可以获取一次。

![水龙头](/images/eost19-03.png "水龙头")

准备好了，我们再回到命令行，在提示符后面输入 go


![注册完成](/images/eost19-04.png "注册完成")

验证注册完成。

## 1.2 在麒麟测试网上注册账号

麒麟测试网上注册要更自动化一些。

三个参数分别是：对应的水龙头网网址，公共节点网址，账号别名。

然后，就会自动完成注册和水龙头获取代币。

```Bash
	python3 utils/register_testnet_via_faucet.py http://faucet.cryptokylin.io http://kylin.fn.eosbixin.com mykylin
```

![注册完成](/images/eost19-05.png "注册完成")

## 1.3 查看测试网账号

```Bash
	python3 utils/testnets.py
```

![查看测试网账号](/images/eost19-06.png "查看测试网账号")

# 2. 部署和测试智能合约

## 2.1 创建新的智能合约

目前，还只有 03_tic_tac_toe 模板支持测试网。所以我们基于这个模板创建

```Bash
	python3 utils/create_project.py foo_bar_net 03_tic_tac_toe --vsc
```

![模板创建](/images/eost19-07.png "模板创建")

## 2.2 在丛林测试网上测试

```Bash
	python3 tests/unittest1.py myjungle
```

![丛林测试1](/images/eost19-08.png "丛林测试1")

```Bash
	python3 tests/test1.py myjungle
```

![丛林测试2](/images/eost19-09.png "丛林测试2")

## 2.3 在麒麟测试网上测试

```Bash
	python3 tests/unittest1.py mykylin
```

![麒麟测试1](/images/eost19-10.png "麒麟测试1")

```Bash
	python3 tests/test1.py mykylin
```

![麒麟测试2](/images/eost19-11.png "麒麟测试2")

# 3 两则技巧
## 3.1 清空账号

测试网中建立的账号在测试时是重复利用的。
如果您要从头开始运行测试，即把现有帐户从EOSFactory缓存中删除并替换为另一组新创建的帐户，请在命令后面添加该参数： -r

```Bash
	python3 tests/unittest1.py myjungle -r
```

## 3.2 使用自己原来注册好的测试网账号

如果您在公共测试网上有帐户，则可以跳过使用EOSFactory注册账号，并直接运行单元测试。

方法是使用-t（或--testnet）选项。

此选项需要四个参数：

- 提供对testnet的访问的公共节点的URL，例如http://kylin.fn.eosbixin.com，
- 您的帐户名称，在testnet上注册，
- 您帐户的Owner私钥，
- 您帐户的Active私钥。

命令类似于：

```Bash
	python3 tests/unittest1.py -t http://kylin.fn.eosbixin.com dgxo1uyhoytn 5JE9XSurh4Bmdw8Ynz72Eh6ZCKrxf63SmQWKrYJSXf1dEnoiKFY 5JgLo7jZhmY4huDNXwExmaWQJqyS1hGZrnSjECcpWwGU25Ym8tA
```


# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- 与公共Testnet交互: http://eosfactory.io/build/html/tutorials/05.InteractingWithPublicTestnet.html


## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。