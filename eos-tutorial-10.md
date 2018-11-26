
（十）卡牌游戏第四课：游戏核心组件
===================================

# 手把手教你玩eos 
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第十篇。本篇教程主要学习卡牌游戏的核心代码编写。


## 0.2 学习内容
1. 相关准备
2. 智能合约代码编写和部署
3. 编写前端代码
4. 测试代码

## 0.3 机器环境

- cpu: 1核
- 内存: 8G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。


# 1 相关准备
## 1.1 课程目标

在本篇教程里，将介绍一些关键组件以及游戏的玩法。
我们在智能合同里定义的数据结构，包含用于显示前端信息的游戏状态。同时增加键屏幕组件GameInfo，GameMap，Handcards和PlayerInfo。
最后将这些前端组件拼接起来，以便玩家可以通过从UI调用智能合约操作来启动游戏并玩牌。

还介绍了一种简单但有效的随机化技术。

整个流程如下图所示：

![](/images/eost10-00.png "流程图")

## 1.2 游戏规则

两位玩家在开始时都以5HP开始。一旦某一玩家的HP下降到0，游戏结束。

每把游戏都有三个状态，即：

- ONGOING--游戏正在进行中。

- PLAYER_WON--游戏已经结束且玩家获胜。

- PLAYER_LOST--游戏已经结束且玩家失败。

游戏中的卡牌：

- Elemental Battles中有11张独特的牌

- 每张卡属于一种元素类型

- 每张卡都分配了攻击力

- 每个玩家都以相同的17张牌开始

- 一些元素类型具有元素兼容性

五种元素类型分别是：

![](/images/eost10-01.png "五种元素类型")


卡片类型是元素类型和攻击力的组合。让我们看一下牌组中的所有牌：

![](/images/eost10-02.png "牌组中的所有牌")


## 1.3 准备工作

### 进入开发环境容器

```Bash
	docker exec -it eosdev /bin/bash
```

### 进入后端智能合约文件夹

```Bash
	cd /eos-work/contracts/cardgame
```

# 2 智能合约代码编写和部署

## 2.1 编写合约代码

打开cardgame.hpp文件：

```Bash
	vi cardgame.hpp
```

编辑代码，最终代码如下：

```C	
	#include <eosiolib/eosio.hpp>

	using namespace std;
	class cardgame : public eosio::contract {
	
	  private:
	
	    enum game_status: int8_t  {
	      ONGOING     = 0,
	      PLAYER_WON   = 1,
	      PLAYER_LOST  = -1
	    };
	
	    enum card_type: uint8_t {
	      EMPTY = 0, // Represents empty slot in hand
	      FIRE = 1,
	      WOOD = 2,
	      WATER = 3,
	      NEUTRAL = 4,
	      VOID = 5
	    };
	
	    struct card {
	      uint8_t type;
	      uint8_t attack_point;
	    };
	
	    typedef uint8_t card_id;
	
	    const map<card_id, card> card_dict = {
	      { 0, {EMPTY, 0} },
	      { 1, {FIRE, 1} },
	      { 2, {FIRE, 1} },
	      { 3, {FIRE, 2} },
	      { 4, {FIRE, 2} },
	      { 5, {FIRE, 3} },
	      { 6, {WOOD, 1} },
	      { 7, {WOOD, 1} },
	      { 8, {WOOD, 2} },
	      { 9, {WOOD, 2} },
	      { 10, {WOOD, 3} },
	      { 11, {WATER, 1} },
	      { 12, {WATER, 1} },
	      { 13, {WATER, 2} },
	      { 14, {WATER, 2} },
	      { 15, {WATER, 3} },
	      { 16, {NEUTRAL, 3} },
	      { 17, {VOID, 0} }
	    };
	
	    struct game {
	      int8_t          life_player = 5;
	      int8_t          life_ai = 5;
	      vector<card_id> deck_player = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17};
	      vector<card_id> deck_ai = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17};
	      vector<card_id> hand_player = {0, 0, 0, 0};
	      vector<card_id> hand_ai = {0, 0, 0, 0};
	      card_id         selected_card_player = 0;
	      card_id         selected_card_ai = 0;
	      uint8_t         life_lost_player = 0;
	      uint8_t         life_lost_ai = 0;
	      int8_t          status = ONGOING;
	    };
	
	    // @abi table users
	    struct user_info {
	      account_name    name;
	      uint16_t        win_count = 0;
	      uint16_t        lost_count = 0;
	      game            game_data;
	
	      auto primary_key() const { return name; }
	    };
	
	    // @abi table seed
	    struct seed {
	      uint64_t        key = 1;
	      uint32_t        value = 1;
	
	      auto primary_key() const { return key; }
	    };
	
	    typedef eosio::multi_index<N(users), user_info> users_table;
	
	    typedef eosio::multi_index<N(seed), seed> seed_table;
	
	    users_table _users;
	
	    seed_table _seed;
	
	    void draw_one_card(vector<uint8_t>& deck, vector<uint8_t>& hand);
	
	    int random(const int range);
	
	  public:
	
	    cardgame( account_name self ):contract(self),_users(self, self),_seed(self, self){}
	
	    void login(account_name username);
	
	    void startgame(account_name username);
	
	    void playcard(account_name username, uint8_t player_card_idx);
	
	};
```

