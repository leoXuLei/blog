# this

- this不指向函数本身也不指向函数的词法作用域
- this是在函数被调用时发生的绑定，也就是说，函数未调用之前是没有this的，它指向谁完全取决于函数在哪里被调用
- this具有语法作用域的特征
- 在函数被调用时，会创建一个活动记录(执行上下文)，这个记录会包含函数在哪里调用(调用栈)，函数的调用方式，传入的参数等信息，this是这个记录的一个属性，会在函数的执行过程中用到，上下文不存在，this也就不存在了

### this的绑定规则
**1.默认绑定**
在全局作用域调用函数，函数的this是window(在严格模式为undefined)
```js
function foo() {
  console.log(this)
}
function bar() {
  foo()
}
bar() // 输出为window
```

**2.隐式绑定**
当函数引用具有上下文对象时，隐式绑定规则会把this绑定到这个上下文对象上
```js
function foo() {
  console.log(this) // obj对象
  console.log(this.a) // 2
}
const obj = {
  a: 2,
  foo: foo
}
obj.foo() // 输出为obj对象，即{a:2,foo:foo}
```

**3.显示绑定**
通过`call(...)` `apply(... )` `bind(...)`方法

`apply()`方法第一个参数是**一个对象**，在调用函数时将this绑定到这个对象上，因为直接指定this的绑定对象，称之为显示绑定
```js
function foo() {
  console.log(this) // obj对象
}
const obj = {
  a: 2
}
foo.call(obj) // foo函数执行
```
`apply()`方法第二个参数是一个**数组**，数组里面是调用函数时要传递的参数
```js
function foo(arg1, arg2, arg3) { // arg1,arg2,arg3对应的分别是**参数数组**的第一项，第二项，第三项
  console.log(arguments) // [1,2,{name: 'xbl'}] arguments为伪数组
  console.log(arg1) // 1
  console.log(arg2) // 2
  console.log(arg3) // {name: 'xbl'}
}
// 使用ES6拓展运算符展开参数更方便
function foo(...args) {
  console.log(...args)
}
const obj = {}
foo.apply(obj, [1,2,{name: 'xbl'}])
// foo.apply(obj, 1) 传递非数组会报错
```

`call()`方法与`apply()`不同的是，可以有2-n个参数，第2-n个参数是调用函数要传的参数
```js
function foo(arg1, arg2, arg3) {
  console.log(arguments) // [1,2,{name: 'xbl'}] arguments为伪数组
  console.log(arg1) // 1
  console.log(arg2) // 2
  console.log(arg3) // {name: 'xbl'}
}
const obj = {}
foo.call(obj, 1, 2, {name: 'xbl'})
```

`bind()`传入参数与`call()`相同，不过bind返回的是一个函数
```js
function foo(a,b) {
  console.log('a:', a, 'b:', b) // a: 2 b: 3
}
// 使用bind进行柯里化？？
// 函数柯里化是指 函数对传入的参数做处理，每处理一个参数，返回一个函数，是函数式编程的重要组成部分
const bar = foo.bind({}, 2,3)
bar()
```
ES6实现bind方法 ??
```js
Function.prototype.bind = function(...rest1) {
  const self = this
  const context = rest1.shift()
  return function(...rest2) {
    return self.apply(context, [...rest1, ...rest2])
  }
}
```

把 `null` 或者 `undefined` 作为this绑定的对象传入 `call,apply,bind`, 这些值 在调用时会被忽略，实际应用的仍然是**默认规则**
使用null可能会产生副作用，可以传空对象{}
js中创建空对象最简单的方法时 Object.create(null), 这个和{}很像，但是不会创建Object.prototype这个委托，比{}更空，即没有原型链
```js
function foo(a,b) {
  console.log('a:', a, 'b:', b)
}
const empty = Object.create(null)
foo.apply(empty, [2,3])  // a:2 b:3
```

**4.new绑定**
- js中，构造函数只是使用new操作符时被调用的普通函数
- 内置对象如 Number,Object等等在内的所有构造函数都可以用new调用，这种调用方式称为构造函数调用
- 实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”

*使用new调用函数时，会自动执行以下操作* 
- 创建一个新对象
- 新对象的prototype指向构造函数
- 新对象赋给当前this
- 执行构造函数
- 如果函数没有返回其他对象，new表达式中的函数会自动返回这个新对象

优先级：new > apply/call/bind > 隐式绑定 > 默认绑定

