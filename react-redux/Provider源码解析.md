## Provider
`Provider` 组件通过`context` 方式将store 注入整个应用的入口组件，供子组件自由读取

```javascript
export function createProvider(storeKey = 'store', subKey) {
  const subscriptionKey = subKey || `${storeKey}Subscription`

  class Provider extends Component {
    // 声明context，注入store和可选的发布订阅对象
    getChildContext {
      return { [storeKey]: this[storeKey], [subscriptionKey]: null }
    }

    constructor(props, context) {
      super(props, context)
      this[storeKey] = props.store    // 缓存store
    }

    render() {
      return Children.only(this.props.children)   // 渲染输出的内容
    }
  }

  if (process.env.NODE_ENV !== 'production') {
    Provider.prototype.componentWillReceiveProps = function (nextProps) {
      // 如果开发环境下，store的引用发生改变的话，则发出警告
      if (this[storeKey] !== nextProps.store) {
        warnAboutReceivingStore()
      }
    }
  }

  Provider.propTypes = {
    store: storeShape.isRequired,
    children: PropTypes.element.isRequired,
  }

  // 声明childContextTypes，不声明的话context将是一个空对象
  Provider.childContextTypes {
    [storeKey]: storeShape.isRequired,
    [subscriptionKey]: subscriptionShape.isRequired
  }

  return Provider
}

export default createProvider
```