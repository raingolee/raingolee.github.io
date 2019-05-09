---
title: JSX介绍
layout: post
tags:
 - frontend
categories:
  - 前端
  - React
---
#### 什么是jsx

jsx是javascrip的一种语法扩展，长得像HTML，但不是HTML，可能会让你想起有点像模板语言；

任何jsx的结构都可以让babel编译成javascript对象，下面举个栗子

<!-- more -->

jsx结构：

```javascript
const element = (
	<h1 className="greeting">
		Hello, world
	</h1>
);
```

javascript对象

```javascript
const element = React.createElement(
	'h1',
  	{className: 'greeting'},
  	'Hello, world!'
);
```

**所谓的 JSX 其实就是 JavaScript 对象** ，最后React会把这些对象组成一个DOM，然后render到页面指定位置

![jsx](https://raingolee.github.io/img/babel.png)



#### embeding exprssion in jsx

在jsx里面可以添加任何javscript expression，不过需要用{}括起来

```javascript
const element = (
	<h1>
		i am {2+2} years old
	</h1>
);
```

#### jsx is an Expression too

因为jsx最后都会成为javascript 对象，因为你可以在if 或者 for循环中使用jsx，还能把jsx赋值给变量，接收参数，甚至在函数中返回
