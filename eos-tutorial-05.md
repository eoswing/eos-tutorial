---
layout: '[layout]'
title: （五）编写智能合约游戏：三连棋
date: 2018-10-15 17:03:02
tags: 手把手教你玩eos
---

# 手把手教你玩eos 
> 我是此系列教程作者，eoswing团队肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你学eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第五篇,主要是如何使用eos智能合约编写一个游戏：三连棋。


## 0.2 学习内容
1. 相关准备知识
2. 编写智能合约
3. 编译和运行智能合约

## 0.3 机器环境

- cpu: 1核
- 内存: 2G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。


**提示：以下命令行默认在root权限下执行。如遇权限问题，请在命令前加sudo。**

# 1 相关准备知识
## 1.1 三连棋游戏规则

三连棋，又称“井字棋”，是棋类的一种，棋盘为九宫格，呈“井”字形，玩家双方各代表○或×，在棋盘上任意一方连成三个（横竖斜均可）就胜利了。

{% asset_img eost05-01.png 三连棋游戏规则 %}

## 1.2 Multi-Index

Multi-Index是eosio上的数据库管理接口，通过eosio::multi_index智能合约能够写入、读取和修改eosio数据库的数据。 
 
一个完整的multi_index表定义如下：

{% codeblock lang:C %}
	struct limit_order {
		uint64_t     id;
		uint128_t    price;
		uint64_t     expiration;
		account_name owner;

		auto primary_key() const { return id; }
		uint64_t get_expiration() const { return expiration; }
		uint128_t get_price() const { return price; }

		EOSLIB_SERIALIZE( limit_order, ( id )( price )( expiration )( owner ) )
	};
{% endcodeblock %}

## 1.3 建立项目框架

使用eosiocpp建立项目框架：

{% codeblock lang:Bash %}
	cd /contracts
	eosiocpp -n tictactoe
	cd tictactoe
	ll
{% endcodeblock %}

命令行输出如下：

{% asset_img eost05-03.png 创建目录框架 %}

# 2 编写智能合约

## 2.1 编写hpp文件

引入标准库，编写基本结构

{% codeblock lang:C %}
	#include <eosiolib/eosio.hpp>

	class tictactoe : public eosio::contract {
	
		public:
      		tictactoe( account_name self ):contract(self){}
		
		//...游戏数据定义...

		//...游戏命令定义...
		
	};
{% endcodeblock %}
	
游戏数据定义，参看1.2 Multi-Index。

{% codeblock lang:C %}
	struct game {
         static const uint16_t board_width = 3;
         static const uint16_t board_height = 3;
         game() { 
            initialize_board(); 
         }
         account_name   challenger; //挑战者
         account_name   host;       //主角
         account_name   turn; // 当前角色 host/challenger
         account_name   winner = N(none); // none/draw/name of host/name of challenger
         std::vector    board;

         // 初始化棋盘
         void initialize_board() {
            board = std::vector(board_width * board_height, 0);
         }

         // 初始化游戏
         void reset_game() {
            initialize_board();
            turn = host;
            winner = N(none);
         }

         auto primary_key() const { return challenger; }
         EOSLIB_SERIALIZE( game, (challenger)(host)(turn)(winner)(board))
      };

	typedef eosio::multi_index< N(games), game> games;
{% endcodeblock %}

游戏命令定义

{% codeblock lang:C %}	
	/// @abi action
    /// 创建新游戏
    void create(const account_name& challenger, const account_name& host);

    /// @abi action
    /// 重新开始游戏
    void restart(const account_name& challenger, const account_name& host, const account_name& by);

    /// @abi action
    /// 结束游戏
    void close(const account_name& challenger, const account_name& host);

    /// @abi action
    /// 下棋
    void move(const account_name& challenger, const account_name& host, const account_name& by, const uint16_t& row, const uint16_t& column);
{% endcodeblock %}
    
hpp的完整源文件可以查看官方的github代码库:

