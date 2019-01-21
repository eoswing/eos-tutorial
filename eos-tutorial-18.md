（十八）Visual Studio Code和EOSFactory的结合使用
===================================

# 手把手教你玩eos 
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第十八篇。本篇教程讲解如何将EOSFactory与Visual Studio Code结合使用，以简化使用EOS智能合约的过程。


## 0.2 学习内容
1. 安装和配置Visual Studio Code
2. 在VSCode中开发智能合约项目
3. 在VSCode中优化工作

## 0.3 机器环境

- 笔记本电脑
- 操作系统：Windows 10

# 1.安装和配置Visual Studio Code

## 1.1 关于VSCode
在前面的教程里，我们大部分时间使用vi来编辑代码，但是对于大部分初学者来说，这是很不方便的。
在这里我们将配置vscode作为我们开发eos智能合约用IDE。

Visual Studio Code (简称 VSCode) 是一款免费开源的现代化轻量级代码编辑器，支持几乎所有主流的开发语言的语法高亮、智能代码补全、自定义快捷键、括号匹配和颜色区分、代码片段、代码对比 Diff、GIT命令 等特性，支持插件扩展，并针对网页开发和云端应用开发做了优化。软件运行流畅，功能强大。


## 1.2 在官网下载VSCode
进入官网：https://code.visualstudio.com/Download，选择Windows版本的安装包下载。

![下载VSCode](/images/eost18-01.png "下载VSCode")

安装过程比较简单，不再赘述。

## 1.3 安装C/C++ 插件

EOS是用c++开发的，所以打开vscode之后先安装c++插件

![安装c++插件](/images/eost18-02.png "安装c++插件")

点击右侧最下方图标，在出现的插件列表里，选择C/C++ 插件。点击Install安装。

我这里已经安装好插件了，所以是一个设置图标。 

## 1.4 配置命令行调用

首先打开设置面板。File -> Preferences -> Settings

![打开设置面板](/images/eost18-03.png "打开设置面板")

然后，在Features -> terminal -> integrated>shell:windows 选项中配置参数为：
C:\\Windows\\sysnative\\bash.exe


![配置参数](/images/eost18-04.png "配置参数")

关闭设置面板。选择View -> Terminal ，在下方打开命令行面板。

![打开命令行面板](/images/eost18-05.png "打开命令行面板")

# 2 在VSCode中开发智能合约项目

# 2.1 创建一个新的智能合约项目

在bash终端中,进入eosfactory文件夹，创建一个新的智能合约。

```Bash
	cd /mnt/f/EOS/eosfactory
	python3 utils/create_project.py foo_bar_vsc 01_hello_world --vsc
```

![创建一个新的智能合约项目](/images/eost18-06.png "创建一个新的智能合约项目")

第一个参数（foo_bar_vsc）是您的合同的名称。它可以是您想要的任何名称，前提是它没有空格。允许使用字母，数字，下划线_，点.和破折号-。

第二个参数（01_hello_world）表示将从中创建新合同的模板。截至目前已经有三个模板可供选择（即01_hello_world，02_eosio_token和03_tic_tac_toe）。此参数是可选的，默认值为01_hello_world。

第三个参数（--vsc）是将项目在VSCode中打开。会启动一个新的VSCode界面，并切换到项目文件夹。

![项目在VSCode中打开](/images/eost18-07.png "项目在VSCode中打开")

# 2.2 项目结构

点开查看下完整的项目结构。


![项目结构](/images/eost18-08.png "项目结构")

在foo_bar_vsc项目文件夹里，智能合约源代码，构建输出文件和单元测试分布在不同文件夹里。


- .vscode目录。     包含IntelliSense定义，任务定义等。
- build目录。       编译后生成文件。
- resources目录。   资源文件，比如Ricardian等合同文件。
- src目录。         源文件，cpp和hpp。
- tests目录。       测试文件。
- CMakeLists.txt   CMake定义文件。

我们打开主要的源文件foo_bar_vsc.cpp查看下:

![源文件查看](/images/eost18-09.png "源文件查看")

vscode中，编辑代码很智能也很方便。

# 2.3 构建智能合约

我们使用cmake和make来构建智能合约。

首先选择View -> Integrated Terminal，在下方打开命令行。

先cmake构建一下：

```Bash
	cd bulid
	cmake ..
```

![cmake](/images/eost18-10.png "cmake")

等待完成后，再make编译。

```Bash
	make
```

![make](/images/eost18-11.png "make")

这样，我们就在build文件夹里编译好了foo_bar_src.abi和foo_bar_src.wasm两个文件。

# 2.4 单元测试

编译好文件后，我们开始单元测试。
输入命令行。

```Bash
	ctest
```

![单元测试](/images/eost18-12.png "单元测试")