打开gameplay.cpp文件：

```Bash
	vi gameplay.cpp
```

编辑代码，最终代码如下：

```C	
	#include "cardgame.hpp"
	
	// Simple Pseudo Random Number Algorithm, randomly pick a number within 0 to n-1
	int cardgame::random(const int range) {
	  // Find the existing seed
	  auto seed_iterator = _seed.begin();
	
	  // Initialize the seed with default value if it is not found
	  if (seed_iterator == _seed.end()) {
	    seed_iterator = _seed.emplace( _self, [&]( auto& seed ) { });
	  }
	
	  // Generate new seed value using the existing seed value
	  int prime = 65537;
	  auto new_seed_value = (seed_iterator->value + now()) % prime;
	
	  // Store the updated seed value in the table
	  _seed.modify( seed_iterator, _self, [&]( auto& s ) {
	    s.value = new_seed_value;
	  });
	
	  // Get the random result in desired range
	  int random_result = new_seed_value % range;
	  return random_result;
	}
	
	// Draw one card from the deck and assign it to the hand
	void cardgame::draw_one_card(vector<uint8_t>& deck, vector<uint8_t>& hand) {
	  // Pick a random card from the deck
	  int deck_card_idx = random(deck.size());
	
	  // Find the first empty slot in the hand
	  int first_empty_slot = -1;
	  for (int i = 0; i <= hand.size(); i++) {
	    auto id = hand[i];
	    if (card_dict.at(id).type == EMPTY) {
	      first_empty_slot = i;
	      break;
	    }
	  }
	  eosio_assert(first_empty_slot != -1, "No empty slot in the player's hand");
	
	  // Assign the card to the first empty slot in the hand
	  hand[first_empty_slot] = deck[deck_card_idx];
	
	  // Remove the card from the deck
	  deck.erase(deck.begin() + deck_card_idx);
	}
```

打开cardgame.cpp文件：

```Bash
	vi cardgame.cpp
```

编辑代码，最终代码如下：

```C	
	#include "gameplay.cpp"

	void cardgame::login(account_name username) {
	  // Ensure this action is authorized by the player
	  require_auth(username);
	
	  // Create a record in the table if the player doesn't exist in our app yet
	  auto user_iterator = _users.find(username);
	  if (user_iterator == _users.end()) {
	    user_iterator = _users.emplace(username,  [&](auto& new_user) {
	      new_user.name = username;
	    });
	  }
	}
	
	void cardgame::startgame(account_name username) {
	  // Ensure this action is authorized by the player
	  require_auth(username);
	
	  auto& user = _users.get(username, "User doesn't exist");
	
	  _users.modify(user, username, [&](auto& modified_user) {
	    // Create a new game
	    game game_data;
	
	    // Draw 4 cards each for the player and the AI
	    for (uint8_t i = 0; i < 4; i++) {
	      draw_one_card(game_data.deck_player, game_data.hand_player);
	      draw_one_card(game_data.deck_ai, game_data.hand_ai);
	    }
	
	    // Assign the newly created game to the player
	    modified_user.game_data = game_data;
	  });
	}
	
	void cardgame::playcard(account_name username, uint8_t player_card_idx) {
	  // Ensure this action is authorized by the player
	  require_auth(username);
	
	  // Checks that selected card is valid
	  eosio_assert(player_card_idx < 4, "playcard: Invalid hand index");
	
	  auto& user = _users.get(username, "User doesn't exist");
	
	  // Verify game status is suitable for the player to play a card
	  eosio_assert(user.game_data.status == ONGOING,
	               "playcard: This game has ended. Please start a new one");
	  eosio_assert(user.game_data.selected_card_player == 0,
	               "playcard: The player has played his card this turn!");
	
	  _users.modify(user, username, [&](auto& modified_user) {
	    game& game_data = modified_user.game_data;
	
	    // Assign the selected card from the player's hand
	    game_data.selected_card_player = game_data.hand_player[player_card_idx];
	    game_data.hand_player[player_card_idx] = 0;
	  });
	}
	
	EOSIO_ABI(cardgame, (login)(startgame)(playcard))
```

## 2.2 部署智能合约覆盖原合约

### 编译智能合约

#### 编译wast文件

```Bash
	eosiocpp -o cardgame.wast cardgame.cpp
```

#### 编译abi文件