[https://github.com/EOSIO/eos/blob/master/contracts/tic_tac_toe/tic_tac_toe.hpp](https://github.com/EOSIO/eos/blob/master/contracts/tic_tac_toe/tic_tac_toe.hpp)

> 本教程为避免代码文件冲突，仅修改tic_tac_toe为tictactoe

## 2.2 编写cpp文件

编写基本结构

{% codeblock lang:C %}		
	#include "tictactoe.hpp"

	using namespace eosio;
{% endcodeblock %}

创建新游戏：

{% codeblock lang:C %}		
	void tictactoe::create(const account_name& challenger, const account_name& host) {
   		
		//验证主角签名
		require_auth(host);

		//验证主角和挑战者不是同一人
		eosio_assert(challenger != host, "challenger shouldn't be the same as host");

		//验证主角和挑战者当前没有游戏
		games existing_host_games(_self, host);
		auto itr = existing_host_games.find( challenger );
		eosio_assert(itr == existing_host_games.end(), "game already exists");
		
		//初始化游戏
		existing_host_games.emplace(host, [&]( auto& g ) {
      		g.challenger = challenger;
      		g.host = host;
      		g.turn = host;
		});
	}
{% endcodeblock %}

重新开始游戏：

{% codeblock lang:C %}	
	void tictactoe::restart(const account_name& challenger, const account_name& host, const account_name& by) {

		//验证请求者签名
		require_auth(by);

		//验证这场游戏是否存在
	   	games existing_host_games(_self, host);
		auto itr = existing_host_games.find( challenger );
		eosio_assert(itr != existing_host_games.end(), "game doesn't exists");

		//验证请求者是不是主角或者挑战者
		eosio_assert(by == itr->host || by == itr->challenger, "this is not your game!");

		//重新开始游戏
		existing_host_games.modify(itr, itr->host, []( auto& g ) {
      		g.reset_game();
		});
	}
{% endcodeblock %}

结束游戏：

{% codeblock lang:C %}
	void tictactoe::close(const account_name& challenger, const account_name& host) {
	  
		//验证主角签名
		require_auth(host);
	
	   //验证这场游戏是否存在
	   games existing_host_games(_self, host);
	   auto itr = existing_host_games.find( challenger );
	   eosio_assert(itr != existing_host_games.end(), "game doesn't exists");
	
	   //删除游戏
	   existing_host_games.erase(itr);
	}
{% endcodeblock %}

下棋：

{% codeblock lang:C %}
	void tictactoe::move(const account_name& challenger, const account_name& host, const account_name& by, const uint16_t& row, const uint16_t& column ) {

	   //验证请求者签名
	   require_auth(by);
	
	   //验证这场游戏是否存在
	   games existing_host_games(_self, host);
	   auto itr = existing_host_games.find( challenger );
	   eosio_assert(itr != existing_host_games.end(), "game doesn't exists");
	
	   //验证游戏没有结束
	   eosio_assert(itr->winner == N(none), "the game has ended!");

	   //验证请求者是不是主角或者挑战者
	   eosio_assert(by == itr->host || by == itr->challenger, "this is not your game!");
	   
	   //验证是不是轮到请求者下棋
	   eosio_assert(by == itr->turn, "it's not your turn yet!");
	
	
	   //验证这一步棋子是不是下在空的棋盘格子
	   //is_valid_movement方法提取出来，在后面单独写
	   eosio_assert(is_valid_movement(row, column, itr->board), "not a valid movement!");
	
	   //记录这一步下棋数据
	   const uint8_t cell_value = itr->turn == itr->host ? 1 : 2;
	   const auto turn = itr->turn == itr->host ? itr->challenger : itr->host;
	   existing_host_games.modify(itr, itr->host, [&]( auto& g ) {
	      g.board[row * tictactoe::game::board_width + column] = cell_value;
	      g.turn = turn;
		  //验证这一步棋，是不是赢了比赛
		  //赢家验证get_winner方法提取出来，在后面单独写
	      g.winner = get_winner(g);
	   });
	}	
{% endcodeblock %}

下棋步数有效验证方法is_valid_movement

{% codeblock lang:C %}
	bool is_empty_cell(const uint8_t& cell) {
	   return cell == 0;
	}

	bool is_valid_movement(const uint16_t& row, const uint16_t& column, const vector<uint8_t>& board) {
	   uint32_t movement_location = row * tictactoe::game::board_width + column;
	   bool is_valid = movement_location < board.size() && is_empty_cell(board[movement_location]);
	   return is_valid;
	}
{% endcodeblock %}

赢家验证  
*游戏规则：在横向、纵向和对角线任意一个方向上连线三点就赢得比赛。*

{% codeblock lang:C %}
	account_name get_winner(const tictactoe::game& current_game) {
	   auto& board = current_game.board;
	
	   bool is_board_full = true;
	
	   //采用&操作符来计算横向、纵向和对角线上的连线值。
	   //3 == 0b11, 2 == 0b10, 1 = 0b01, 0 = 0b00
	   vector consecutive_column(tictactoe::game::board_width, 3 );
	   vector consecutive_row(tictactoe::game::board_height, 3 );
	   uint32_t consecutive_diagonal_backslash = 3;
	   uint32_t consecutive_diagonal_slash = 3;
	   for (uint32_t i = 0; i < board.size(); i++) {
	      is_board_full &= is_empty_cell(board[i]);
	      uint16_t row = uint16_t(i / tictactoe::game::board_width);
	      uint16_t column = uint16_t(i % tictactoe::game::board_width);
	
	      //计算横向和纵向的连线值
	      consecutive_row[column] = consecutive_row[column] & board[i]; 
	      consecutive_column[row] = consecutive_column[row] & board[i];

	      //计算对角线的连线值
	      if (row == column) {
	         consecutive_diagonal_backslash = consecutive_diagonal_backslash & board[i];
	      }	     
	      if ( row + column == tictactoe::game::board_width - 1) {
	         consecutive_diagonal_slash = consecutive_diagonal_slash & board[i]; 
	      }
	   }
	
	   //检查所有横向、纵向和对角线的连线值并决定赢家
	   vector aggregate = { consecutive_diagonal_backslash, consecutive_diagonal_slash };
	   aggregate.insert(aggregate.end(), consecutive_column.begin(), consecutive_column.end());
	   aggregate.insert(aggregate.end(), consecutive_row.begin(), consecutive_row.end());
	   for (auto value: aggregate) {
	      if (value == 1) {
	         return current_game.host;
	      } else if (value == 2) {
	         return current_game.challenger;
	      }
	   }

	   // 检查是否已有赢家，有就返回
	   return is_board_full ? N(draw) : N(none);
	}
{% endcodeblock %}
   
cpp的完整源文件可以查看官方的github代码库:

[https://github.com/EOSIO/eos/blob/master/contracts/tic_tac_toe/tic_tac_toe.cpp](https://github.com/EOSIO/eos/blob/master/contracts/tic_tac_toe/tic_tac_toe.cpp)

> 本教程为避免代码文件冲突，仅修改tic_tac_toe为tictactoe

# 3 编译和运行智能合约

## 3.1 编译合约

{% codeblock lang:Bash %}	
	eosiocpp -o tictactoe.wast tictactoe.cpp
	eosiocpp -g tictactoe.abi tictactoe.cpp
	ll
{% endcodeblock %}

命令行输出如下：

{% asset_img eost05-06.png 编译合约 %}

## 3.2 创建合约账户tttaccount

解锁xiao钱包，创建公钥-私钥对，导入公钥-私钥对到钱包，最后，用公钥创建tttaccount。  
具体步骤不再赘述，不熟悉的可以翻看以前的教程。 
 
tttaccount创建后命令行输出如下:

{% asset_img eost05-02.png 创建合约账户tttaccount %}

## 3.3 上传合同到账户

{% codeblock lang:Bash %}	
	cleos set contract tttaccount /contracts/tictactoe -p tttaccount@active
{% endcodeblock %}

命令行输出如下：

{% asset_img eost05-07.png 上传合同到账户 %}

## 3.4 运行智能合约

创建游戏

{% codeblock lang:Bash %}
	cleos push action tttaccount create '{"challenger":"xiaoaccount", "host":"tttaccount"}' -p tttaccount@active
{% endcodeblock %}

命令行输出如下：

{% asset_img eost05-08.png 创建游戏 %}


下棋

{% codeblock lang:Bash %}
	cleos push action tttaccount move '{"challenger":"xiaoaccount", "host":"tttaccount", "by":"tttaccount", "row":0, "column":0}' --permission tttaccount@active
	
	cleos push action tttaccount move '{"challenger":"xiaoaccount", "host":"tttaccount", "by":"xiaoaccount", "row":1, "column":1}' --permission xiaoaccount@active
{% endcodeblock %}

命令行输出如下：

{% asset_img eost05-09.png 下棋 %}

查看游戏状态数据：

{% codeblock lang:Bash %}
	cleos get table tttaccount tttaccount games
{% endcodeblock %}

命令行输出如下：

{% asset_img eost05-12.png 查看游戏状态数据 %}

重新开始游戏

{% codeblock lang:Bash %}
	cleos push action tttaccount restart '{"challenger":"xiaoaccount", "host":"tttaccount", "by":"tttaccount"}' --permission tttaccount@active
{% endcodeblock %}

命令行输出如下：

{% asset_img eost05-10.png 重新开始游戏 %}

结束游戏

{% codeblock lang:Bash %}
	cleos push action tttaccount close '{"challenger":"xiaoaccount", "host":"tttaccount"}' --permission tttaccount@active
{% endcodeblock %}

命令行输出如下：

{% asset_img eost05-11.png 结束游戏 %}

# 5 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- 三连棋教程: https://developers.eos.io/eosio-cpp/docs/tic-tac-toe-tutorial

# 下一篇：<a href="https://blog.eoswing.io/2018/10/23/eos-tutorial-06/" target="_blank">（六）架设EOS区块浏览器</a>

