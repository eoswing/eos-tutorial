（十四）卡牌游戏第八课：优化细节体验
===================================

# 手把手教你玩eos 
> 我是此系列教程作者，<a href="https://www.eoswing.io" >eoswing团队</a>肖南飞,区块链技术开发人员。

# 0.引言
## 0.1教程概况
手把手教你玩eos系列教程，从最基础开始，一步一步教你学会用eos。比如发代币，开发DAPP等等。  
本文是第十四篇。本篇教程是卡牌游戏的最后一课，讲解怎么进一步优化整个游戏的细节体验。


## 0.2 学习内容
1. 相关准备
2. 编写前端代码
3. 测试代码

## 0.3 机器环境

- cpu: 1核
- 内存: 8G
- 操作系统：CentOS 7.4 64位
- 服务器所在地：香港

> 推荐将服务器放在网络较为优质的环境，比如香港。不然会有很多配置依赖下载上的问题。


# 1 相关准备
## 1.1 课程目标

在本课程中，我们将优化游戏细节体验，比如添加游戏帮助对话框，进度提示，优化加载体验等等。


## 1.2 准备工作

### 进入开发环境容器

```Bash
	docker exec -it eosdev /bin/bash
```

### 进入前端文件夹

```Bash
	cd /eos-work/frontend/
```

# 2 编写前端代码

## 2.1 添加游戏帮助对话框

在GameInfo中添加RulesModal模态对话框。

```Bash
	cd ./src/components/Game/components/GameInfo

	svn checkout https://github.com/EOSIO/eosio-card-game-repo/branches/lesson-8/frontend/src/components/Game/components/GameInfo/components
```

在GameInfo.jsx中加入RulesModal组件

```Bash
	vi GameInfo.jsx
```

更新代码如下：

{% codeblock lang:C %}	
	import React, { Component } from 'react';
	// Components
	import { Button } from 'components';

	//在此次添加引用
	// Game subcomponents
	import { RulesModal } from './components';
	
	class Info extends Component {
	  render() {
	    // Extract data and event functions from props
	    const { className, deckCardCount, handCardCount, onEndGame } = this.props;
	    // Display:
	    // Round number: 18 <-- ((max deck = 17) + 1) - Deck Cards - Hand Cards
	    // Rules button to trigger a modal
	    // Button to end the current game
	    return (
	      <div className={`Info${ className ? ' ' + className : '' }`}>
	        { <p>ROUND <span className="round-number">{ 18 - deckCardCount - handCardCount }/17</span></p> }
			//在此处添加组件
	        <RulesModal />
	        <div><Button onClick={ onEndGame } className="small red">QUIT</Button></div>
	      </div>
	    )
	  }
	}
	
	export default Info;
```

## 2.2 添加进度提示

```Bash
	cd /eos-work/frontend/src/components/Button/

	vi Button.jsx
```

编辑代码如下:

{% codeblock lang:C %}	
	import React, { Component } from 'react';

	class Button extends Component {
	
	  constructor(props) {
	    // Inherit constructor
	    super(props);
	    // Component state setup
	    this.state = {
	      loading: false,
	    };
	    // Bind functions
	    this.handleClick = this.handleClick.bind(this);
	  }
	
	  handleClick() {
	    const { onClick } = this.props;
	    // Show the loading indicator in case the action to be performed takes too long
	    this.setState({ loading: true });
	
	    // If the prop onClick is a function, invoke it and stores its return value in ``promise``
	    // If the prop onClick is NOT a function, the value of ``promise`` will be false
	    const promise = typeof onClick === "function" && onClick();
	
	    // If ``promise`` is a function (a Promise), invoke setState after it has been resolved.
	    if (promise && typeof promise.then === "function") {
	      return promise.then(() => {
	        this.isComponentMounted && this.setState({ loading: false });
	      });
	    }
	    // Otherwise, just invoke setState directly
	    this.isComponentMounted && this.setState({ loading: false });
	  }
	
	  componentDidMount() {
	    this.isComponentMounted = true;
	  }
	
	  componentWillUnmount() {
	    this.isComponentMounted = false;
	  }
	
	  render() {
	    const { className, type, style, children } = this.props;
	    let { loading } = this.state;
	    // Enable the loading CSS class if either the private state attribute `loading`
	    // or the props `loading` is true
	    loading = loading || this.props.loading;
	    return (
	      <button
	        className={`Button${ className ? ' ' + className : '' }${ loading ? ' loading' : '' }`}
	        onClick={ this.handleClick }
	        { ...{ type, style } }
	      >{ children }</button>
	    );
	  }
	
	}
	
	export default Button;
```

