---
title: 学习理解Redux Middleware
date: 2018-05-27 12:33:20
description: Redux中的middleware其实就像是给你提供一个在action发出到实际reducer执行之前处理一些事情的机会
---

Redux中的middleware其实就像是给你提供一个在action发出到实际reducer执行之前处理一些事情的机会。可以允许我们添加自己的逻辑在这段当中。<b>它提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。</b>

加入middleware后，整个数据的流动如下图所示：![middleware-data-flow](/images/reduxMiddleware/reduxmiddleware.png)

举个简单的例子，我们使用middleware将每次action的执行详细信息都打出来。就用官方demo中的todoApp来举例，我们先实现一个简单的reducer用来添加一个todo:
``` javascript
    const todoApp = (state = {todo: []}, action) => {
        if (action.type === 'addTodo') {
            state.todo = [...state.todo, action.value]
        }
        return state
    }
```
然后再补上其他逻辑测试，用最原始的方法实现将每次action的执行信息log出来：
``` javascript
    const redux = require('redux')

    const todoApp = (state = {todo: []}, action) => {
        if (action.type === 'addTodo') {
            state.todo = [...state.todo, action.value]
        }
        return state
    }

    let store = redux.createStore(todoApp);

    const action = {
        type: 'addTodo',
        value: 'todo',
    }
    console.log('state: ', store.getState());
    console.log('action: ', action);
    store.dispatch(action)
    console.log('next state: ', store.getState())
```
如果不出什么意外的话，我们这段代码应该会成功运行，并且将这段log出这个action的运行情况， 如下图：![noMiddlewarelogout.png](/images/reduxMiddleware/noMiddlewarelogout.png)

接下来我们将log这件事尝试使用redux的middleware来完成。

1. 首先，根据我们之前的了解，middleware其实是一段在action到reducer之间的处理逻辑。我们都知道，标准的一个redux发送一个action是调用store自身的`dispatch`方法。那么，我们想要在一个action到达reducer之前去做些处理的话，最好的地方应该就是尝试将store的`dispatch`替换为我们自己的，在其中加上我们的处理逻辑，例如打印log这件事。
    ``` javascript
        let store = redux.createStore(todoApp);
        const next = store.dispatch;
        const dispatchWithLog = (action) => {
            console.log('state: ', store.getState());
            console.log('action: ', action);
            next(action);
            console.log('next state: ', store.getState());
        }
        store.dispatch = dispatchWithLog;
    ```
    我们将默认store的`dispatch`替换为自己的`dispatchWithLog`, 通过这种方式，完成了我们的需求，只要任何地方调用了store的`diapatch`去发送新的action, 我们都能讲其log出来，这个看起来已经有一点middleware的意思了。

2. 虽然上面已经可以解决问题了，并且已经有点middleware的意思了，但是还有一点硬伤就是，需求多了就比较难搞了，例如就像官方上既需要log又需要Crash Reporting, 再通过这种方式去处理就显得很不优雅。Crash Reporting的middle的一个简单实现如下：
    ``` javascript
        const next1 = store.dispatch;
        const dispatchWithCreshReporting = (action) => {
            try {
                next1(action);
            } catch (err) {
                console.error(err);
            }
        }
        store.dispatch = dispatchWithCreshReporting;
    ```
    我们在log之后加上这段代码, 其实就是在log那件事替换完之后，我们再次替换store上的`dispatch`，其实这个时候，我们拿到的`next1`已经是log那个替换过后的`dispatch`方法了，里面有打印log的逻辑，所以，我们在后面通过`store.dispatch`去发送action时，每一个action都会先经过Crash Reporting包装的`dispacth`方法（其中包含了Crash Report的逻辑），然后再经过log包装过后的`dispatch`方法（其中包含了打印log的逻辑）。这其实就是redux middleware的基本思想。

3. 当然，redux本身给我们提供了包装过后的工具方法来专门应用middleware。其中也不是简单粗暴的替换store上的dispatch了。这个方法即为`applyMiddleware`。

4. 官方的doc也给出了一个关于`applyMiddleware`的一个简单粗暴的直接替换`dispatch`的一个示例，如下：
    ``` javascript
        function applyMiddlewareByMonkeypatching(store, middlewares) {
            middlewares = middlewares.slice()
            middlewares.reverse()
        ​
            // Transform dispatch function with each middleware.
            middlewares.forEach(middleware =>
                store.dispatch = middleware(store)
            )
        }
    ```
    关于先逆序middlewares再进行替换，这里主要是为了，让middleware的执行顺序按照我么传给他的array顺序来进行。就像我们上面直接替换的那个例子，越往后面进行替换`dispatch`的在执行过程中先运行。

5. 当然，官放的具体实现中不是这么简单粗暴的直接替换的方式，因为一来不够优雅，这种方式在链式的调用过程中有可能出现问题。比如某一个middleware并不是同步执行的，这样在进行`store.dispatch = middleware(store)`就有可能到下一个middleware时，`store.dispatch`还没有被替换。因此，官方的middleware是接受一个`next`的参数来，来拿到dispatch，并不是直接从store上对dispatch进行操作的。

6. 一般一个标准的middleware是这个样子的，我们使用最初的log的那个middleware来举例，让它接受一个`next`(就是一个下一个的dispatch方法)，再返回一个`dispatch`方法。
    ``` javascript
        function logger(store) {
            return function(next) {
                return function(action) {
                    console.log('state: ', store.getState());
                    console.log('action: ', action);
                    let result = next(action);
                    console.log('next state: ', store.getState());
                    return result;
                }
            }
        }

        function creshReporting(store) {
            return function(next) {
                return function(action) {
                    try {
                        return next(action);
                    } catch (err) {
                        console.error(err);
                        return next;
                    }
                }
            }
        }
    ```
    然后假设我们在apply时这样应用一下：
    ``` javascript
        function applyMiddleware(store, middlewares = [logger, crashReporting]) {
            middlewares = middlewares.slice()
            middlewares.reverse()
            let dispatch = store.dispatch
            middlewares.forEach(middleware =>
                dispatch = middleware(store)(dispatch)
            )
            return Object.assign({}, store, { dispatch })
        }
    ```
    这样就能够进行优雅的链式调用了。并且用上ES6箭头函数后，这样写出来会更加的优雅：
    ``` javascript
        const logger = store => next => action => {
            console.log('state: ', store.getState());
            console.log('action: ', action);
            let result = next(action);
            console.log('next state: ', store.getState());
            return result;
        }
    ```
7. 最后，其实redux middleware使用起来其实是非常的方便的，只需要记住`applyMiddleware`这个API即可。即`const store = createStore(reducer, applyMiddleware(middlewares))`


Reference： [Redux Middleware Doc](https://redux.js.org/advanced/middleware)