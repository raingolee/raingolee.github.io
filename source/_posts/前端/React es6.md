---
title: React ES6新特效
layout: post
tags:
 - frontend
categories:
  - 前端
  - React
---

在React里面我们逐渐迁移了ES6的新写法，下面来介绍一下ES6中的新特性
<!-- more -->

### const && let

不再使用var去声明和赋值变量，用const和let去取代之

- const : 被 const 声明的变量不能被重新赋值或重新声明

- let: 被关键字let声明的变量可以被改变



### 箭头函数
``` javascript
//function expresstion
function () { ... }

// 箭头函数 arrow funciont expresstion
() => { ... }
```

举个简单的例子
``` javascript
cosnt students = [
  {
    name: "tom",
    hobby: "pinao",
  },
  {
    name: "jack",
    hobby: "guita",
  }
];

students.map(item => {
  return (
  	<div key=item.objectID}>
  	  <span>{item.name}</span>
  	  <span>{item.hobby}</span>
  	</div>
  )
});
```

### ES6 类(class)
熟悉面向对象编程的同学应该对这个很熟悉了，ES6终于也引入了类的概念，可以在JavaScript里面写函数式编程或者面向对象编程都可以

下面用React组件类来举个例子

``` javascript
import React, { Component } from 'react';

class App extends Component {
  contrutor() {
    ...
  };
  render() {
    ...
    return(
      ...
    );
  };
}
```
App类继承了Component，Component类则是一个基本ES6类，这个Component封装了所有React类需要事先的细节，而且保留出来的方法都是公共的接口。

这些方法中有一个方法必须被重写，他就是render()，而contrustor()这个方法则没有要求是否需要被重写，他和python类中的__init__()方法类似

### ES6结构

ES6中有一种更方便的方法来访问对象和数组的属性，叫解构

``` javascript
const user = {
  name: "roar",
  hobby: "basketball",
};

//ES5
var name = user.name;
var hobby = user.hobby;
console.log(name + ' ' + hobby);

//ES6
const { name, hobby } = user;
console.log(name + ' ' + hobby);
```

### ES6模块：Import & Export

这个和python中的import基本一样，可以重模块中导入和到处某些功能，这些功能可以是函数，类，组件，常量

文件`/home/test/es6lint/name.js`

``` javascript
const name = "roar";
const hobby = basketball";

export { name, hobby };

```

文件`/home/test/es6lint/getname.js`

``` javascript
import { name, hobby } from './name.js'

console.log(name);

//也可以全部import进来

import * as person from './name.js'
console.log(persion.name);
```



