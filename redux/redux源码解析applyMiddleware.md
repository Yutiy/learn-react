## middleware
比如我们需要对每次`dispatch(action)` 都做一次日志记录，方便记录用户的行为，又或者你在某些操作前和操作后都要
获取服务端的数据，这时就可能需要对`dispatch` 或`reducer` 做一些封装了， 这时，`middleware`就可以派上用场了。

```javascript
const printLogMiddleware = ({ getState }) => next => action => {
  console.log('state before dispatch', getState())

  let returnValue = next(action)                        // 相当于dispatch一个action，返回的还是这个action

  console.log('state after dispatch', getState())
  return returnValue                                    // 继续传给下一个中间件作为参数action
}
```

## enhancer
记得在 `createStore` 中有这么一句`return enhancer(createStore)(reducer, preloadedState)`，其中，`enhancer`表示增强器，就是对生成的
`store` API进行改造，它与中间件最大的区别就是它会修改 `store` 的API。

修改`store` API需要从`createStore`开始入手，因此，**redux** 提供了`applyMiddleware` 这样的API:
```javascript
import compose from './compose'   // 导入compose方法

export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    const store = createStore(reducer, preloadedState, enhancer)
    let dispatch = store.dispatch
    let chain = []

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }

    chain = middlewares.map(middleware => middleware(middlewareAPI))      // 生成一个middleware链
    dispatch = compose(...chain)(store.dispatch)        // 将多个middleware串联起来执行

    return {
      ...store,
      dispatch,    // 新 dispatch 覆盖原 dispatch，往后调用 dispatch 就会触发 chain 内的中间件链式串联执行
    }
  }
}
```
