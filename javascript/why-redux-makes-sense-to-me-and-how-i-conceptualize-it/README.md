# 对Redux的理解及类比

> 原文[Why Redux makes sense to me and how I conceptualize it](https://medium.freecodecamp.com/why-redux-makes-sense-to-me-and-how-i-conceptualize-it-c8a3a9db15ca#.o6tvum1ir)，作者Willson Mock。

当我一开始学习React的时候，记得我读了很多与它相关，却有着不同技术的文章。

我想特别指出，其中有一篇提到React的生态圈非常混乱，以至于让开发者觉得在使用React之前必须掌握其它所有与之相关的类库。

和一些有着8个月React使用经历的人一样，我仍然只是浮于整个生态圈的表面，甚至不知道这个庞大的系统是如何工作的。

但我花费在React中的时间已经足以让我明白，什么时候适合使用另外一个相关技术——Redux（一个Flux架构的实现），以及我为什么要使用它。

> 当你需要用到Flux的时候，你就会知道。如果你不确定你是否真的需要它，那么就说明你不需要。

这句话来自于我[上边提到的那篇文章](https://github.com/petehunt/react-howto#learning-flux)，这就是我对于Redux的感悟。

一开始我的确没有发现有任何使用Flux的需要，我觉得能够只依靠React来创建产品级应用，但我开始看到了Redux的好处。

在这篇文章中，我会探讨为什么我认为Redux由他存在的理由，并且为了能够更好地了解Redux内部不同部分，我还提供了一些生动的比喻来说明。

## 为什么Redux有存在的理由

以下是我对于过去8个月使用React的一些感想：

1. 尽可能多地使用无状态的组件，即纯组件。纯组件能够更好地推理和测试。
2. 当我需要把一些状态合并到我的组件当中的时候，我都会小心的判断应该由哪个组件来管理这个状态。它到底是应该是一个无状态组件的直接父组件负责管理，还是应该由其他组件也需要的高阶状态组件来负责管理？
3. 能够回答出上条的问题是开发React应用非常重要的一步。事实上，即使你使用Redux（或者其他一些Flux的实现），你也仍然需要知道这一点，因为这取决于是哪些组件升级为容器（Container）组件（例如智能组件）。
4. 当我的程序变得越来越大时，我发现整个程序中有很多只管理自己状态的组件，也因为有很多个组件需要访问同一状态而需要经常进行重构。这种情况下我只能把那个状态组件提升，让这个组件成为那些组件的父组件来管理那些状态。
5. 我越来越多地会遇到这个情况，我开始想一个更好的解决办法：是否把所有的组件状态都放到一个顶层的高阶组件中去，然后只把下层组件需要的状态分别传递下去。我考虑这个方法的原因在于，除了最顶层的组件，每一个下层组件都能变成无状态组件（通常可能会把一些UI组件状态放在某些子组件中）。

## 介绍Redux

React是一个UI库，它本应只用来处理UI组件及数据的渲染。如果数据改变了，它应该重新渲染一个新的UI。

可是，当我的程序变复杂之后，我就会把状态和业务逻辑代码弄得到处都是。这样使得组件的重用变得困难，并且让我的组件都变得臃肿不堪。

Redux尝试将应用数据和业务逻辑放到它自己的容器中去，从而让React能够只负责管理自己的视图。这样就能够使你的软件更加灵活，因为你可能可以用其它的视图库来替代React。

## 虽然我不是Redux专家，但我能把它的不同部分形象地表示出来

当你第一次学习Redux的时候，有一些关键概念需要弄清楚：**store，actions / action生成函数，和reducer函数**。[官方文档](http://redux.js.org/)讲的非常棒，我十分推荐阅读官方文档及其中的代码来学习。如果你先阅读了官方文档，你就会对下面的例子熟悉许多。

我是一个以视觉学习为主的人，这是我自己理解的一些概念：

！[](https://cdn-images-1.medium.com/max/800/1*ELf2a1L1gSkDbX6wlW3CyA.jpeg)

### Store - 篮球筐

在Redux中，你整个应用状态都被一个store来管理。你能想象作为一个Javascript对象的store，每一个key都是一个状态，而你想获取的每一个值都是对应的状态的值。

你可能想问，store和篮球框有什么联系？我的比喻有一些灵活性，比如说在篮球赛中你想得分的话，只能把篮球投进篮筐才行。在Redux中也一样，只有将action（请看下面）传入store中，才是修改应用状态的唯一方式。

使用术语的话就应该是：将action dispatch到store时修改Redux状态的唯一方式。

```js
// 下面是一些伪代码

import { createStore } from "redux";

const team1Hoop = createStore( /* 现在可以忽略这些参数 */ );

/*
 * 上面创建的store用来管理你整个应用的状态，你能够想象出来
 * 作为一个巨大的Javascript对象，store的每一个key都是应用
 * 状态的一部分，状态值是它的value。
 *
 * 我不知道在store中状态是如何管理的，但我猜可能是像这样：
 *
 * team1Hoop.__applicationState__ = {
 *   totalPoints: 0,
 *   playerList: [ 球队球员list ]
 * };
 *
 */
```

### Action - 篮球

上文提到action，但没有对它作出解释，他们只是简单的POJO（Plain Old Javascript Object，持久化Javascript对象）。下面是它们的一些示例代码：

```js
/*
 * action是持久化Javascript对象，他们建立在某个特殊的接口对象之上，
 * 要求必须拥有type属性，但对于其它的属性，你都可以任意添加
 */

const FREE_THROW = "FREE_THROW";
const TWO_POINT_SHOT = "TWO_POINT_SHOT";

// action的一个示例
const twoPointer = {
  type: TWO_POINT_SHOT,
  payload: {
    points: 2
  }
};

// action的另一个示例
const freeThrow = {
  type: FREE_THROW,
  payload: {
    points: 1
  }
};
```

当把应用作为一个整体考虑时，你应该写出全部可能用到的action。

请记住action是声明式的，它们只是描述你在应用里能做什么，而不管如何去实现。action都是纯粹的数据。

必须反复说明的是，为了改变应用的状态，你必须把action“投进”store中才行。

```js
const team1Hoop = createStore( /* 现在可以忽略这些参数 */ );

const twoPointer = {
  type: TWO_POINT_SHOT,
  payload: {
    points: 2
  }
};

/*
 * 改变应用状态的唯一方式就是将action dispatch到store中去。
 */
team1Hoop.dispatch(twoPointer);

/*
 * 在store中的状态现在可能像这样了：
 *
 * team1Hoop.__applicationState__ = {
 *   totalPoints: 2,
 *   playerList: [ list of players on team ]
 * };
 */
```

### Reducer - 球队中的教练和球员

在上面我们说到，action用来描述应用中可以做的行为，但它们不知道这些action到底是如何修改应用状态的。这就是reducer的工作了。

reducer是一个纯函数，它将目前的状态和action作为两个参数输入，然后输出下一个状态。

reducer是纯函数，不会有任何的副作用，这是非常必要的。每一次提供相同的输入，都应该得到相同的输出。给定一个初始状态和一系列action，你就能知道在每一个action执行后状态的结果是什么，这听起来相当酷。

可能会有一个单独的reducer函数来管理入口状态，接收每一个action并进行转换，Redux特别适用**reducer composion**来称呼它。将一个巨大的reducer函数拆分成许多子reducer函数来分别处理整个应用状态的一小部分，这是多么美好的一件事。

继续我的比喻，你可以把那个专门负责合并操作的reducer当作是教练，其它的子reducer函数当做是队员。一旦action“投进”了store中，负责合并操作的reducer就“捕获”到了这个action然后将它“传递”给每一个子reducer函数中。每一个子reducer函数都会检测这个action，决定自己是否需要针对这个action做出应用状态上的改变。如果是的话，它就会产生一个新的状态。

代码如下：

```js
// 下面是伪代码

import { combineReducers, createStore } from "redux";

// 这个子reducer函数只负责管理全局状态的比分部分
function pointsReducer(pointsState = 0, action) {
  switch (action.type) {
    case TWO_POINT_SHOT:
    case FREE_THROW:
      return pointsState + action.payload.points;
    default:
      return pointsState;
  }
}

// 这个子reducer函数只负责管理全局状态的球员list部分
function playerListReducer(playerListState = [], action) {
  switch (action.type) {
    case ADD_PLAYER:
      return playerListState.concat(action.payload.player); // no mutations allowed!
    default:
      return playerListState;
  }
}

// 负责合并的reducer会把每个action传递给每一个子reducer函数
const coach = combineReducer({
  points: pointsReducer,
  playerList: playerListReducer
});

const twoPointer = {
  type: TWO_POINT_SHOT,
  payload: {
    points: 2
  }
};

const team1Hoop = createStore(coach); // 创建store时，传递reducer函数作为参数

team1Hoop.dispatch(twoPointer);
```

## 啥？

在这一节，你可能有一个合理的提问：在每一个子reducer函数生成了下一个状态之后，发生了什么？

在每一个子reducer函数生成了相应的下一个状态之后，就会产生一个更新后的全局状态对象，并且存储在store当中。记住，store是整个全局状态中的唯一可信资源，在每一个action传入reducer后，都会产生一个新的状态，并且存入store当中。

你可能想知道这个过程是如何发生的——action最开始是如何创建的？

Redux介绍了另一个概念，叫action生成器，这是一个产生并返回action的函数。action生成器勾住React组件，当用户和UI交互时，action生成器就会执行并且创建一个新的action，然后dispatch到store当中。

## 结论

这篇文章的目的不是教你Redux的输入和输出，而是帮助更多以视觉学习为主的学习者了解Redux中不同部分之间的交互过程。希望这篇文章能够给你一点启发。

我最近看了[Dan Abramov的视频](https://egghead.io/lessons/javascript-redux-extracting-container-components-filterlink)，它也提到了我上面所说的关于在React中不使用Redux的后果。你几乎总要通过中间组件传递许多props给子组件，即使这些中间组件并不需要它。这是一个解释React中Redux和高阶组件的很棒的视频。