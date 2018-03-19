## compose
这个API很简单，没有任何依赖，是一个纯函数

源码如下:
```javascript
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  // compose(f, g, h) is identical to doing (...args) => f(g(h(...args)))
  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

接收一组函数，实际使用可能是这样的`compose(f, g, h)(...arg) => f(g(h(...args)))`

## 浅析
```javascript
[f, g, h].reduce((a, b) => (...args) => a(b(...args)))

setp1: 因为reduce没有默认值，所以reduce的第一个值为f，第二个值为g，这个时候第一次循环返回 `(...args) => f(g(...args))`，先用 compose1 来代替
step2: 第二次循环，第一个值为第一次循环的返回值 compose1，第二个值为h，所以此时得到的值就是 `(...args) => compose1(h(...args))`，也就是 `f(g(h(...args)))`
```

简单举个例子来说明一下:
```javascript
function f(num) {
  console.log('f 获得参数 ' + num)    // 0
  return num + 1
}

function g(num) {
  console.log('g 获得参数 ' + num)    // 1
  return num + 2
}

function h(num) {
  console.log('h 获得参数 ' + num)    // 2
  return num + 3
}

let res1 = f(g(h(0)))
console.log(res1)      // 6

let res2 = compose(f, g, h)(0)
console.log(res2)     // 6
```

可以看到，使用 `compose` 来处理的话会优雅很多
