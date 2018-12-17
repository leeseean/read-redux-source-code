## 导读

我们经常使用compose来合并reducer，那么compose内部是怎么合并多个reducer的呢？

下面上源码

## 源码

```
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

如上所示，参数funcs是一组reducer函数，
如果没有传入reducer，则默认返回一个函数。
如果只传入一个reducer，则返回这个reducer。
如果传入多个reducer，运用js数组的原生reduce方法层层调用达到每个reducer都能调用到的效果。