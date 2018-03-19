# warning方法

redux源码中utils文件夹中封装了一个对`warning` 警告信息的方法，非常简单，我们来看一下:

```javascript
/**
 * Prints a warning in the console if it exists.
 *
 * @param {String} message The warning message.
 * @returns {void}
 */
export default function warning(message) {
  /* eslint-disable no-console */
  if (typeof console !== 'undefined' && typeof console.error === 'function') {
    console.error(message)
  }
  /* eslint-enable no-console */
  try {
    // This error was thrown as a convenience so that if you enable
    // "break on all exceptions" in your console,
    // it would pause the execution at this line.
    throw new Error(message)
  /* eslint-disable no-empty */
  } catch (e) { }
  /* eslint-enable no-empty */
}
```

首先传入一个`message`，表示需要在控制台打印的信息内容，然后判断了`console` 对象及其 `error` 方法，其中混入了对 `eslint` 的支持，并且注释十分友好，浅显易懂

