---
layout: '[layout]'
title: （十一）卡牌游戏第五课：AI部分
date: 2018-11-26 21:10:02
tags: 手把手教你玩eos
---

# 手把手教你玩eos 
> 我是此系列教程作者，eoswing团队肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第十一篇。本篇教程主要学习卡牌游戏的AI对手策略编写。


## 0.2 学习内容
1. 相关准备
2. 智能合约代码编写和部署
3. 测试代码

## 0.3 机器环境

- cpu: 1核
- 内存: 8G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。


# 1 相关准备
## 1.1 课程目标

在本课中，我们将为对手AI设计四种策略。

在每轮出牌时，AI随机选择其中一种策略并决定要出哪张牌。

在本课程结束时，我们将有一个可对战的AI!

## 1.2 AI的四种策略

AI的四种策略分别是：

- 策略1：最大程度争取获胜。 AI Best Card Wins

- 策略2：最大程度防止失败。 AI Minimize Losses

- 策略3：最大程度伤害对手。 AI Points Tally

- 策略4：最大程度减少自己伤害。 AI Loss Prevention

### 策略1解读

AI Best Card Win 最大程度争取获胜。 强调选择一张最有可能获胜的牌。

为此，该策略采取以下加权方式：

- 当AI的伤害值大于玩家的伤害值的时候，记3分
- 当AI的伤害值等于玩家的伤害值的时候，记-1分
- 当AI的伤害值小于玩家的伤害值的时候，记-2分

具体算法如下图所示：

{% asset_img eost11-01.png 策略1解读 %}


### 策略2解读

AI Minimize Losses 最大程度防止失败。 强调选择失败率最低的卡。

为此，该策略采取以下加权方式：

- 当AI的伤害值大于玩家的伤害值的时候，记1分
- 当AI的伤害值等于玩家的伤害值的时候，记-1分
- 当AI的伤害值小于玩家的伤害值的时候，记-4分

具体算法如下图所示：

{% asset_img eost11-02.png 策略2解读 %}

### 策略3解读

AI Points Tally 最大程度伤害对手。 强调选择造成最大伤害的牌。

为此，该策略采取以下加权方式：

（玩家卡牌伤害值 + 元素兼容性）- （AI卡牌伤害值 + 元素兼容性）。

具体算法如下图所示：

{% asset_img eost11-03.png 策略3解读 %}

### 策略4解读

AI Loss Prevention 最大程度减少自己伤害。 强调确保最大限度的从该游戏中生存下来。

当AI剩余大量HP时，此策略不适用。只有当AI剩余小于或等于2 HP时才会选择此策略。

为此，该策略采取以下加权方式：

- 不会输掉游戏的牌，记为1。

- 会导致输掉游戏的牌，记为0。

具体算法如下图所示：

{% asset_img eost11-04.png 策略4解读 %}

## 1.3 准备工作

### 进入开发环境容器

{% codeblock lang:Bash %}
	docker exec -it eosdev /bin/bash
{% endcodeblock %}	

### 进入后端智能合约文件夹

{% codeblock lang:Bash %}
	cd /eos-work/contracts/cardgame
{% endcodeblock %}

# 2 智能合约代码编写和部署

## 2.1 编写合约代码

打开cardgame.hpp文件：

{% codeblock lang:Bash %}
	vi cardgame.hpp
{% endcodeblock %}

编辑代码，添加代码如下：

{% codeblock lang:C %}	
	void draw_one_card(vector<uint8_t>& deck, vector<uint8_t>& hand);
	//在上面代码行后添加如下代码
	//===下面为添加代码===

	int calculate_attack_point(const card& card1, const card& card2);

    int ai_best_card_win_strategy(const int ai_attack_point, const int player_attack_point);

    int ai_min_loss_strategy(const int ai_attack_point, const int player_attack_point);

    int ai_points_tally_strategy(const int ai_attack_point, const int player_attack_point);

    int ai_loss_prevention_strategy(const int8_t life_ai, const int ai_attack_point, const int player_attack_point);

    int calculate_ai_card_score(const int strategy_idx, const int8_t life_ai,
                                const card& ai_card, const vector<uint8_t> hand_player);

    int ai_choose_card(const game& game_data);

	//======
{% endcodeblock %}

打开gameplay.cpp文件：

{% codeblock lang:Bash %}
	vi gameplay.cpp
{% endcodeblock %}

编辑代码，在代码行的最后面添加代码如下：

