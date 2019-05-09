---
title: rendering elements
layout: post
tags:
 - frontend
categories:
  - 前端
  - React
---

#### component

components 有点像javascipt的函数，可以接受参数（called "props")，然后返回需要展示的内容

<!-- more -->

可以用关键字function去定义一个component，在es6中还能用关键字class来定义一个component，一个组件类必须有一个render的方法，且返回一个JSX元素，必须要用一个外层的 JSX 元素把所有内容包裹起来

**functional component**

```javascript
function Human(props) {
  return <h1> this is {props.name}</h1>;
}
```

**class component**

```javascript
class Human extend React.Component {
  render() {
    return(
      <div>
      	<h1> this is {props.name} </h1>
      	<h2> i am {props.year} years old</h2>
      </div>
    )    
  }
}

ReactDOM.render(
  <Human name="raingolee" year="7" />
  document.getElementById('root')
)
```



