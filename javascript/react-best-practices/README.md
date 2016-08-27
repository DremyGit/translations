# React最佳实践

> 原文[React Best Practices and Useful Functions](https://medium.com/@nesbtesh/react-best-practices-a76fd0fbef21?swoff=true#.4xyk6zpps)，作者Nessim Btesh，本文对原文有删节。

近年来，React已经成为开发者们建立从单页应用（SPA）到移动应用的一个新工具。但是自从我开始逐步深入React后，我发现所有看似非常酷的Node模块实际上对开发有非常不利的影响。它们不遵循任何规则，组件的体积巨大。它们为几乎所有的组件都使用了状态，也包括那些根本就不作输出的组件。任何有着足够经验的人都明白，维护这样的代码会有多麻烦，以及如果每次都渲染全部组件的时候，这在浏览器上加载会有多慢。在这篇文章中，我会带你探索React的最佳实践，包括如何设置React以及如何让它变得足够快。

> 在你开始阅读前请注意：React是一个函数式编程（Functional Programming，FP）库，如果你不知道函数式编程是什么，请阅读其它相关文章。

**使用ES6（通过[Babel](https://babeljs.io/)进行编译）**：ES6会让你的工作变得更容易许多，它使得JS不管是看起来还是感觉起来都更加的现代化。一个非常好的ES6的例子是Generator和Promis，记住当你需要做大量的嵌套调用的时候，你可以使用它们来进行异步调用。下面来介绍下同步和异步JS，这是有关Generator的一个示例：

```js
getJSON(url, function(response) {
    getJSON(url, function(response) {
      console.log(response);
    });
});
```

可以转变为这样的形式：

```js
function* getStockValue() {
    var entry1 = yield request('http://myrl.com/stock/key');
    var data1  = JSON.parse(entry1);
    var entry2 = yield request('http://myurl/stock/value');
    var data2  = JSON.parse(entry2);
}
```

**使用Webpack**：使用Webpack的理由很简单：热重载，压缩文件，Node模块加载，以及你可以把你的应用进行分片，从而可以进行延迟按需加载。

**总是注意你的打包文件大小**：让你的打包文件变小的一个途径是从Node模块相应目录直接导入：

```js
// 像这样做
import Foo from ‘foo/Foo’

// 而不是像这样做
import {Foo} from ‘foo’
```

**使用JSX语法**：如果你有Web开发的背景，[JSX](https://facebook.github.io/react/docs/jsx-in-depth.html)语法会让你感觉很自然。但是如果你的背景不是Web开发，你也不用太担心，JSX非常容易学习。**注意**如果你不使用JSX语法进行开发的话，代码维护会变得异常艰难。

**保持你的组件微型化（非常微小）**：最佳实践证明如果你的渲染函数超过10行就可能太大了，使用React的主要思想就是代码重用，所以如果你只是把所有的代码都扔到一个文件里面去，你就会失去React的优雅。

**使用`ShouldComponentUpdate`方法**：React是一个模板语言，在每次组件状态改变的时候都会进行渲染操作，所以想象一下每当页面中有动作发生时，整个页面都要被重新渲染，这会花费浏览器很长的加载时间。这就是[`ShouldComponentUpdate`方法](https://facebook.github.io/react/docs/component-specs.html#updating-shouldcomponentupdate)出现的原因，当React要重新渲染每一个组件时都要检查它`ShouldComponentUpdate`方法返回的结果是`false`还是`true`。所以只要你的组件是完全静态的，为了你自己着想，你应该返回`false`。如果组件不是静态的话，就检查它的的`props`或者`state`是否发生了改变。

**使用无状态组件**：不需要在每一个组件内部存储状态这一点已经不必多说，理想的情况下，你应该有一个全功能的父组件，并且里边所有的子组件都应该是无状态的，只是接收传来的`props`并且在里边没有任何的逻辑处理。你能够像这样来创建一个无状态组件：

```js
const DumbComponent = ({props}) => {
  return (<div />);
}
```

无状态组件也非常容易调试，因为它坚持自顶向下的思想，而这正是React的全部。

**总是在构造方法中对函数进行绑定**：任何有状态的组件都要尽量在构造方法中进行函数绑定。

```js
export default class BindFunctionExample extends React.Component {
    constructor() {
        super();
        this.state = {
            hidden: true,
        };
        this.toggleHidden = this.toggleHidden.bind(this);
    }
    
    toggleHidden() {
            const hidden = !this.state.hidden;
            this.setState({hidden})
    }
    
    render(){
        return(
            <button onClick={this.toggleHidden} />
        );
    }
}
```

**使用Redux/Flux**：处理数据时，你应该使用Flux或者Redux。Flux/Redux允许你处理方便地处理数据以及远离因处理前端缓存时候而产生的病痛。我个人使用Redux因为它强制你对文件目录结构有更好的安排。

**使用Normalizr**：既然我们谈到了数据，我想花一些时间来介绍一下处理复杂数据结构的最佳方式。[Normalizr](https://github.com/paularmstrong/normalizr)可以把你的嵌套json对象格式化成为简单的数据结构，让你能够很方便的对数据进行操作。

**文件目录结构**：在这直言不讳地说，我只看过两种文件目录结构，通过React/Redux使得一切变得简单：

第一种文件目录结构：

![first-structure](https://cdn-images-1.medium.com/max/800/1*q0eeZMnRokYYVAESn163jw.png)

第二种文件目录结构：

![second-structure](https://cdn-images-1.medium.com/max/800/1*9UplJ2y-A2vfBs32nTCjTg.png)

**使用Container**：你应该避免把每一个组件与Flux/Redux的Store直接连接起来，这就是你应该使用Container为下层传递数据的原因。最好的方式是创建两个Container：一个包含所有的安全组件（需要认证的组件），另一个包含所有的不安全组件。创建一个父Container的最好方式是克隆所有子节点并且把期望的Props传递下去：

```js
class Container extends React.Component {
    render(){
        var { props } = this;
        return(
                <div className="main-content">
                    {  
                        React.Children.map(this.props.children, function(child) {
                            return React.cloneElement(
                                child, 
                                { ...props }
                            );
                        })
                    }
                </div>
        );
    }
}

const mapStateToProps = (state) => {
    return state;
};

function mapDispatchToProps(dispatch) {
    return {
        actions: bindActionCreators(actions, dispatch)
    };
}

export default connect(
    mapStateToProps,
    mapDispatchToProps
)(Container);
```

（以下是译者的补充）

**使用Immutable.js作为Redux中Store的数据结构**：[Immutable.js](https://facebook.github.io/immutable-js/)即为不可变数据结构，在判断对象是否相等时，可以通过最外层引用级别的比较来替代复杂的深层比较，用在`ShouldComponentUpdate`方法中能够极大地提升性能。另外，具有函数式编程特性的Immutable.js能够非常方便地对数据进行处理，如查找、排序、合并、删除等等，与Normalizr搭配使用能够极大地提升开发效率，值得使用。对于Immutable.js的介绍可看我另一篇译文[《Immutable.js介绍与函数式编程概念》](https://github.com/DremyGit/translations/tree/master/javascript/introduction-to-immutablejs-and-functional-programming-concepts)。