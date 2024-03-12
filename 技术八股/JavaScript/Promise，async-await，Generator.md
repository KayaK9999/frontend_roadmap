加载脚本时：存在Error优先回调但是随着调用嵌套的增加，代码层次变得更深，维护难度也随之增加，有时这些被称为“回调地狱”或“厄运金字塔”。

### Promise

传递给 `new Promise` 的函数被称为 **executor**。当 `new Promise` 被创建，executor 会自动运行。它包含最终应产出结果的生产者代码。当 executor 获得了结果，无论是早还是晚都没关系，它应该调用以下回调之一：
- `resolve(value)` —— 如果任务成功完成并带有结果 `value`。
- `reject(error)` —— 如果出现了 error，`error` 即为 error 对象。

`.then` 的第一个参数是一个函数，该函数将在 promise resolved 且接收到结果后执行。
`.then` 的第二个参数也是一个函数，该函数将在 promise rejected 且接收到 error 信息后执行。
如果只出现一个函数参数，则执行成功完成的情况

.catch语法与.then类似，只执行error的情况，与.then(null, f) 完全一样

.finally的功能是设置一个处理程序在前面的操作完成后，执行清理/终结。

如果 `.then`（或 `catch/finally` 都可以）处理程序返回一个 promise，那么链的其余部分将会等待，直到它状态变为 settled。当它被 settled 后，其 result（或 error）将被进一步传递下去。
新手常犯的一个经典错误：从技术上讲，我们也可以将多个 `.then` 添加到一个 promise 上。但这并不是 promise 链（chaining）

捕获所有 error 的最简单的方法是，将 `.catch` 附加到链的末尾
promise 的执行者（executor）和 promise 的处理程序周围有一个“隐式的 `try..catch`”。如果发生异常，它就会被捕获，并被视为 rejection 进行处理。所以下面两段代码工作上完全相同：

在 Promise 类中，有 6 种静态方法。详见 https://zh.javascript.info/promise-api
1. `Promise.all(promises)` —— 等待所有 promise 都 resolve 时，返回存放它们结果的数组。如果给定的任意一个 promise 为 reject，那么它就会变成 `Promise.all` 的 error，所有其他 promise 的结果都会被忽略。
2. `Promise.allSettled(promises)`（ES2020 新增方法）—— 等待所有 promise 都 settle 时，并以包含以下内容的对象数组的形式返回它们的结果：
    - `status`: `"fulfilled"` 或 `"rejected"`
    - `value`（如果 fulfilled）或 `reason`（如果 rejected）。
3. `Promise.race(promises)` —— 等待第一个 settle 的 promise，并将其 result/error 作为结果返回。
4. `Promise.any(promises)`（ES2021 新增方法）—— 等待第一个 fulfilled 的 promise，并将其结果作为结果返回。如果所有 promise 都 rejected，`Promise.any` 则会抛出 [`AggregateError`](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/AggregateError) 错误类型的 error 实例。
5. `Promise.resolve(value)` —— 使用给定 value 创建一个 resolved 的 promise。
6. `Promise.reject(error)` —— 使用给定 error 创建一个 rejected 的 promise。

Promisification：promise化，它指将一个接受回调的函数转换为一个返回 promise 的函数。

Promise 处理始终是异步的，因为所有 promise 行为都会通过内部的 “promise jobs” 队列，也被称为“微任务队列”（V8 术语）。
因此，`.then/catch/finally` 处理程序总是在当前代码完成后才会被调用。
如果我们需要确保一段代码在 `.then/catch/finally` 之后被执行，我们可以将它添加到链式调用的 `.then` 中。

### async-await

async/await 是以更舒适的方式使用 promise 的一种特殊语法，同时它也非常易于理解和使用。

在函数前面的 “async” 这个单词表达了一个简单的事情：即这个函数总是返回一个 promise。其他值将自动被包装在一个 resolved 的 promise 中。

关键字 `await` 让 JavaScript 引擎等待直到 promise 完成（settle）并返回结果。它只在 `async` 函数内工作

Promise 前的关键字 `await` 使 JavaScript 引擎等待该 promise settle，然后：
1. 如果有 error，就会抛出异常 —— 就像那里调用了 `throw error` 一样。
2. 否则，就返回结果。

这两个关键字一起提供了一个很好的用来编写异步代码的框架，这种代码易于阅读也易于编写。
有了 `async/await` 之后，我们就几乎不需要使用 `promise.then/catch`，但是不要忘了它们是基于 promise 的，因为有些时候（例如在最外层作用域）我们不得不使用这些方法。并且，当我们需要同时等待需要任务时，`Promise.all` 是很好用的。

### Generator

- generator 是通过 generator 函数 `function* f(…) {…}` 创建的。
- 在 generator（仅在）内部，存在 `yield` 操作。
- 外部代码和 generator 可能会通过 `next/yield` 调用交换结果。

在现代 JavaScript 中，generator 很少被使用。但有时它们会派上用场，因为函数在执行过程中与调用代码交换数据的能力是非常独特的。而且，当然，它们非常适合创建可迭代对象。

要使对象异步迭代：
1. 使用 `Symbol.asyncIterator` 取代 `Symbol.iterator`。
2. `next()` 方法应该返回一个 `promise`（带有下一个值，并且状态为 `fulfilled`）。
    - 关键字 `async` 可以实现这一点，我们可以简单地使用 `async next()`。
3. 我们应该使用 `for await (let item of iterable)` 循环来迭代这样的对象。
    - 注意关键字 `await`。

Spread 语法 `...` 无法异步工作