在Game.jsx中更新引用

```Bash
	cd /eos-work/frontend/src/components/Game/

	vi Game.jsx
```

编辑代码如下:

{% codeblock lang:C %}	
	// React core
	import React, { Component } from 'react';
	import { connect } from 'react-redux';
	// Game subcomponents
	import { GameInfo, GameMat, PlayerProfile, Resolution } from './components';
	// Services and redux action
	import { UserAction } from 'actions';
	import { ApiService } from 'services';
	
	class Game extends Component {
	
	  constructor(props) {
	    // Inherit constructor
	    super(props);
	    // State for showing/hiding components when the API (blockchain) request is loading
	    this.state = {
	      loading: true,
	    };
	    // Bind functions
	    this.loadUser = this.loadUser.bind(this);
	    this.handleStartGame = this.handleStartGame.bind(this);
	    this.handlePlayCard = this.handlePlayCard.bind(this);
	    this.handleNextRound = this.handleNextRound.bind(this);
	    this.handleEndGame = this.handleEndGame.bind(this);
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
	      // Set the loading state to false for displaying the app
	      this.setState({ loading: false });
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
	    // Show the loading indicator if the connection took too long
	    this.setState({ loading: true });
	    // Send a request to API (blockchain) to play card with card index
	    // And call `loadUser` again for react to render latest game status to UI
	    return ApiService.playCard(cardIdx).then(()=>{
	      return this.loadUser();
	    });
	  }
	
	  handleNextRound() {
	    // Send a request to API (blockchain) to trigger next round
	    // And call `loadUser` again for react to render latest game status to UI
	    return ApiService.nextRound().then(()=>{
	      return this.loadUser();
	    });
	  }
	
	  handleEndGame() {
	    // Send a request to API (blockchain) to end the game
	    // And call `loadUser` again for react to render latest game status to UI
	    return ApiService.endGame().then(()=>{
	      return this.loadUser();
	    });
	  }
	
	  render() {
	    // Extract data from state and user data of `UserReducer` from redux
	    const { loading } = this.state;
	    const { user: { name, win_count, lost_count, game } } = this.props;
	
	    // Flag to indicate if the game has started or not
	    // By checking if the deckCard of AI is still 17 (max card)
	    const isGameStarted = game && game.deck_ai.length !== 17;
	
	    // If game hasn't started, display `PlayerProfile`
	    // If game has started, display `GameMat`, `Resolution`, `Info` screen
	    return (
	      <section className={`Game${ (loading ? " loading" : "") }`}>
	        { !isGameStarted ?
	            <PlayerProfile
	              name={ name }
	              winCount={ win_count }
	              lostCount={ lost_count }
	              onLogout={ this.handleLogout }
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
	              <Resolution
	                status={ game.status }
	                aiCard={ game.selected_card_ai }
	                aiName="COMPUTER"
	                aiLost={ game.life_lost_ai }
	                playerCard={ game.selected_card_player }
	                playerName={ name }
	                playerLost={ game.life_lost_player }
	                onNextRound={ this.handleNextRound }
	                onEndGame={ this.handleEndGame }
	              />
	              <GameInfo
	                deckCardCount={ game.deck_ai.length }
	                handCardCount={ game.hand_ai.filter( x => x > 0 ).length }
	                onEndGame={ this.handleEndGame }
	              />
	            </div>
	        }
	        {
	          isGameStarted && loading &&
	          <div className="spinner">
	            <div className="image"></div>
	            <div className="circles">
	              <div className="circle">
	                <div className="inner"></div>
	              </div>
	              <div className="circle">
	                <div className="inner"></div>
	              </div>
	              <div className="circle">
	                <div className="inner"></div>
	              </div>
	              <div className="circle">
	                <div className="inner"></div>
	              </div>
	              <div className="circle">
	                <div className="inner"></div>
	              </div>
	            </div>
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

## 2.3 优化加载体验

现在每次刷新游戏屏幕时都会短暂显示登录屏幕。

这是由ApiService从区块链获取最新用户数据引起的。

在检索用户数据时，dApp认为用户没有登录。

为了防止这种情况，我们将向App.jsx添加一个名为loading的组件状态。

在检索区块链数据时出现，直到有反馈，确定是应用是在登录页面，游戏结束屏幕还是个人资料屏幕等等。

```Bash
	cd /eos-work/frontend/src/components/Login/

	vi Login.jsx 
```

更新代码如下：

{% codeblock lang:C %}	
	import React, { Component } from 'react';
	import { connect } from 'react-redux';
	// Components
	import { Button } from 'components';
	// Services and redux action
	import { UserAction } from 'actions';
	import { ApiService } from 'services';
	
	class Login extends Component {
	
	  constructor(props) {
	    // Inherit constructor
	    super(props);
	    // State for form data and error message
	    this.state = {
	      form: {
	        username: '',
	        key: '',
	        error: '',
	      },
	      isSigningIn: false,
	    }
	    // Bind functions
	    this.handleChange = this.handleChange.bind(this);
	    this.handleSubmit = this.handleSubmit.bind(this);
	  }
	
	  // Runs on every keystroke to update the React state
	  handleChange(event) {
	    const { name, value } = event.target;
	    const { form } = this.state;
	
	    this.setState({
	      form: {
	        ...form,
	        [name]: value,
	        error: '',
	      },
	    });
	  }
	
	  componentDidMount() {
	    this.isComponentMounted = true;
	  }
	
	  componentWillUnmount() {
	    this.isComponentMounted = false;
	  }
	
	  // Handle form submission to call api
	  handleSubmit(event) {
	    // Stop the default form submit browser behaviour
	    event.preventDefault();
	    // Extract `form` state
	    const { form } = this.state;
	    // Extract `setUser` of `UserAction` and `user.name` of UserReducer from redux
	    const { setUser } = this.props;
	    // Set loading spinner to the button
	    this.setState({ isSigningIn: true });
	    // Send a login transaction to the blockchain by calling the ApiService,
	    // If it successes, save the username to redux store
	    // Otherwise, save the error state for displaying the message
	    return ApiService.login(form)
	      .then(() => {
	        setUser({ name: form.username });
	      })
	      .catch(err => {
	        this.setState({ error: err.toString() });
	      })
	      .finally(() => {
	        if (this.isComponentMounted) {
	          this.setState({ isSigningIn: false });
	        }
	      });
	  }
	
	  render() {
	    // Extract data from state
	    const { form, error, isSigningIn } = this.state;
	
	    return (
	      <div className="Login">
	        <div className="title">Elemental Battles - powered by EOSIO</div>
	        <div className="description">Please use the Account Name and Private Key generated in the previous page to log into the game.</div>
	        <form name="form" onSubmit={ this.handleSubmit }>
	          <div className="field">
	            <label>Account name</label>
	            <input
	              type="text"
	              name="username"
	              value={ form.username }
	              placeholder="All small letters, a-z, 1-5 or dot, max 12 characters"
	              onChange={ this.handleChange }
	              pattern="[\.a-z1-5]{2,12}"
	              required
	            />
	          </div>
	          <div className="field">
	            <label>Private key</label>
	            <input
	              type="password"
	              name="key"
	              value={ form.key }
	              onChange={ this.handleChange }
	              pattern="^.{51,}$"
	              required
	            />
	          </div>
	          <div className="field form-error">
	            { error && <span className="error">{ error }</span> }
	          </div>
	          <div className="bottom">
	            <Button type="submit" className="green" loading={ isSigningIn }>
	              { "CONFIRM" }
	            </Button>
	          </div>
	        </form>
	      </div>
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
	export default connect(mapStateToProps, mapDispatchToProps)(Login);
```


再在App.jsx中更新引用

```Bash
	cd /eos-work/frontend/src/components/App/

	vi App.jsx
```
更新代码如下：

{% codeblock lang:C %}	
	// React core
	import React, { Component } from 'react';
	import { connect } from 'react-redux';
	// Components
	import { Game, Login } from 'components';
	// Services and redux action
	import { UserAction } from 'actions';
	import { ApiService } from 'services';
	
	class App extends Component {
	
	  constructor(props) {
	    // Inherit constructor
	    super(props);

	    // 此处添加状态，默认loading为true
	    // State for showing/hiding components when the API (blockchain) request is loading
	    this.state = {
	      loading: true,
	    };
	    // Bind functions
	    this.getCurrentUser = this.getCurrentUser.bind(this);
	    // Call getCurrentUser before mounting the app
	    this.getCurrentUser();
	  }
	
	  getCurrentUser() {
	    // Extract setUser of UserAction from redux
	    const { setUser } = this.props;
	    // Send a request to API (blockchain) to get the current logged in user
	    return ApiService.getCurrentUser()
	      // If the server return a username
	      .then(username => {
	        // Save the username to redux store
	        // For structure, ref: ./frontend/src/reducers/UserReducer.js
	        setUser({ name: username });
	      })
	      // To ignore 401 console error
	      .catch(() => {})
		  //在获取到区块链数据反馈后，设置loading状态为false
	      // Run the following function no matter the server return success or error
	      .finally(() => {
	        // Set the loading state to false for displaying the app
	        this.setState({ loading: false });
	      });
	  }
	
	  render() {
	    // Extract data from state and props (`user` is from redux)
	    const { loading } = this.state;
	    const { user: { name, game } } = this.props;
	
		// 确定游戏处于什么状态
	    // Determine the app status for styling
	    let appStatus = "login";
	    if (game && game.status !== 0) {
	      appStatus = "game-ended";
	    } else if (game && game.selected_card_ai > 0) {
	      appStatus = "card-selected";
	    } else if (game && game.deck_ai.length !== 17) {
	      appStatus = "started";
	    } else if (name) {
	      appStatus = "profile";
	    }
	
	    // Set class according to loading state, it will hide the app (ref to css file)
	    // If the username is set in redux, display the Game component
	    // If the username is NOT set in redux, display the Login component
	    return (
	      <div className={ `App status-${ appStatus }${ loading ? " loading" : "" }` }>
	        { name && <Game /> }
	        { !name && <Login /> }
	      </div>
	    );
	  }
	
	}
	
	// Map all state to component props (for redux to connect)
	const mapStateToProps = state => state;
	
	// Map the following action to props
	const mapDispatchToProps = {
	  setUser: UserAction.setUser,
	};
	
	// Export a redux connected component
	export default connect(mapStateToProps, mapDispatchToProps)(App);
```

# 3 测试代码

```Bash
	cd /eos-work/frontend

	npm start
```

在浏览器中输入网址测试。

点击右边的rules按钮查看游戏规则帮助

![游戏规则帮助](/images/eost14-01.png "游戏规则帮助")	

选择卡牌后，在等待时会出现进度加载提示：

![游戏主界面](/images/eost14-02.png "游戏主界面")

还有刷新后回到主界面等等，就不一一截图了。

至此，一个完整的EOS卡牌游戏就制作完成了。

# 4 后记
## 延伸阅读
在本文的学习中如果遇到问题，欢迎留言或者在如下链接寻找解决方案：

- EOS官方游戏开发第八课: https://battles.eos.io/tutorial/lesson8/chapter1

## 请投票给柚翼节点
如果觉得这系列教程有点意思，<a href="https://www.myeoskit.com/tools/vote/?voteTo=eoswingdotio" >请投票给柚翼节点（eoswingdotio）</a>。您的投票是本教程持续更新的动力源泉，谢谢。
