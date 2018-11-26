
（二）钱包和账户的创建与管理
===================================

# 手把手教你玩eos
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第二篇,主要是讲解了钱包与账户的关系，并教你如何使用cleos创建与管理eos钱包，钱包导入公钥-私钥对，以及创建和查看账户。


## 0.2 学习内容
1. 理解相关概念
2. 创建和管理钱包
3. 生成和导入公钥-私钥对
4. 创建和查看账户

## 0.3 机器环境

- cpu: 1核
- 内存: 2G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。


**提示：以下命令行默认在root权限下执行。如遇权限问题，请在命令前加sudo。**


# 1.理解相关概念
## 1.1 回顾一下架构图
再看看上篇教程发的架构图，此图务必反复看，深入理解。
![架构图](/images/eost02-01.png "架构图")

钱包其实就是存放公钥-私钥对的加密存储库，由keosd组件负责管理。

账户是区块链上的一个访问凭证，由nodeos组件负责管理。

cleos可以理解为一个命令行工具。主要作用是keosd组件和nodeos组件之间的中介，将两者联系起来。


## 1.2 更深入的理解
看了上面的是不是感觉有点绕口？
我将这个架构图，针对账号和钱包之间关系画了一个更详细的解读图。
![推荐配置](/images/eost02-02.png "解读图")

先看右边的nodeos账户体系。  
账户由一个唯一名称来标识，名称的最大长度为12个字符。在账户里有token、智能合约等等资产。  
区块链的透明性使得每一个账户的情况，任何人都可以查看。而管理使用这个账户的权限分配给了n个拥有不同权限的公钥。  

 eos系统默认是owner权限和active权限两个公钥，后续自己可以自定义更多的公钥。

- owner权限是最高权限，象征着帐户的所有权。只有少数交易需要此权限。通常，建议所有者保持冷藏，不与任何人共享。owner可用于恢复可能已被泄露的其他权限。
- active权限用于转移资金，投票给生产者等。

谁拥有对应公钥的私钥对，就拥有了对应的公钥管理账户权限。
比如，owner权限是最高权限，拥有了owner权限私钥，就完全掌握了整个eos账户。

再看左边的keosd钱包体系。钱包就是存放私钥的地方。通过钱包对应的私钥对签名，就可以对账户进行相关权限的操作。
> 和现实中的钱包可以存放许多银行卡一样，eos钱包可以放n多的公钥-私钥对。

钱包有名称，也有对应的密码。使用钱包密码才能操作钱包。  
钱包有两种状态，锁定和解锁状态。keosd默认设置是15分钟没有操作就自动锁定钱包。

# 2 创建和管理钱包
## 2.1 进入eos运行环境
如果你在学习了第一讲后退出了运行环境，关闭了eos镜像。请在进入centos7后，进行如下操作：

### 启动eos镜像
```Bash
    docker run --rm --name eosio -d -p 8888:8888 -p 9876:9876 -v /tmp/work:/work -v /tmp/eosio/data:/mnt/dev/data -v /tmp/eosio/config:/mnt/dev/config eosio/eos-dev  /bin/bash -c "nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::wallet_plugin --plugin eosio::producer_plugin --plugin eosio::history_plugin --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin --plugin eosio::http_plugin -d /mnt/dev/data --config-dir /mnt/dev/config --http-server-address=0.0.0.0:8888 --access-control-allow-origin=* --contracts-console --http-validate-host=false"
```

### 进入docker容器
```Bash
    docker exec -it eosio /bin/bash
```

### 验证cleos
```Bash
	cleos --help
```

## 2.2 创建钱包
### 创建第一个钱包
```Bash
    cleos wallet create --to-console
```

命令行输出如下：
![创建钱包](/images/eost02-03.png "创建钱包")

现在keosd已经为我们创建了一个名称为default的钱包，并且生成了钱包的秘钥："PW5J3SQJefLLcFj6WbourV8tthPhr52Xvsr9d2XYs4UY9RpBGzjqy"。

> 如果是实际使用此钱包来管理账号私钥，请务必将此密码保存到安全的地方。

这里有一点比较容易混淆，就是钱包的秘钥和下面将接触到的钱包里的私钥。为区别开来，钱包的秘钥我在今后教程中将统一表述成**钱包的密码**。
对两者有不理解的，请反复看下上面我画的解读图。

### 再创建一个钱包
创建钱包时可以在后面加上 -n 的参数，指定钱包名称。下面我们创建一个名称为xiao的钱包。  
```Bash
    cleos wallet create -n xiao --to-console
```

命令行输出如下：
![创建钱包xiao](/images/eost02-04.png "创建钱包xiao")


钱包xiao的密码就是"PW5JzuEJJD3iEyN6uE36A23sGCtFRSs3ax7UUbdBrWZgFpS1yQxBU"。

### 查看钱包列表
看一下我们刚刚创建的两个钱包:
```Bash
    cleos wallet list
```

命令行输出如下：
![查看钱包列表](/images/eost02-05.png "查看钱包列表")

细心的朋友会发现两个钱包中，xiao后面有个"\*"号，而default后面没有"\*"号。  
钱包后面这个星号（\*）很重要，代表着相应的钱包已解锁。这说明xiao钱包处于解锁状态，而default钱包则处于锁定状态。

> 如果你是处于连续创建账号的，可能你会看到两个账号都是解锁状态。同时，keosd默认设置是15分钟没有操作就自动锁定钱包。如果你耽搁时间久，可能看到两个账号都是锁定状态。