```Bash
	eosiocpp -g cardgame.abi cardgame.cpp
```

### 解锁钱包

```Bash
	cleos wallet unlock -n gamewallet
```

命令行显示如下：

![](/images/eost10-03.png "解锁钱包")

### 部署智能合约

```Bash
	cleos -u https://api-kylin.eosasia.one set contract 123123gogogo /eos-work/contracts/cardgame -p 123123gogogo@active
```

命令行输出如下：

![](/images/eost10-04.png "部署智能合约")

至此，后端的合约重新部署完成。

# 3 编写前端代码

## 3.1 下载Game中五个组件代码

由于时间和篇幅关系，在这里只是下载代码，就不把代码贴出来一一解读了。
建议读者最好挨个下载，然后逐一查看代码逻辑。

```Bash
	cd /eos-work/frontend/src/components/Game/components/	

	svn checkout https://github.com/EOSIO/eosio-card-game-repo/branches/lesson-4/frontend/src/components/Game/components/Card

	svn checkout https://github.com/EOSIO/eosio-card-game-repo/branches/lesson-4/frontend/src/components/Game/components/GameInfo

	svn checkout https://github.com/EOSIO/eosio-card-game-repo/branches/lesson-4/frontend/src/components/Game/components/GameMat

	svn checkout https://github.com/EOSIO/eosio-card-game-repo/branches/lesson-4/frontend/src/components/Game/components/HandCards

	svn checkout https://github.com/EOSIO/eosio-card-game-repo/branches/lesson-4/frontend/src/components/Game/components/PlayerInfo
```

## 3.2 整合组件

```Bash
	vi index.js
```

更新代码如下：

```C		
	import Card from './Card';
	import GameInfo from './GameInfo';
	import GameMat from './GameMat';
	import HandCards from './HandCards';
	import PlayerInfo from './PlayerInfo';
	import PlayerProfile from './PlayerProfile';
	
	export {
	  Card,
	  GameInfo,
	  GameMat,
	  HandCards,
	  PlayerInfo,
	  PlayerProfile,
	}
```

返回上一级，编辑Game.jsx

```Bash
	cd ..

	vi Game.jsx
```

更新代码如下:

```C	
	// React core
	import React, { Component } from 'react';
	import { connect } from 'react-redux';
	// Game subcomponents
	import { GameInfo, GameMat, PlayerProfile } from './components';
	// Services and redux action
	import { UserAction } from 'actions';
	import { ApiService } from 'services';
	
	class Game extends Component {
	
	  constructor(props) {
	    // Inherit constructor
	    super(props);
	    // Bind functions
	    this.loadUser = this.loadUser.bind(this);
	    this.handleStartGame = this.handleStartGame.bind(this);
	    this.handlePlayCard = this.handlePlayCard.bind(this);
	    // Call `loadUser` before mounting the app
	    this.loadUser();
	  }
	
	  // Get latest user object from blockchain
	  loadUser() {
	    // Extract `setUser` of `UserAction` and `user.name` of UserReducer from redux
	    const { setUser, user: { name } } = this.props;
	    // Send request the blockchain by calling the ApiService,
	    // Get the user object and store the `win_count`, `lost_count` and `game_data` object
	    return ApiService.getUserByName(name).then(user => {
	      setUser({
	        win_count: user.win_count,
	        lost_count: user.lost_count,
	        game: user.game_data,
	      });
	    });
	  }
	
	  handleStartGame() {
	    // Send a request to API (blockchain) to start game
	    // And call `loadUser` again for react to render latest game status to UI
	    return ApiService.startGame().then(()=>{
	      return this.loadUser();
	    });
	  }
	
	  handlePlayCard(cardIdx) {
	    // Extract `user.game` of `UserReducer` from redux
	    const { user: { game } } = this.props;
	
	    // If it is an empty card, not going to do anything
	    if (game.hand_player[cardIdx] === 0) {
	      return;
	    }
	
	    // Send a request to API (blockchain) to play card with card index
	    // And call `loadUser` again for react to render latest game status to UI
	    return ApiService.playCard(cardIdx).then(()=>{
	      return this.loadUser();
	    });
	  }
	
	  render() {
	    // Extract data from user data of `UserReducer` from redux
	    const { user: { name, win_count, lost_count, game } } = this.props;
	
	    // Flag to indicate if the game has started or not
	    // By checking if the deckCard of AI is still 17 (max card)
	    const isGameStarted = game && game.deck_ai.length !== 17;
	
	    // If game hasn't started, display `PlayerProfile`
	    // If game has started, display `GameMat`, `Info` screen
	    return (
	      <section className="Game">
	        { !isGameStarted ?
	            <PlayerProfile
	              name={ name }
	              winCount={ win_count }
	              lostCount={ lost_count }
	              onStartGame={ this.handleStartGame }
	            />
	          :
	            <div className="container">
	              <GameMat
	                deckCardCount={ game.deck_ai.length }
	                aiLife={ game.life_ai }
	                aiHandCards={ game.hand_ai }
	                aiName="COMPUTER"
	                playerLife={ game.life_player }
	                playerHandCards={ game.hand_player }
	                playerName={ name }
	                onPlayCard={ this.handlePlayCard }
	              />
	              <GameInfo
	                deckCardCount={ game.deck_ai.length }
	                handCardCount={ game.hand_ai.filter( x => x > 0 ).length }
	              />
	            </div>
	        }
	      </section>
	    )
	  }
	
	}
	
	// Map all state to component props (for redux to connect)
	const mapStateToProps = state => state;
	
	// Map the following action to props
	const mapDispatchToProps = {
	  setUser: UserAction.setUser,
	};
	
	// Export a redux connected component
	export default connect(mapStateToProps, mapDispatchToProps)(Game);
```

