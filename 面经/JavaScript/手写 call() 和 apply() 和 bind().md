[[装饰器和转发]]·[[symbol]]·[[函数绑定]]
##### 手写call
`call() 方法`使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数

语法：`function.call(thisArg, arg1, arg2, ...)`

`call()`的原理比较简单，由于函数的this指向它的直接调用者，我们变更调用者即完成this指向的变更：

```
// 将一个自定义的方法 ownCall 添加到 Function 类型的原型对象
// Function.prototype 上。这意味着所有函数对象都可以调用 ownCall 方法。
Function.prototype.ownCall = function(context, ...args) {
  context = (typeof context === 'object' ? context : window)
  // 使用 Symbol() 函数创建一个唯一的标识符，并将其赋值给 key 变量。
  // 这个标识符将用作临时属性的键，以避免与上下文对象的现有属性冲突。
  const key = Symbol()
  // 这里的this为需要执行的方法
  // 将方法作为一个方法属性添加进对象中，然后调用它就完成了这个对象的方法调用
  context[key] = this
  // 方法执行
  const result = context[key](...args)
  delete context[key]
  return result
}

```

---
##### 手写apply()

apply() 方法调用一个具有给定this值的函数，以及作为一个==类数组对象==提供的参数。
语法：func.apply(thisArg, [argsArray])

```
Function.prototype.myApply = function(thisArg, args) {
    const fn = Symbol('fn')        
    thisArg = thisArg || window    
    thisArg[fn] = this
    // 虽然apply()接收的是一个数组，但在调用原函数时，依然要展开参数数组。
    // 可以对照原生apply()，原函数接收到展开的参数数组
    const result = thisArg[fn](...args)  
    delete thisArg[fn]
    return result
}
```

---
##### 手写bind()

`bind()` 方法创建一个新的函数，在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

```
Function.prototype.myBind = function (thisArg, ...args) {
    var self = this
    // new优先级
    var fbound = function () {
	    // 这一行代码使用 apply 方法调用原始函数 self。
	    // 根据调用时的上下文，如果绑定函数 fbound 是通过 new 运算符调用的（即 this instanceof self 为真），则将绑定函数的 this 设置为新创建的对象，否则将其设置为传入的 thisArg。
        self.apply(this instanceof self ? this : thisArg, args.concat(Array.prototype.slice.call(arguments)))
    }
    // 继承原型上的属性和方法
    fbound.prototype = Object.create(self.prototype);

    return fbound;
}

//测试
const obj = { name: '写代码像蔡徐抻' }
function foo() {
    console.log(this.name)
    console.log(arguments)
}

foo.myBind(obj, 'a', 'b', 'c')()    //输出写代码像蔡徐抻 ['a', 'b', 'c']
```
