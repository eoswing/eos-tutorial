（十五）配置windows10下的EOSFactory开发测试框架
===================================

# 手把手教你玩eos 
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第十五篇。本篇教程讲解如何在windows10中安装配置EOSFactory开发测试框架。


## 0.2 学习内容
1. 相关概念
2. 在WSL中配置EOS环境
3. 安装EOSFactory

## 0.3 机器环境

- 笔记本电脑
- 操作系统：Windows 10

# 1 相关概念

## 1.1 技术配置栈

主要是依托windows10的WSL功能，在其中安装运行ubuntu作为EOS的测试环境,安装eosio.cdt开发包，采用EOSFactory开发测试框架。

## 1.2 WSL
WSL全称Windows Subsystem for Linux,是一个为在Windows 10上能够原生运行Linux二进制可执行文件（ELF格式）的兼容层。通俗来讲是在Windows10 嵌入了个Linux子系统，方便运行大部分Linux 命令及软件。我们的windows10下EOS开发环境，很大部分就运行在WSL里面。

## 1.3 eosio.cdt
EOSIO.CDT是WebAssembly（WASM）的工具链和一组工具，用于EOSIO平台的智能合约编译。

## 1.3 EOSFactory
EOSFactory是由Tokenika创建的基于Python3的EOS智能合约开发和测试框架。目标是实现与以太坊Truffle框架类似的功能。
EOSFactory将EOS智能合约开发中一些繁琐枯燥工作简化。通过脚本自动化执行：编译智能合约，创建新的测试网络，部署合同，调用其方法并验证响应，然后撤除测试网络，最后报告结果等等。

EOSFactory由两层组成：

- C++客户端（即cleos）连接到EOS节点（即nodeos）运行私有或公共testnet，
- Python包装器充当操作调用接口。


# 2 在WSL中配置EOS环境

## 2.1 启用WSL功能

在 命令行界面 输入winver,查看windows10版本，确保在1803以上。
![查看windows10版本](/images/eost15-01.png "查看windows10版本")

在 控制面板 中打开 “启用或关闭 Windows 功能” ，然后滚动至底部，如截图所示，勾选 “适用于 Linux 的 Windows 子系统”，点击确定。它将会下载安装需要的包。

![勾选linux](/images/eost15-02.png "勾选linux")

然后，重启windows10系统。

## 2.2 安装Ubuntu18.04

在Microsoft Store中搜索ubuntu,安装Ubuntu 18.04 LTS。

![安装ubuntu](/images/eost15-03.png "安装ubuntu")

安装完成后，在开始菜单就可以找到ubuntu。

![查找ubuntu](/images/eost15-04.png "查找ubuntu")

## 2.3 安装EOSIO

在硬盘空间较大的盘符下设置工作目录。这里我新建了F:/EOS文件夹。
打开Ubuntu，进入命令行终端。
首先进入工作目录：

```Bash
	cd /mnt/f/EOS
```
	
然后，在github上查看最新版本https://github.com/EOSIO/eos/releases。
目前最新发行版是1.5.2。
新建一个tools目录，下载到本地。

```Bash
	mkdir tools
	cd tools
	wget https://github.com/EOSIO/eos/releases/download/v1.5.2/eosio_1.5.2-1-ubuntu-18.04_amd64.deb
```

![下载EOSIO](/images/eost15-05.png "下载EOSIO")

安装依赖项

```Bash
	sudo apt-get update
	wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
	sudo apt-get install clang-4.0 lldb-4.0 libclang-4.0-dev cmake make libbz2-dev libssl-dev libgmp3-dev autotools-dev build-essential libbz2-dev libicu-dev python-dev autoconf libtool git mongodb
	sudo apt-get install libstdc++6
```

编译安装openssl 1.1版本
去官网下載： https://www.openssl.org/source/ ，我下载的是 openssl-1.1.0j.tar.gz 這個版本。
解压

```Bash
	tar -zxvf openssl-1.1.0j.tar.gz	
```

