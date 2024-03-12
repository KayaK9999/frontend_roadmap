简单： https://www.ruanyifeng.com/blog/2018/06/javascript-this.html
深刻： https://segmentfault.com/a/1190000011194676


> [!NOTE] 注意函数 func 和 func() 的区别
> 一个代表本身，一个代表调用

#### this代表什么

1. this是JavaScript的关键字之一。它是 对象 自动生成的一个内部对象，只能在 对象 内部使用。随着函数使用场合的不同，this的值会发生变化。
2. **this指向什么，==完全取决于 什么地方以什么方式调用，而不是 创建时**==。（比较多人误解的地方）（它非常语义化，this在英文中的含义就是 **这，这个** ，但这其实起到了一定的误导作用，因为this并不是一成不变的，并不一定一直指向当前 **这个**）

#### this绑定规则

###### 默认绑定
```
function foo(){
    var a = 1 ;
    console.log(this.a);    // 10
}
var a = 10;
foo();
```

**这种直接使用而不带任何修饰的函数调用** ，就 **默认且只能** 应用 默认绑定。

那默认绑定到哪呢，一般是`window`上，严格模式下 是`undefined`。

###### 隐形绑定

```
function foo(){
    console.log(this.a);
}
var obj = {
    a : 10,
    foo : foo
}
foo();                // undefined

obj.foo();            // 10
```


> [!NOTE] 与默认绑定的区别就是有没有对象调用

###### 显性绑定

使用 call，apply，bind 绑定[[装饰器和转发]]·[[函数绑定]]

###### new 绑定

什么是 new？
**js中的只要用new修饰的 函数就是'构造函数'**，准确来说是 **函数的`构造调用`**，因为在js中并不存在所谓的'构造函数'。

那么用new 做到函数的`构造调用`后，js帮我们做了什么工作呢:
1. 创建一个新对象。
2. 把这个新对象的`__proto__`属性指向 原函数的`prototype`属性。(即继承原函数的原型)
3. **将这个新对象绑定到 此函数的this上** 。
4. 返回新==对象==，如果这个函数没有返回其他**对象**。

```
function foo(){
    this.a = 10;
    console.log(this);
}
// window对象
foo(); 
// 10   默认绑定
console.log(window.a); 

// foo{ a : 10 }  创建的新对象的默认名为函数名
// 然后等价于 foo { a : 10 };  var obj = foo;
var obj = new foo();      
                          
console.log(obj.a);       // 10    new绑定
```

###### this绑定优先级
new 绑定 > 显示绑定 > 隐式绑定 > 默认绑定

###### 面试题
```
var x = 10;
var obj = {
    x: 20,
    f: function(){
        console.log(this.x);        // ?
        var foo = function(){ 
            console.log(this.x);    
            }
        foo();                      // ?
    }
};
obj.f();
```

```
function foo(arg){
    this.a = arg;
    return this
};

var a = foo(1);
var b = foo(10);

console.log(a.a);    // ?
console.log(b.a);    // ?
```

```
var x = 10;
var obj = {
    x: 20,
    f: function(){ console.log(this.x); }
};
var bar = obj.f;
var obj2 = {
    x: 30,
    f: obj.f
}
obj.f();
bar();
obj2.f();
```

```
function foo() {
    getName = function () { console.log (1); };
    return this;
}
foo.getName = function () { console.log(2);};
foo.prototype.getName = function () { console.log(3);};
var getName = function () { console.log(4);};
function getName () { console.log(5);}
 
foo.getName ();                // ?
getName ();                    // ?
foo().getName ();              // ?
getName ();                    // ?
new foo.getName ();            // ?
new foo().getName ();          // ?
new new foo().getName ();      // ?
```

###### 箭头函数的this绑定
1. 箭头函数不使用我们上面介绍的四种绑定，而是**完全根据外部作用域来决定this**。(它的父级是使用我们的规则的哦)
2. 箭头函数的this绑定无法被修改

