# 使用Promise/Generators/Coroutines写现代异步Javascript

> 本文由原作者William Gottschalk原著于[https://medium.freecodecamp.com/write-modern-asynchronous-javascript-using-promises-generators-and-coroutines-5fa9fe62cf74](https://medium.freecodecamp.com/write-modern-asynchronous-javascript-using-promises-generators-and-coroutines-5fa9fe62cf74#.46mrb735k)，文章原标题：《Write Modern Asynchronous Javascript using Promises, Generators, and Coroutines》

近几年，“回调地狱（Callback Hell)”一词经常被提及，成为Javascript并发管理中最为讨厌的设计之一。它让你忘记了代码本来应有的样子，以下便是Express中验证和处理一个交易的例子：

```js
app.post("/purchase", (req, res) => {
    user.findOne(req.body, (err, userData) => {
        if (err) return handleError(err);
        permissions.findAll(userData, (err2, permissions) => {
            if (err2) return handleError(err2);
            if (isAllowed(permissions)) {
                transaction.process(userData, (err3, confirmNum) => {
                    if (err3) return handleError(err3);
                    res.send("Your purchase was successful!");
                });
            }
        });
    });
});
```

## Promise应该可以拯救我们

Promise允许Javascript开发者像书写同步的代码一般书写异步代码，我们只需要把异步函数包裹在一个特殊的对象里即可。如果要访问Promise对象的值的话，只需要通过Promise对象的`.then`或者`.catch`方法即可获取。但当我们尝试通过Promise来重构上面的代码会发生什么呢？

```js
// 所有的异步方法已经被promise化了
app.post("/purchase", (req, res) => {
    user.findOneAsync(req.body)
        .then( userData => permissions.findAllAsync(userData) )
        .then( permissions => {
            if (isAllowed(permissions)) {
                return transaction.processAsync(userData);
                // userData是undefined，这不在相应的作用域中
            }
        })
        .then( confirmNum => res.send("Your purchase was successful!") )
        .catch( err => handleError(err) )
});
```

这样每一个回调函数属于一个单独的作用域，我们便不能在第二个`.then`回调函数里面访问`user`对象了。

在一阵思考过后，我仍然无法找到一个优雅的解决办法，只是找到了一个令人沮丧的办法：

> 只需要把你的Promise对象缩进，让他们有合适的作用域即可

把Promise对象缩进！？这不就又回到了原本锥型的样子了吗？

```js
app.post("/purchase", (req, res) => {
    user.findOneAsync(req.body)
        .then( userData => {
            return permissions
                .findAllAsync(userData)
                .then( permissions => {
                    if (isAllowed(permissions)) {
                        return transaction.processAsync(userData);
                    }
            });
        })
        .then( confirmNum => res.send("Your purchase was successful!"))
        .catch( err => handleError(err) )
});
```

我还计较原本那个嵌套的回调函数版本比这个嵌套的Promise版本看起来更清晰易懂呢。

## Async/Await会拯救我们的

`async`和`await`关键字可以让我们当做写同步代码一样写Javascript代码。以下便是使用ES7语法写成的代码：

```js
app.post("/purchase", async function (req, res) {
    const userData = await user.findOneAsync(req.body);
    const permissions = await permissions.findAllAsync(userData);
    if (isAllowed(permissions)) {
        const confirmNum = await transaction.processAsync(userData);
        res.send("Your purchase was successful!")
    }
});
```

不幸的是，包括`async/await`在内的大部分ES7的功能特性依旧没有被实现，因此，需要使用别的编译器来完成。但是，你能够使用ES6的特性来写十分类似于以上风格的代码，这已经被大多数现代浏览器和Node(v4.0+)实现了。

## Generators和Coroutine组合

generator(生成器函数）是一个很棒的元编程工具。它能用来进行惰性求值、遍历内存密集型数据集合以及从多个使用如`RxJs`库的数据源中按需处理数据。

但是，我们并不想在产品代码中只使用generator，因为它让我们不得不去推理执行的顺序。并且每次我们调用下一个函数的时候，都会像goto语句一样跳回到generator中。

coroutine知道这一点，它通过包裹generator解决了这个问题，并且通过抽象避免了复杂性。

## 使用Coroutine的ES6版本

coroutine允许我们一次`yield`一个异步函数，让代码看起来是同步的。

请注意我使用的co库，co的Coroutine会立即执行generator，但是Bluebird的Coroutine会返回一个函数，你必须调用这个函数来执行generator。

```js
import co from 'co';
app.post("/purchase", (req, res) => {
    co(function* () {
        const person = yield user.findOneAsync(req.body);
        const permissions = yield permissions.findAllAsync(person);
        if (isAllowed(permissions)) {
            const confirmNum = yield transaction.processAsync(user);
            res.send("Your transaction was successful!")
        }
    }).catch(err => handleError(err))
    // 如果在generator中的任意一步出现错误，coroutine会停止并且返回一个被reject的Promise对象
});
```

让我们来列举一些使用coroutine的基本原则：

1. 任意在`yield`右侧的函数必须返回一个Promise对象。
2. 如果你想立刻执行你的代码，请使用`co`。
3. 如果你想稍后再执行你的代码，请使用`co.warp`。
4. 保证在你coroutine的尾部调用了`.catch`去捕获处理错误。否则，你就应该把你的代码包裹在一个`try/catch`块之中。
5. `Bluebird`的`Promise.coroutine`等价于Co的`co.wrap`，但并不等同于`co`自己的函数。

## 那该如何并发运行多条语句呢？

你可以使用对象或者数组，带上`yield`关键字，之后就可以通过解构来获取结果。

```js
import co from 'co';
// 使用对象
co(function*() {
    const {user1, user2, user3} = yield {
        user1: user.findOneAsync({name: "Will"}),
        user2: user.findOneAsync({name: "Adam"}),
        user3: user.findOneAsync({name: "Ben"})
    };
).catch(err => handleError(err))

// 使用数组
co(function*() {
    const [user1, user2, user3] = yield [
        user.findOneAsync({name: "Will"}),
        user.findOneAsync({name: "Adam"}),
        user.findOneAsync({name: "Ben"})
    ];
).catch(err => handleError(err))
```

```js

// 使用Bluebird库
import {props, all, coroutine} from 'bluebird';

// 使用对象
coroutine(function*() {
    const {user1, user2, user3} = yield props({
        user1: user.findOneAsync({name: "Will"}),
        user2: user.findOneAsync({name: "Adam"}),
        user3: user.findOneAsync({name: "Ben"})
    });
)().catch(err => handleError(err))

// 使用数组
coroutine(function*() {
    const [user1, user2, user3] = yield all([
        user.findOneAsync({name: "Will"}),
        user.findOneAsync({name: "Adam"}),
        user.findOneAsync({name: "Ben"})
    ]);
)().catch(err => handleError(err))
```

## 当前你能用到的库：

+ [Promise.coroutine | bluebird](http://bluebirdjs.com/docs/api/promise.coroutine.html)
+ [co](https://www.npmjs.com/package/co)
+ [Babel](https://babeljs.io/)
+ [asyncawait](https://www.npmjs.com/package/asyncawait)