如果要使单元测试更详细，请加上-V参数： ctest -V

注意: 您可能已经注意到，有两种类型的单元测试：标准测试（命名为unittest1，unittest2等）和ad-hoc测试（命名为test1，test2等）。

这种二元性的原因在于：我们发现使用标准单元测试来证明事情按预期工作是有用的，并且通过临时测试来调查错误并且通常监视智能合约的内部工作。

EOSFactory支持两者，因此您可以选择任何适合您需求的产品。

当然，我们也可以直接用Python调用测试文件进行测试。

```Bash
	cd ..
	python3 tests/test1.py
	python3 tests/unittest1.py
```

![调用测试](/images/eost18-13.png "调用测试")

# 3 在VSCode中优化工作

在菜单栏选择 Terminal > Run Task。

![菜单栏选择](/images/eost18-14.png "菜单栏选择")

稍等片刻，你会看到下拉菜单有五个选项提示：

![五个选项提示](/images/eost18-15.png "五个选项提示")

分别是：

- Build 完整构建合约，在build文件夹生成ABI和WAST文件。
- Compile 快速完成合约编译，注意，不会构建合约（既不生成ABI也不生成WAST）。如果编译中代码有错，会列出了代码错误。
- EOSIO API 打开EOSIO文档。
- Test 执行test1.py脚本。
- Unittest 执行unittest1.py脚本。

我们先将build文件夹下面的所有文件删除，然后挨个测试下这些命令。

在这里有一个小的环境设置问题。
要将.vscode中的tasks.json中的bash.exe都替换为C:\\Windows\\sysnative\\bash.exe
共有5处，都要修改。
![编译](/images/eost18-16.png "编译")

## 3.1 编译

Terminal > Run Task > Compile

![编译](/images/eost18-17.png "编译")

代码编译通过，没有报错。

将print 修改为 myprint

再次运行编译，会发现报错。

![编译报错](/images/eost18-18.png "编译报错")

再修改会print

## 3.2 构建

将整个build文件夹删除，我们再看看构建的效果。

Terminal > Run Task > Build

![构建](/images/eost18-19.png "构建")

我们看到，自动新建了build文件夹，并生成了abi和wasm文件。

## 3.3 测试

Terminal > Run Task > Test

![测试](/images/eost18-20.png "测试")

## 3.4 单元测试

Terminal > Run Task > Unittest

![单元测试](/images/eost18-21.png "单元测试")

## 3.5 绑定快捷键

我们还可以将这些task绑定成快捷键，来进一步优化工作。

File -> Preferences -> Keyboard Shortcuts 

选择编辑 keybindings.json 文件

![绑定快捷键1](/images/eost18-22.png "绑定快捷键1")

输入类似如下配置文件：

```Bash
	{
        "key": "ctrl+shift+c",
        "command": "workbench.action.tasks.runTask",
        "args": "Compile"
    }
    {
        "key": "ctrl+shift+t",
        "command": "workbench.action.tasks.runTask",
        "args": "Test"
    }
    {
        "key": "ctrl+shift+u",
        "command": "workbench.action.tasks.runTask",
        "args": "Unittest"
    }
```
	
![绑定快捷键2](/images/eost18-23.png "绑定快捷键2")

然后保存退出。这样我们就为Compile,Test和Unittest都绑定了对应的快捷键。

可以在开发环境中测试下，直接输入快捷键的调用情况。

## 3.6 智能提示

点击左侧的扩展快捷，在输入框中输入 C++ IntelliSense

安装C++ IntelliSense插件。

![智能提示1](/images/eost18-24.png "智能提示1")

安装完成后，鼠标移动到对应函数方法，可以看到函数方法提示。


![智能提示2](/images/eost18-25.png "智能提示2")

## 3.7 使用智能合约logger调试

使用智能合约logger实际上是调试智能合约的唯一方法。

比如，在源文件foo_bar_vsc.cpp第15行，就是使用智能合约logger。

```Bash
	logger_info( "debug user name: ", name{user} );
```
	
我们按快捷键ctrl+shift+t。可以在测试中，看到对应的调试信息。

![智能合约logger调试](/images/eost18-26.png "智能合约logger调试")

在示例合约 02_eosio_token 模板中也有类似的调试查看代币数的logger语句:

```Bash
	logger_info("quantity.amount: ", quantity.amount);
```

# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- 使用EOSFactory构建和部署智能合约: http://eosfactory.io/build/html/tutorials/04.WorkingWithEOSContractsUsingEOSFactoryInVSC.html

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。

# 下一篇：<a href="https://github.com/eoswing/eos-tutorial/blob/master/eos-tutorial-19.md" target="_blank">（十九）在公共Testnet上部署和测试智能合约</a>	