## 2.3 钱包的锁定与解锁
### 钱包的锁定
我们锁定xiao钱包:
```Bash
    cleos wallet lock -n xiao
```

命令行输出如下:
![锁定xiao钱包](/images/eost02-06.png "锁定xiao钱包")

再次查看钱包列表:
```Bash
    cleos wallet list
```

命令行输出如下:
![查看钱包列表2](/images/eost02-07.png "查看钱包列表2")

可以看到，xiao钱包后面的星号（\*）已经没有了，已变为锁定状态。

> 注意，在锁定钱包时，不需要输入钱包的密码。但是在解锁钱包时需要输入密码。

### 钱包的解锁
下面我们解锁xiao钱包：
```Bash
    cleos wallet unlock -n xiao
```

回车后，会提示我们输入密码（password）。输入正确密码后，解锁xiao钱包。
![解锁xiao钱包](/images/eost02-08.png "解锁xiao钱包")

>在这里为了演示方便，我采用了加"--password"参数附加密码的方式。但在生产环境中不建议如此，因为会导致密码记录到命令行历史记录中。

再次查看钱包列表:
```Bash
    cleos wallet list
```

命令行输出如下:
![查看钱包列表3](/images/eost02-09.png "查看钱包列表3")

可以看到，xiao钱包后面出现星号（\*），已变为解锁状态。

# 3 生成和导入公钥-私钥对
## 3.1 生成公钥-私钥对
生成两个公钥/私钥对。连续输入两次命令:
```Bash
    cleos create key
```

命令行输出如下:
![生成秘钥](/images/eost02-10.png "生成秘钥")

## 3.2 导入公钥-私钥对到钱包
> 如果xiao钱包还没解锁，记得先解锁。

将上面生成的两个公钥-私钥对都导入xiao钱包：
```Bash
    cleos wallet import -n xiao --private-key XXXXXXXXXXXXXXX(上面生成的private key)
```

命令行输出如下:
![导入私钥1](/images/eost02-11.png "导入私钥1")

![导入私钥2](/images/eost02-12.png "导入私钥2")

## 3.3 查看导入钱包的公钥-私钥对
只查看公钥：
```Bash
	cleos wallet keys
```

命令行输出如下:
![查看钥1](/images/eost02-13.png "查看钥1")

输入钱包密码，查看公钥-私钥对:
```Bash
	cleos wallet private-keys -n xiao --password PW5JzuEJJD3iEyN6uE36A23sGCtFRSs3ax7UUbdBrWZgFpS1yQxBU
```

命令行输出如下:
![查看钥2](/images/eost02-14.png "查看钥2")

## 3.4 备份钱包

钱包文件默认存放在~/eosio-wallet文件夹中。
```Bash
	cd ~/eosio-wallet && ls
```

命令行输出如下:
![钱包文件地址](/images/eost02-15.png "钱包文件地址")

可以看到default.wallet和xiao.wallet两个钱包文件。

现在您的钱包中包含密钥，最好养成将钱包备份的习惯，例如存在U盘里，以防止丢失钱包文件。


# 4 创建和管理账户
## 4.1 命令行解读
eos的新账户需要有原有的账户创建。创建命令行格式如下:
```Bash
	cleos create account authorizing_account NEW_ACCOUNT OWNER_KEY ACTIVE_KEY
```

- authorizing_account 是为帐户创建提供资金的帐户的名称。
- new_account 是您要创建的帐户的名称。
- owner_key是分配给帐户owner权限的公钥。
- active_key是分配给您帐户的active权限的公钥。

新帐户名称必须符合以下准则：

- 必须在12个字符以内，包括12字符。
- 只能包含以下符号：.12345abcdefghijklmnopqrstuvwxyz
> 请注意，账户名称不允许使用6,7,8,9,0。

## 4.2 使用eosio初始账户创建新账户
eosio帐户是用于引导EOSIO节点的特殊帐户。  
由于我们没有其他账户，所以用初始eosio账户来创建新帐户。

> eosio帐户的密钥可以在nodeos配置文件中找到，位于~/.local/share/eosio/nodeos/config/config.ini。
> 同时，记得解锁xiao钱包

首先，需要将默认账号eosio的私钥导入xiao钱包：
```Bash
	cleos wallet import -n xiao --private-key  5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

命令行输出如下：
![eosio导入私钥](/images/eost02-16.png "eosio导入私钥")

然后，用初始eosio账户来创建新帐户xiaoaccount：
```Bash
	cleos create account -x 1000 eosio xiaoaccount EOS6FiGRKoJsE1YwwqnQ2TWxV3An1tcwiL5X6H1S75Mr3ijjCtDsh EOS5vfNsEWhxwuzxts7Mg29gEQW3qNibb4FUvn7pWcppNJm783a95
```

命令行输出如下：
![创建新帐户](/images/eost02-17.png "创建新帐户")


## 4.3 查看账户信息
```Bash
	cleos get account xiaoaccount -j
```

命令行输出如下：
![查看账户信息](/images/eost02-18.png "查看账户信息")

至此，xiaoaccount账户创建成功。

# 5 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- 钱包与账户: https://developers.eos.io/eosio-nodeos/docs/learn-about-wallets-keys-and-accounts-with-cleos

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。

# 下一篇：<a href="https://github.com/eoswing/eos-tutorial/blob/master/eos-tutorial-03.md" target="_blank">（三）使用智能合约创建和发放代币</a>