{% codeblock lang:C %}	
	//在代码行的最后添加如下代码

	// Calculate the final attack point of a card after taking the elemental bonus into account
	int cardgame::calculate_attack_point(const card& card1, const card& card2) {
	  int result = card1.attack_point;
	
	  return result;
	}
	
	// AI Best Card Win Strategy
	int cardgame::ai_best_card_win_strategy(const int ai_attack_point, const int player_attack_point) {
	  eosio::print("Best Card Wins");
	  if (ai_attack_point > player_attack_point) return 3;
	  if (ai_attack_point < player_attack_point) return -2;
	  return -1;
	}
	
	// AI Minimize Loss Strategy
	int cardgame::ai_min_loss_strategy(const int ai_attack_point, const int player_attack_point) {
	  eosio::print("Minimum Losses");
	  if (ai_attack_point > player_attack_point) return 1;
	  if (ai_attack_point < player_attack_point) return -4;
	  return -1;
	}
	
	// AI Points Tally Strategy
	int cardgame::ai_points_tally_strategy(const int ai_attack_point, const int player_attack_point) {
	  eosio::print("Points Tally");
	  return ai_attack_point - player_attack_point;
	}
	
	// AI Loss Prevention Strategy
	int cardgame::ai_loss_prevention_strategy(const int8_t life_ai, const int ai_attack_point, const int player_attack_point) {
	  eosio::print("Loss Prevention");
	  if (life_ai + ai_attack_point - player_attack_point > 0) return 1;
	  return 0;
	}
	
	// Calculate the score for the current ai card given the  strategy and the player hand cards
	int cardgame::calculate_ai_card_score(const int strategy_idx,
	                                      const int8_t life_ai,
	                                      const card& ai_card,
	                                      const vector<uint8_t> hand_player) {
	   int card_score = 0;
	   for (int i = 0; i < hand_player.size(); i++) {
	      const auto player_card_id = hand_player[i];
	      const auto player_card = card_dict.at(player_card_id);
	
	      int ai_attack_point = calculate_attack_point(ai_card, player_card);
	      int player_attack_point = calculate_attack_point(player_card, ai_card);
	
	      // Accumulate the card score based on the given strategy
	      switch (strategy_idx) {
	        case 0: {
	          card_score += ai_best_card_win_strategy(ai_attack_point, player_attack_point);
	          break;
	        }
	        case 1: {
	          card_score += ai_min_loss_strategy(ai_attack_point, player_attack_point);
	          break;
	        }
	        case 2: {
	          card_score += ai_points_tally_strategy(ai_attack_point, player_attack_point);
	          break;
	        }
	        default: {
	          card_score += ai_loss_prevention_strategy(life_ai, ai_attack_point, player_attack_point);
	          break;
	        }
	      }
	    }
	    return card_score;
	}
	
	// Chose a card from the AI's hand based on the current game data
	int cardgame::ai_choose_card(const game& game_data) {
	  // The 4th strategy is only chosen in the dire situation
	  int available_strategies = 4;
	  if (game_data.life_ai > 2) available_strategies--;
	  int strategy_idx = random(available_strategies);
	
	  // Calculate the score of each card in the AI hand
	  int chosen_card_idx = -1;
	  int chosen_card_score = std::numeric_limits<int>::min();
	
	  for (int i = 0; i < game_data.hand_ai.size(); i++) {
	    const auto ai_card_id = game_data.hand_ai[i];
	    const auto ai_card = card_dict.at(ai_card_id);
	
	    // Ignore empty slot in the hand
	    if (ai_card.type == EMPTY) continue;
	
	    // Calculate the score for this AI card relative to the player's hand cards
	    auto card_score = calculate_ai_card_score(strategy_idx, game_data.life_ai, ai_card, game_data.hand_player);
	
	    // Keep track of the card that has the highest score
	    if (card_score > chosen_card_score) {
	      chosen_card_score = card_score;
	      chosen_card_idx = i;
	    }
	  }
	  return chosen_card_idx;
	}
{% endcodeblock %}

打开cardgame.cpp文件：

{% codeblock lang:Bash %}
	vi cardgame.cpp
{% endcodeblock %}

编辑代码，添加代码如下：

{% codeblock lang:C %}	
	game_data.hand_player[player_card_idx] = 0;
	//在上面代码行后添加如下代码
	//===下面为添加代码===

	// AI picks a card
    int ai_card_idx = ai_choose_card(game_data);
    game_data.selected_card_ai = game_data.hand_ai[ai_card_idx];
    game_data.hand_ai[ai_card_idx] = 0;
{% endcodeblock %}

## 2.2 部署智能合约覆盖原合约

### 编译智能合约

#### 编译wast文件

{% codeblock lang:Bash %}
	eosiocpp -o cardgame.wast cardgame.cpp
{% endcodeblock %}	

#### 编译abi文件

{% codeblock lang:Bash %}
	eosiocpp -g cardgame.abi cardgame.cpp
{% endcodeblock %}

### 解锁钱包

{% codeblock lang:Bash %}
	cleos wallet unlock -n gamewallet
{% endcodeblock %}

### 部署智能合约

{% codeblock lang:Bash %}
	cleos -u https://api-kylin.eosasia.one set contract 123123gogogo /eos-work/contracts/cardgame -p 123123gogogo@active
{% endcodeblock %}

至此，后端的合约重新部署完成。

# 3 测试代码

{% codeblock lang:Bash %}
	cd /eos-work/frontend

	npm start
{% endcodeblock %}

在浏览器中输入网址测试。

因为现在游戏并不完整，测试后遗留数据不方便清零。

所以每次阶段性查看下效果，最好新建一个账户测试。

此次测试，新建了一个账户cardgame2334。


登录游戏。

{% asset_img eost11-05.png 登录游戏 %}	  


进入游戏。

{% asset_img eost11-06.png 进入游戏 %}


开始游戏。

{% asset_img eost11-07.png 开始游戏 %}	


出牌，同时AI出牌。

{% asset_img eost11-08.png 出牌 %}

# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- EOS官方游戏开发第五课: https://battles.eos.io/tutorial/lesson5/chapter1	

	