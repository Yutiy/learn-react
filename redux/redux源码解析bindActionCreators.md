# bindActionCreators
这个API有点鸡肋，它就做了一件事情: `dispatch(bindActionCreators(xxx))`

比如我们需要派发一个action，可以这样使用: `dispatch({ type: 'LOADING', loading: false } })`

为了复用action，可以抽离出一个函数出来:
```javascript
export const isFetching = ({ loading }) => ({
  type: 'LOADING',
  loading
})

dispatch(isFetching({ loading: false }))
```

为了不显示的调用dispatch，就可以使用`bindActionCreators` 来进行处理了，`bindActionCreators(isFetching, dispatch)`，
这里会返回一个函数，然后我们就可以传入参数进行调用了。


源码如下，可以看到`actionCreator`，支持函数和对象两种情况:
```javascript
// 为actionCreator 加上自动 dispatch 技能
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args))
}

export function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  // 错误处理
  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${actionCreators === null ? 'null' : typeof actionCreators}. ` +
      `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

  const keys = Object.keys(actionCreators)
  const boundActionCreators = {}
  for (let i = 0; i < keys.length; i++) {
    const key = keys[i]
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)   // 循环遍历，进行dispatch
    }
  }
  return boundActionCreators
}
```
