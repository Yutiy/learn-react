## 异步操作thunk
```javascript
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {    // next 就相当于传进来的store.dispatch
    if (typeof action === 'function') {                     // 判断传入的action是否是一个函数
      return action(dispatch, getState, extraArgument)
    }

    // 用next而不是dispatch，保证可以进入下一个中间件
    // 如果在这里执行 dispatch 就会从最一开始的中间件重新再走一遍，如果 middleWare 一直调用 dispatch 就可能导致无限循环
    return next(action)
  }
}

const thunk = createThunkMiddleware()
thunk.withExtraArgument = createThunkMiddleware

export default thunk
```

可以看出其实现非常简单，这里拥有三层函数:
- `({ dispatch, getState }) =>` 这一层的逻辑就是对应 `applyMiddleware` 中的 `middleware(middlewareAPI)`
- `next =>` 对应`applyMiddleware` 中`compose` 链的逻辑
- `action =>` 即是用户操作的action

**下面我们来看这么一段创建`store` 的方法:**
> const store = createStore(reducers, compose(applyMiddleware(thunk), window.devToolsExtension ? window.devToolsExtension() : f => f))

- 首先可以看到创建`store` 的时候传入了不止一个参数
- 所以我们的逻辑就会变成`compose(applyMiddleware(thunk, log), window.devToolsExtension ? window.devToolsExtension() : f => f)(createStore)(reducer, preloadedState)`
  - 先看`compose(applyMiddleware(thunk, log), window.devToolsExtension ? window.devToolsExtension() : f => f)`,
  - applyMiddleware(thunk, log)，逐个执行，返回一个新的`dispatch`
  - 然后compose逐个执行，返回一个新的函数，最后依次把`createStore` 和 `(reducer, preloadedState)` 传入进行执行
