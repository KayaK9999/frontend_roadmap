
```
let timerId = setTimeout(func|code, [delay], [arg1], [arg2], ...)
```

`arg1`，`arg2`…要传入被执行函数（或代码字符串）的参数列表（IE9 以下不支持）
`setTimeout` 在调用时会返回一个“定时器标识符（timer identifier）”，在我们的例子中是 `timerId`，我们可以使用它来取消执行。

```
let timerId = setInterval(func|code, [delay], [arg1], [arg2], ...)
```
所有参数的意义也是相同的。不过与 `setTimeout` 只执行一次不同，`setInterval` 是每间隔给定的时间周期性执行。
想要阻止后续调用，我们需要调用 `clearInterval(timerId)`。