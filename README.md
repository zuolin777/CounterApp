最近的项目中使用了React。React真是火得一塌糊涂，在研究学习React的过程中，接触了`React-Native`，并且写了一个入门的app，基于Redux实现。

##一、环境搭建

　　按照[React-Native](http://facebook.github.io/react-native/)官网的说明，只有使用Mac才能开发iOS和Android项目（ Apple only lets you develop for iOS on a Mac），而windows只能开发Android的项目，首先介绍Mac下的环境配置。

###1.Xcode

　　安装Xcode是开发iOS必不可少，这里使用它的模拟器。
###2.node

　　node不用多说了。
###3.利用Homebrew安装watchman和flow

`$ /usr/bin/ruby -e
“$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”`
　　安装Homebrew
###4.安装wget
`$  brew install wget`
###5.通过brew安装watchman和flow
`$  brew install watchman`

`$  brew install flow`
###6.通过npm安装react-native-cli工具
`$  npm install -g react-native-cli`

　　如果提示没有权限，加上sudo就好了。
###7.安装nvm
`$ git clone https://github.com/creationix/nvm`

`$ cd nvm`

`$ source nvm.sh`

　　环境搭建完成，接下来创建一个hello world
##二、创建项目


###1.使用React-Native的command line来初始化一个项目

`$ react-native init HelloWorld`

`$ cd HelloWorld`

`$ react-native run-ios`

　　然后就会在模拟器中看到HelloWorld项目



　　BTW，接下来的项目中我使用了[yarn](https://yarnpkg.com/)来替代npm，效率会提高不少。
比如，在安装项目依赖时，npm命令`npm install`，而yarn则是`yarn`，够简单。

##三、修改项目
###1.入口文件
　　项目中的index.ios.js和index.android.js分别是ios和android的入口文件。
　　这个简单的项目是实现了按增加按钮使数字加1，按减小按钮使数字减1。使用redux来进行项目状态管理。
```
import {
    AppRegistry
} from 'react-native';

import App from './app/Components/App';

AppRegistry.registerComponent('CounterApp', () => App);
```

###2.Action
　　action是用户触发或程序触发的一个普通对象。
```
import { INCREASE, DECREASE } from '../constants/CounterTypes';

const CounterAction = {

    increase(){
        return {
            "type": INCREASE
        }
    },
    decrease(){
        return {
            "type": DECREASE
        }
    }
};

export default CounterAction;
```
　　CounterTypes是用到的常量。
```
export const INCREASE = "INCREASE";
export const DECREASE = "DECREASE";
```
###3.Reducer
　　reducer是根据action操作来做出不同的数据响应，返回一个新的state。
```
import { INCREASE, DECREASE } from '../constants/CounterTypes';

const counter = (state = 0,action) => {

    switch (action.type){

        case INCREASE:
            return state + 1;
        case DECREASE:
            return state - 1;
        default:
            return state;
    }

};

export default counter;
```
　　当你的项目大而复杂时，一个reducer显然是很乱的，可以将不同的reducer拆分，最后汇总在一个总的reducer里，这样使用的时候就不那么麻烦。

###4.Components
　　这里是组件相关。
```
import React, { Component } from 'react';

import { connect } from 'react-redux';
import CounterActions from '../actions/CounterAction';
import {
    Text,
    View,
    StyleSheet
} from 'react-native';

class Counter extends Component {

    render() {

        const {
            counter,
            decrease,
            increase
        } = this.props;

        return (
            <View style={styles.container}>
                <Text
                    style={styles.text}
                >{counter}</Text>
                <Text
                    style={styles.upDownStyle}
                    onPress={increase}
                >up</Text>
                <Text
                    style={styles.upDownStyle}
                    onPress={decrease}
                >down</Text>
            </View>
        )
    }
}

const styles = StyleSheet.create({
    text: {
        color: 'red',
        fontSize: 20
    },
    container: {
        flex: 1,
        backgroundColor: 'lightblue',
        justifyContent: 'center',
        alignItems: 'center'
    },
    upDownStyle: {
        backgroundColor: 'hotpink',
        margin: 10,
        width: 80,
        height: 30,
        lineHeight: 30,
        textAlign: 'center'
    }
});

export default Counter = connect(
    (state) => ({counter: state.counter}),
    {
        increase: CounterActions.increase,
        decrease: CounterActions.decrease
    }
)(Counter);
```
　　connect 主要为 React 组件提供 store 中的部分 state 数据 及 dispatch 方法，这样 React 组件就可以通过 dispatch 来更新全局 state。
　　两个按钮应该使用Button的啊，可是我用了Text，是因为我还没有弄好Button的样式啊。
###5.Store
在 Redux 项目中，Store 是单一的。维护着一个全局的 State，并且根据 Action 来进行事件分发处理 State。
```
import React, { Component } from 'react';

import Counter from './Counter';
import { createStore, combineReducers ,applyMiddleware } from 'redux';
import * as reducers from '../reducers';
import thunk from 'redux-thunk';
const createStoreWithMiddleware = applyMiddleware(thunk)(createStore);
const reducer = combineReducers(reducers);
const store = createStoreWithMiddleware(reducer);

import { Provider } from 'react-redux';

class App extends Component {

    render() {

        return (
            <Provider store={store}>
                <Counter/>
            </Provider>
        )
    }
}

export default App;
```
　　redux并没有进行异步处理的功能。但是可以自己编写符合需要的中间件，现在有第三方的中间件`redux-thunk`来处理异步请求。它的原理就是判断action是否是函数，如果是则进行递归式的操作。
　　`Provider`继承了 `React.Component`，所以可以以 React 组件的形式来为 Provider 注入 store，从而使得其子组件能够在上下文中得到 store 对象。


##四、参考资料
http://facebook.github.io/react-native/
http://redux.js.org/
