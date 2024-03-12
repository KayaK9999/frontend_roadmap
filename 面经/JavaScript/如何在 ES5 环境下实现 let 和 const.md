[[代码结构]]·[[属性配置]]

本质在问 let 和 var 的区别：

- `var`声明的变量会挂到window上，而`let`和`const`不会
- `var`声明的变量存在变量提升(及可以在声明语句前被使用)，而`let`和`const`不会
- `let`和`const`声明形成块作用域，只能在块作用域里访问，不能跨块访问，也不能跨函数访问
- 同一作用域下`let`和`const`不能声明同名变量，而`var`可以
- 暂时性死区，`let`和`const`声明的变量不能在声明前被使用

```
(function(){ 
	for(var i = 0; i < 5; i ++){ 
		console.log(i) // 0 1 2 3 4 
	} 
})();
```

使用无限接近闭包的形式保证var 不会污染，因为var有函数作用域和块级作用域

const 和 let 的区别在于 const 不可修改

对象存在一个属性描述符：writable，它能够配置对象为只读，和const类似，然后把它挂载在window上

```
function _const(key, value) { 
	const desc = { 
		value, 
		writable: false 
	} 
	Object.defineProperty(window, key, desc) 
} 

_const('obj', {a: 1}) //定义obj 
obj.b = 2 //可以正常给obj的属性赋值 
obj = {} //无法赋值新对象
```


