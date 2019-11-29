# 赋值，浅拷贝，深拷贝

## 1. 赋值

赋值发生在栈存储结构

```js
let a = 1
let b = a
```

以上就是一个简单的赋值，`let a = 1`表示在栈内开辟一个新空间存储变量a，栈内的值是1

`let b = a`表示在栈内开辟一个新空间存储b，栈内的值与a相同，为1


## 2. 浅拷贝

浅拷贝比较快捷的方式是使用 `Object.assign()` 或者 展开运算符 `...`

## 3. 深拷贝

深拷贝比较快捷的方式是借助 `JSON.stringify()` 以及 `JSON.parse()`

缺点是函数类型，Symbol类型等无法正确的拷贝

## 参考