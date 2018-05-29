---
title: 学习理解Redux Middleware
date: 2018-05-27 12:33:20
tags: Redux, Middleware
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
    我们在log之后加上这段代码

Reference： [Redux Middleware Doc](https://redux.js.org/advanced/middleware)