[[Promise，async-await，Generator]]
Promise 解决了什么问题：在传统的异步编程中，如果异步之间存在依赖关系，我们就需要通过层层嵌套回调来满足这种依赖，如果嵌套层数过多，可读性和可维护性都变得很差，产生所谓“回调地狱”，而Promise将回调嵌套改为链式调用，增加可读性和可维护性。

### 1. 实现最简单的Promise使用：

```
class MyPromise {
  // 构造方法接收一个回调
  constructor(executor) {
	// 这里队列很重要
    this._resolveQueue = []    // then收集的执行成功的回调队列
    this._rejectQueue = []     // then收集的执行失败的回调队列

    // 由于resolve/reject是在executor内部被调用, 
    // 因此需要使用箭头函数固定this指向, 否则找不到this._resolveQueue
    let _resolve = (val) => {
      // 从成功队列里取出回调依次执行
      while(this._resolveQueue.length) {
        const callback = this._resolveQueue.shift()
        callback(val)
      }
    }
    // 实现同resolve
    let _reject = (val) => {
      while(this._rejectQueue.length) {
        const callback = this._rejectQueue.shift()
        callback(val)
      }
    }
    // new Promise()时立即执行executor,并传入resolve和reject
    executor(_resolve, _reject)
  }

  // then方法,接收一个成功的回调和一个失败的回调，并push进对应队列
  then(resolveFn, rejectFn) {
    this._resolveQueue.push(resolveFn)
    this._rejectQueue.push(rejectFn)
  }
}
```

### 2. Promise A+规范

