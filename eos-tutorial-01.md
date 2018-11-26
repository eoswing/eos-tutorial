
（一）使用docker搭建eos本地运行环境
===================================

# 手把手教你玩eos
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。
本文是第一篇,主要是教你如何在linux环境下，安装docker,安装eos镜像，验证三大组件。
通过本文的学习，你会对eos有一个初步直观的印象。


## 0.2 学习内容
1. 安装docker
2. 安装eos镜像
3. 验证三大组件

## 0.3 机器环境
官网推荐配置：7G内存空间，20G硬盘空间。

![配置](/images/eost01-01.png "配置")


不过，不要被官网推荐的配置吓到了。
其实实践中，配置低一点也没问题。

比如本系列教程中，采用的系统环境配置就很屌丝：

- cpu: 1核
- 内存: 2G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。


**提示：以下命令行默认在root权限下执行。如遇权限问题，请在命令前加sudo。**


# 1.安装docker
## 1.1安装存储库
### 安装所需的包
```Bash
    yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 配置
```Bash
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

## 1.2安装Docker CE
### 安装最新版的Docker  CE
```Bash
    yum install docker-ce
```

### 启动Docker
```Bash
    systemctl start docker
```

### 运行Hello-world镜像，验证dockers是否正确安装
```Bash
	docker run hello-world
```

此命令下载测试映像并在容器中运行它。当容器运行时，它会打印一条信息性消息并退出。

如果在命令行中看到类似如下显示，说明安装成功。

![docker](/images/eost01-02.png "docker")

# 2.安装eos镜像
## 2.1拉取官方eos开发镜像
eos-dev是官方为本地开发而制作的eosio软件的编译版本
```Bash
	docker pull eosio/eos-dev
```

## 2.2启动EOSIO节点


> 感谢网友<font color=#0099ff >@Seven、苹果控</font> 反馈，已更新适配EOS 1.30以上版本。

```Bash
	docker run --rm --name eosio -d -p 8888:8888 -p 9876:9876 -v /tmp/work:/work -v /tmp/eosio/data:/mnt/dev/data -v /tmp/eosio/config:/mnt/dev/config eosio/eos-dev  /bin/bash -c "nodeos -e -p eosio --plugin eosio::producer_plugin --plugin eosio::history_plugin --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin --plugin eosio::http_plugin -d /mnt/dev/data --config-dir /mnt/dev/config --http-server-address=0.0.0.0:8888 --access-control-allow-origin=* --contracts-console --http-validate-host=false"
```


## 2.3验证是否正常工作
### 出块检查
```Bash
	docker logs --tail 10 eosio
```

如果看到如下输出：
![出块检查](/images/eost01-03.png "出块检查")

恭喜！您已经在Docker容器中运行了一个的eos单节点！

### 检查RPC接口
```Bash
	curl http://localhost:8888/v1/chain/get_info
```

您应该看到类似于以下内容的消息：
![RPC接口](/images/eost01-04.png "RPC接口")

# 3.验证三大组件
## 3.1了解三大组件
eos架构图:
![eos架构](/images/eost01-05.png "eos架构")

eos主要由以下三个组件构成:

- nodeos（node + eos = nodeos） - 节点守护程序。负责块生产，提供API端口等。
- cleos （cli + eos = cleos） - 命令行界面，用于与节点交互和管理钱包。
- keosd （key + eos = keosd） - 将EOSIO密钥安全存储在钱包中的组件。

## 3.2验证三大组件
### 进入docker容器
进入eosio容器，后续的命令都在该界面中执行。
```Bash
	docker exec -it eosio /bin/bash
```

### 验证nodeos
```Bash
	nodeos --help
```

您应该看到以下输出：
![验证nodeos](/images/eost01-06.png "验证nodeos")

### 验证cleos
```Bash
	cleos --help
```

您应该看到以下输出：
![验证cleos](/images/eost01-07.png "验证cleos")

### 验证keosd
```Bash
	keosd --help
```

您应该看到以下输出：
![验证keosd](/images/eost01-08.png "验证keosd")

## 3.3 退出和关闭docker容器
### 退出容器
按Ctrl+P+Q进行退出容器
### 关闭容器（可选）
eos一直运行会不断出块，占用资源。测试用机器如果配置不够好后，建议及时关闭。
```Bash
	docker stop eosio
```

> 关闭eos容器后，所有数据会清零。

# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- 在centos环境下安装Docker CE: https://docs.docker.com/install/linux/docker-ce/centos/

- Docker快速入门: https://developers.eos.io/eosio-nodeos/docs/docker-quickstart

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。

# 下一篇：<a href="https://github.com/eoswing/eos-tutorial/blob/master/eos-tutorial-02.md" target="_blank">（二）钱包和账户的创建与管理</a>