## this词法
- ES6新增一种特殊函数类型：箭头函数，箭头函数无法使用上述四条规则，而是根据外层(函数或者全局)作用域(词法作用域)来决定this
- 箭头函数的this无法被修改
- 箭头函数没有构造函数constructor,不可以使用new 调用
```js
function foo() {
  // 返回一个匿名箭头函数，该函数this取决于foo的this，foo的this仍然遵照语法作用域
  return a => {
    console.log(this.a)
  }
}
const obj1 = {
  a: 2
}
const obj2 = {
  a: 3
}
const bar = foo.call(obj1) // foo的this是obj1
bar() // 输出2，由于是箭头函数，所以在window下面执行箭头函数，但实际还是要看bar的父函数foo的this指向
bar.call(obj2)  // 输出2， bar的this是obj2，看似箭头函数的this已经指向了obj2,其实不是的，箭头函数的this无法被语法作用域改变，只会为词法作用域的上一层函数改变
const baz = foo.call(obj2)
baz() // 输出3
```

回调函数里面的this
```js
function foo(callback) {
  return callback()
}
```

多个嵌套函数  
```js
function foo() {
  console.log(`foo内部`,this) // {outObj: 1}
  function bar() {
    console.log(`bar内部`,this) // {innerObj: 2, bar: f}
    function baz() {
      console.log(`baz1`, this) // window
      function baz1() {
        console.log(`baz2`, this) // window
      }
      baz1()
    }
    baz()
  }
  const innerObj = {inner: 2, bar}
  return innerObj.bar() // 执行bar函数，return可以不用加
}
const outObj = {outObj: 1}
foo.call(outObj)
```
可以看出，每个函数的this都是独立的，无法继承自父函数，默认规则绑定的是window

定时器
```js
function foo() {
  const self = this
  window.setTimeout(function() { // 此函数是在setTimeout函数内部执行的,setTimieout执行的时候，应用了隐式绑定，
    console.log(self) // obj
    console.log(this) // window 隐式绑定
  }, 1000)
  window.setTimeout(() => { // window调用了setTimeout函数，所以setTimeout优先命中的是隐式绑定，this是window
    console.log(this) // obj ?? 
  }, 2000)
}
const obj = {a:1}
foo.call(obj) // foo this 是 obj
```

这里再看另外一个回调函数的例子  
```js
function foo() {
  console.log(`foo this是`, this) // {a:1}
  const innerObj = {b: 2, bar}
  function bar(callback) { // 定义一个高阶函数，接收一个回调函数
    console.log(`bar的this是`, this) // 两个都是{bar: 2, bar}
    callback()
  }
  innerObj.bar(function() { // 执行bar函数，bar隐式绑定this, bar函数内部有个函数打印this
    console.log(`bar 内部 非箭头函数`, this) // window?? 所以callback是应用了默认绑定？？
  }) 
  innerObj.bar(() => { // 执行bar函数，bar隐式绑定this, bar函数内部有个箭头函数内部打印this
    console.log(`bar 内部 箭头函数`, this) // {a:1} ?? callback在bar内部，bar外部的this是foo内部的this，即{a:1}
  })
}
const outObj = {a:1}
foo.call(outObj)
```

对象中属性值为箭头函数
```js
var a = 'hello'
const obj = {
  a: 'world',
  b: this,
  foo: () => {
    console.log(this.a)
  }
}
console.log(obj.b)  // window，，对象是没有上下文环境也就是this的，，只能继承
obj.foo() // hello, 隐式绑定, foo this是obj外面的this，即window
```

this的一个例子
```js
var a = 20 
// 用const a = 20 无法得到想要的输出,因为此时a没有被绑定到window对象上
const obj = {
  a: 40,
  // 应用默认绑定，this是window
  foo: () => {
    console.log(this.a) // 词法作用域，obj外面的this, this是window对象，输出20

    function func() {
      this.a = 60  // 语法作用域，this现在不确定
      console.log(this.a) // this不确定，但是this.a确定，是60
    }

    func.prototype.a = 50  // func的prototype的a值是50
    return func
  }
} 
const bar = obj.foo() // 执行foo函数，并返回func函数  20
bar() // 执行bar函数  60
new bar() // 60
```


### 参考
- [你不知道的javascript上第二部分this和对象原型](https://github.com/yygmind/Reading-Notes/blob/master/%E4%BD%A0%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84JavaScript%E4%B8%8A%E5%8D%B7.md)
- [You Don't Know JS: Scope & Closures](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20&%20closures/README.md#you-dont-know-js-scope--closures)
- [慕课网JavaScript 设计模式](https://www.imooc.com/read/38)