ES6的Promise实现需要遵循[Promise/A+规范](https://link.juejin.cn/?target=https%3A%2F%2Fpromisesaplus.com%2F "https://promisesaplus.com/")是规范对Promise的状态控制做了要求。Promise/A+的规范比较长，这里只总结两条核心规则：
1. Promise本质是一个状态机，且状态只能为以下三种：`Pending（等待态）`、`Fulfilled（执行态）`、`Rejected（拒绝态）`，状态的变更是单向的，只能从Pending -> Fulfilled 或 Pending -> Rejected，状态变更不可逆
2. `then方法`接收两个可选参数，分别对应状态改变时触发的回调。then方法返回一个promise。then 方法可以被同一个 promise 调用多次。

```
//Promise/A+规范的三种状态
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  // 构造方法接收一个回调
  constructor(executor) {
    this._status = PENDING     // Promise状态
    this._resolveQueue = []    // 成功队列, resolve时触发
    this._rejectQueue = []     // 失败队列, reject时触发

    // 由于resolve/reject是在executor内部被调用, 
    // 因此需要使用箭头函数固定this指向, 否则找不到this._resolveQueue
    let _resolve = (val) => {
	  // 对应规范中的"状态只能由pending到fulfilled或rejected"
      if(this._status !== PENDING) return   
      // 变更状态
      this._status = FULFILLED              

      // 这里之所以使用一个队列来储存回调,
      // 是为了实现规范要求的 "then 方法可以被同一个 promise 调用多次"
      // 如果使用一个变量而非队列来储存回调,那么即使多次p1.then()也只会执行一次回调
      while(this._resolveQueue.length) {    
        const callback = this._resolveQueue.shift()
        callback(val)
      }
    }
    // 实现同resolve
    let _reject = (val) => {
      // 对应规范中的"状态只能由pending到fulfilled或rejected"
      if(this._status !== PENDING) return   
      this._status = REJECTED               // 变更状态
      while(this._rejectQueue.length) {
        const callback = this._rejectQueue.shift()
        callback(val)
      }
    }
    // new Promise()时立即执行executor,并传入resolve和reject
    executor(_resolve, _reject)
  }

  // then方法,接收一个成功的回调和一个失败的回调
  then(resolveFn, rejectFn) {
    this._resolveQueue.push(resolveFn)
    this._rejectQueue.push(rejectFn)
  }
}
```

### 3. then 的链式调用

实现.then的链式调用需要一下条件：
1. 显然`.then()`需要返回一个Promise，这样才能找到then方法，所以我们会把then方法的返回值包装成Promise。
2. `.then()`的回调需要拿到上一个`.then()`的返回值
3. `.then()`的回调需要顺序执行，以上面这段代码为例，虽然中间return了一个Promise，但执行顺序仍要保证是1->2->3。我们要等待当前Promise状态变更后，再执行下一个then收集的回调，这就要求我们对then的返回值分类讨论

```
// then方法
then(resolveFn, rejectFn) {
  //return一个新的promise
  return new MyPromise((resolve, reject) => {
    //把resolveFn重新包装一下,再push进resolve执行队列,这是为了能够获取回调的返回值进行分类讨论
    const fulfilledFn = value => {
      try {
        //执行第一个(当前的)Promise的成功回调,并获取返回值
        let x = resolveFn(value)
        //分类讨论返回值,如果是Promise,那么等待Promise状态变更,否则直接resolve
        //这里resolve之后，就能被下一个.then()的回调获取到返回值，从而实现链式调用
        x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
      } catch (error) {
        reject(error)
      }
    }
    //把后续then收集的依赖都push进当前Promise的成功回调队列中(_rejectQueue), 这是为了保证顺序调用
    this._resolveQueue.push(fulfilledFn)

    //reject同理
    const rejectedFn  = error => {
      try {
        let x = rejectFn(error)
        x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
      } catch (error) {
        reject(error)
      }
    }
    this._rejectQueue.push(rejectedFn)
  })
}
```

### 4. 值穿透 && 状态以变更的情况

仍然存在以下细节：
- **值穿透**：根据规范，如果 then() 接收的参数不是function，那么我们应该忽略它。如果没有忽略，当then()回调不为function时将会抛出异常，导致链式调用中断
- **处理状态为resolve/reject的情况**：其实我们上边 then() 的写法是对应状态为`padding`的情况，但是有些时候，resolve/reject 在 then() 之前就被执行（比如`Promise.resolve().then()`），如果这个时候还把then()回调push进resolve/reject的执行队列里，那么回调将不会被执行，因此对于状态已经变为`fulfilled`或`rejected`的情况，我们直接执行then回调：

```
// then方法,接收一个成功的回调和一个失败的回调
  then(resolveFn, rejectFn) {
    // 根据规范，如果then的参数不是function，则我们需要忽略它, 让链式调用继续往下执行
    typeof resolveFn !== 'function' ? resolveFn = value => value : null
    typeof rejectFn !== 'function' ? rejectFn = reason => {
      throw new Error(reason instanceof Error? reason.message:reason);
    } : null
  
    // return一个新的promise
    return new MyPromise((resolve, reject) => {
      // 把resolveFn重新包装一下,再push进resolve执行队列,这是为了能够获取回调的返回值进行分类讨论
      const fulfilledFn = value => {
        try {
          // 执行第一个(当前的)Promise的成功回调,并获取返回值
          let x = resolveFn(value)
          // 分类讨论返回值,如果是Promise,那么等待Promise状态变更,否则直接resolve
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      // reject同理
      const rejectedFn  = error => {
        try {
          let x = rejectFn(error)
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      switch (this._status) {
        // 当状态为pending时,把then回调push进resolve/reject执行队列,等待执行
        case PENDING:
          this._resolveQueue.push(fulfilledFn)
          this._rejectQueue.push(rejectedFn)
          break;
        // 当状态已经变为resolve/reject时,直接执行then回调
        case FULFILLED:
          fulfilledFn(this._value)    // this._value是上一个then回调return的值(见完整版代码)
          break;
        case REJECTED:
          rejectedFn(this._value)
          break;
      }
    })
  }
```

### 5. 兼容同步任务

完成了then的链式调用以后，我们再处理一个前边的细节，然后放出完整代码。上文我们说过，Promise的执行顺序是`new Promise -> then()收集回调 -> resolve/reject执行回调`，这一顺序是建立在**executor是异步任务**的前提上的，如果executor是一个同步任务，那么顺序就会变成`new Promise -> resolve/reject执行回调 -> then()收集回调`，resolve的执行跑到then之前去了，为了兼容这种情况，我们给`resolve/reject`执行回调的操作包一个setTimeout，让它异步执行。

```
//Promise/A+规定的三种状态
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  // 构造方法接收一个回调
  constructor(executor) {
    this._status = PENDING     // Promise状态
    this._value = undefined    // 储存then回调return的值
    this._resolveQueue = []    // 成功队列, resolve时触发
    this._rejectQueue = []     // 失败队列, reject时触发

    // 由于resolve/reject是在executor内部被调用, 因此需要使用箭头函数固定this指向, 否则找不到this._resolveQueue
    let _resolve = (val) => {
      //把resolve执行回调的操作封装成一个函数,放进setTimeout里,以兼容executor是同步代码的情况
      const run = () => {
        if(this._status !== PENDING) return   // 对应规范中的"状态只能由pending到fulfilled或rejected"
        this._status = FULFILLED              // 变更状态
        this._value = val                     // 储存当前value

        // 这里之所以使用一个队列来储存回调,是为了实现规范要求的 "then 方法可以被同一个 promise 调用多次"
        // 如果使用一个变量而非队列来储存回调,那么即使多次p1.then()也只会执行一次回调
        while(this._resolveQueue.length) {    
          const callback = this._resolveQueue.shift()
          callback(val)
        }
      }
      setTimeout(run)
    }
    // 实现同resolve
    let _reject = (val) => {
      const run = () => {
        if(this._status !== PENDING) return   // 对应规范中的"状态只能由pending到fulfilled或rejected"
        this._status = REJECTED               // 变更状态
        this._value = val                     // 储存当前value
        while(this._rejectQueue.length) {
          const callback = this._rejectQueue.shift()
          callback(val)
        }
      }
      setTimeout(run)
    }
    // new Promise()时立即执行executor,并传入resolve和reject
    executor(_resolve, _reject)
  }

  // then方法,接收一个成功的回调和一个失败的回调
  then(resolveFn, rejectFn) {
    // 根据规范，如果then的参数不是function，则我们需要忽略它, 让链式调用继续往下执行
    typeof resolveFn !== 'function' ? resolveFn = value => value : null
    typeof rejectFn !== 'function' ? rejectFn = reason => {
      throw new Error(reason instanceof Error? reason.message:reason);
    } : null
  
    // return一个新的promise
    return new MyPromise((resolve, reject) => {
      // 把resolveFn重新包装一下,再push进resolve执行队列,这是为了能够获取回调的返回值进行分类讨论
      const fulfilledFn = value => {
        try {
          // 执行第一个(当前的)Promise的成功回调,并获取返回值
          let x = resolveFn(value)
          // 分类讨论返回值,如果是Promise,那么等待Promise状态变更,否则直接resolve
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      // reject同理
      const rejectedFn  = error => {
        try {
          let x = rejectFn(error)
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      switch (this._status) {
        // 当状态为pending时,把then回调push进resolve/reject执行队列,等待执行
        case PENDING:
          this._resolveQueue.push(fulfilledFn)
          this._rejectQueue.push(rejectedFn)
          break;
        // 当状态已经变为resolve/reject时,直接执行then回调
        case FULFILLED:
          fulfilledFn(this._value)    // this._value是上一个then回调return的值(见完整版代码)
          break;
        case REJECTED:
          rejectedFn(this._value)
          break;
      }
    })
  }
}
```

### 6. 剩下方法：

#### Promise.prototype.catch()
`catch()方法`返回一个Promise，并且处理拒绝的情况。它的行为与调用Promise.prototype.then(undefined, onRejected) 相同。

```
//catch方法其实就是执行一下then的第二个回调
catch(rejectFn) {
  return this.then(undefined, rejectFn)
}

```

#### Promise.prototype.finally()
`finally()方法`返回一个Promise。在promise结束时，无论结果是fulfilled或者是rejected，都会执行指定的回调函数。在finally之后，还可以继续then。并且会将值原封不动的传递给后面的then

```
//finally方法
finally(callback) {
  return this.then(
    // MyPromise.resolve执行回调,并在then中return结果传递给后面的Promise
    value => MyPromise.resolve(callback()).then(() => value),       
    // reject同理      
    reason => MyPromise.resolve(callback()).then(() => { throw reason })  
  )
}

```

#### Promise.resolve()
`Promise.resolve(value)`方法返回一个以给定值解析后的Promise 对象。如果该值为promise，返回这个promise；如果这个值是thenable（即带有"then" 方法)），返回的promise会“跟随”这个thenable的对象，采用它的最终状态；否则返回的promise将以此值完成。此函数将类promise对象的多层嵌套展平。

```
//静态的resolve方法
static resolve(value) {
  // 根据规范, 如果参数是Promise实例, 直接return这个实例
  if(value instanceof MyPromise) return value 
  return new MyPromise(resolve => resolve(value))
}
```

#### Promise.reject()
`Promise.reject()`方法返回一个带有拒绝原因的Promise对象。
```
//静态的reject方法
static reject(reason) {
  return new MyPromise((resolve, reject) => reject(reason))
}
```

#### Promise.all()
`Promise.all(iterable)`方法返回一个 Promise 实例，此实例在 iterable 参数内所有的 promise 都“完成（resolved）”或参数中不包含 promise 时回调完成（resolve）；如果参数中 promise 有一个失败（rejected），此实例回调失败（reject），失败原因的是第一个失败 promise 的结果。

# ...看不动了