## 3.3 编辑与区块链交互的ApiService.js

```Bash
	cd /eos-work/frontend/src/services/
     
	vi ApiService.js
```

修改代码如下:

```C	
	import { Api, Rpc, SignatureProvider } from 'eosjs';

	// Main action call to blockchain
	async function takeAction(action, dataValue) {
	  const privateKey = localStorage.getItem("cardgame_key");
	  const rpc = new Rpc.JsonRpc(process.env.REACT_APP_EOS_HTTP_ENDPOINT);
	  const signatureProvider = new SignatureProvider([privateKey]);
	  const api = new Api({ rpc, signatureProvider, textDecoder: new TextDecoder(), textEncoder: new TextEncoder() });
	
	  // Main call to blockchain after setting action, account_name and data
	  try {
	    const resultWithConfig = await api.transact({
	      actions: [{
	        account: process.env.REACT_APP_EOS_CONTRACT_NAME,
	        name: action,
	        authorization: [{
	          actor: localStorage.getItem("cardgame_account"),
	          permission: 'active',
	        }],
	        data: dataValue,
	      }]
	    }, {
	      blocksBehind: 3,
	      expireSeconds: 30,
	    });
	    return resultWithConfig;
	  } catch (err) {
	    throw(err)
	  }
	}
	
	class ApiService {
	
	  static getCurrentUser() {
	    return new Promise((resolve, reject) => {
	      if (!localStorage.getItem("cardgame_account")) {
	        return reject();
	      }
	      takeAction("login", { username: localStorage.getItem("cardgame_account") })
	        .then(() => {
	          resolve(localStorage.getItem("cardgame_account"));
	        })
	        .catch(err => {
	          localStorage.removeItem("cardgame_account");
	          localStorage.removeItem("cardgame_key");
	          reject(err);
	        });
	    });
	  }
	
	  static login({ username, key }) {
	    return new Promise((resolve, reject) => {
	      localStorage.setItem("cardgame_account", username);
	      localStorage.setItem("cardgame_key", key);
	      takeAction("login", { username: username })
	        .then(() => {
	          resolve();
	        })
	        .catch(err => {
	          localStorage.removeItem("cardgame_account");
	          localStorage.removeItem("cardgame_key");
	          reject(err);
	        });
	    });
	  }
	
	  static startGame() {
	    return takeAction("startgame", { username: localStorage.getItem("cardgame_account") });
	  }
	
	  static playCard(cardIdx) {
	    return takeAction("playcard", { username: localStorage.getItem("cardgame_account"), player_card_idx: cardIdx });
	  }
	
	  static async getUserByName(username) {
	    try {
	      const rpc = new Rpc.JsonRpc(process.env.REACT_APP_EOS_HTTP_ENDPOINT);
	      const result = await rpc.get_table_rows({
	        "json": true,
	        "code": process.env.REACT_APP_EOS_CONTRACT_NAME,    // contract who owns the table
	        "scope": process.env.REACT_APP_EOS_CONTRACT_NAME,   // scope of the table
	        "table": "users",    // name of the table as specified by the contract abi
	        "limit": 1,
	        "lower_bound": username,
	      });
	      return result.rows[0];
	    } catch (err) {
	      console.error(err);
	    }
	  }
	
	}

	export default ApiService;
```
	
# 4 测试代码

```Bash
	cd /eos-work/frontend

	npm start
```


在浏览器中输入网址测试。

我新建了一个账户cardgame2333，登录游戏。

![](/images/eost10-05.png "登录游戏") 

进入游戏。

![](/images/eost10-06.png "进入游戏")	

# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- EOS官方游戏开发第四课: https://battles.eos.io/tutorial/lesson4/chapter1	

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。

# 下一篇：<a href="https://github.com/eoswing/eos-tutorial/blob/master/eos-tutorial-11.md" target="_blank">（十一）卡牌游戏第五课：AI部分</a>