编译安装

```Bash
	cd openssl-1.1.0j
	./config --prefix=/usr/local/ssl shared zlib
 	make depend 
 	make && make install
```

添加软链接

```Bash
	sudo ln -s /usr/local/lib/libssl.so.1.1 /usr/lib/libssl.so.1.1
	sudo ln -s /usr/local/lib/libcrypto.so.1.1 /usr/lib/libcrypto.so.1.1
```

更新ubuntu环境

```Bash
	sudo add-apt-repository ppa:ubuntu-toolchain-r/test
	sudo apt-get update
	sudo apt-get upgrade
	sudo apt-get dist-upgrade
```

安装EOSIO

```Bash
	sudo apt install ./eosio_1.5.2-1-ubuntu-18.04_amd64.deb
```

验证下安装情况

```Bash
	cleos --help
```

![验证cleos](/images/eost15-07.png "验证cleos")

```Bash
	nodeos --help
```

![验证nodeos](/images/eost15-08.png "验证nodeos")

```Bash
	keosd --help
```

![验证keosd](/images/eost15-09.png "验证keosd")

## 2.4 安装eosio.cdt

```Bash
	cd /mnt/f/EOS/tools
	wget https://github.com/eosio/eosio.cdt/releases/download/v1.4.1/eosio.cdt-1.4.1.x86_64.deb

	sudo apt install ./eosio.cdt-1.4.1.x86_64.deb
```

# 3 安装EOSFactory

## 3.1 配置Python3环境

通常来说，ubuntu18.04里是有python3的，但是没有pip3。
查看python3

```Bash
	python3 --version
```

![查看python3](/images/eost15-11.png "查看python3")

安装python3的pip。

```Bash
	sudo apt-get install python3-pip

	pip3 install --upgrade pip
```

此时如果输入 pip3 --version
提示错误大致为：cannot import name 'main'。
还需要修改/usr/bin/pip3文件。

```Bash
	sudo vim /usr/bin/pip3 
```

打开文件，并将文件修改为

```Bash
	from pip import __main__

	if __name__ == '__main__':
    		sys.exit(__main__._main())
```

修改后，/usr/bin/pip3文件如下图所示：
![修改pip3](/images/eost15-14.png "修改pip3")

保存退出后即可完成pip3的更新。

再次查看：
![查看pip3](/images/eost15-15.png "查看pip3")

## 3.2 安装EOSFactory

从github下载代码

```Bash
	cd /mnt/f/EOS
	git clone https://github.com/tokenika/eosfactory.git
```

![下载代码](/images/eost15-10.png "下载代码")

创建contracts文件夹

```Bash
	mkdir contracts
```

安装EOSFactory

```Bash
	cd eosfactory
	./install.sh
```

![安装EOSFactory](/images/eost15-12.png "安装EOSFactory")

中间遇到提示时，输入contreacts文件夹的地址。

![输入contreacts](/images/eost15-13.png "输入contreacts")

## 3.3 执行测试脚本

运行测试脚本01_hello_world.py

```Bash
	python3 tests/01_hello_world.py
```

![测试脚本01](/images/eost15-16.png "测试脚本01")

![测试脚本01](/images/eost15-17.png "测试脚本01")


运行测试脚本02_eosio_token.py

```Bash
	python3 tests/02_eosio_token.py
```

![测试脚本02](/images/eost15-18.png "测试脚本02")

![测试脚本02](/images/eost15-19.png "测试脚本02")

运行测试脚本03_tic_tac_toe.py

```Bash
	python3 tests/03_tic_tac_toe.py
```

![测试脚本03](/images/eost15-20.png "测试脚本03")

![测试脚本03](/images/eost15-21.png "测试脚本03")

至此，在windows10系统下的EOSfactory开发测试框架就安装完成了。

# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- 安装EOSFactory: http://eosfactory.io/build/html/tutorials/01.InstallingEOSFactory.html

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。