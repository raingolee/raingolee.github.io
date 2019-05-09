---
title: stack
layout: post
tags:
  - leetcode
categories:
  - 数据结构
  - stack
---

### 数据结构

- Stack（栈）, Queue（队列, Deque（双端队列和List（列表

  数据项之间存在先后的次序关系，新的数据项加入到数据集中时，只会加到原有的某个数据项之前或之后

- 线性结构总有两端

- 不同线性结构的关键区别在于数据项的增减方式

<!-- more -->

### 栈

- stack是一种有次序的数据项集合，在stack中，数据项的加入和移出都仅发生在同一端，一般这一端叫top，另一端则为base
- 这种次序称为LIFO,last in first out

![](https://raingolee.github.io/img/stack_illustration.png)

#### 抽象数据类型stack

抽象数据类型stack是一个有次序的数据集，我们可以为他定义如下的操作

```
Stack(): 创建一个新的空栈，其中不包含任何数据项
push(item): 将item数据项加入栈顶
pop(): 抢栈顶数据项移除，返回栈顶的数据项，栈被修改
peek(): 查看栈顶数据项，但是不移除此数据项
isEmpty(): 返回栈是否为空栈
size(): 返回栈中有多少个数据项
```

#### 用python实现Abstract Data Type stack

python的面向对象机制，可以用来实现用户自定义类型，采用Python List来实现这个数据集，将List的未端(index = -1)作为栈顶

- 将ADT Stack实现为Python的一个class
- 将ADT Stack的操作实现为class的方法

```python
class Stack:
    def __init__(self):
        self.items = []
    def isEmpty(self):
        return self.itmes == []
    def push(self, item):
        self.items.append(item)
    def pop(self):
        self.items.pop()
    def peek(self):
        return self.items[len(self.items)-1]
    def size(self):
        return len(self.items)
```

其push/pop的复杂度为O(1)

#### 栈的应用：简单括号匹配

对括号是否正确匹配的识别，是很多编译器的基础算法，那么我们以后利用stack去构造这个括号匹配识别的算法

从左到右扫扫描括号串，如果是左括号就push进stack，如果是右括号，则pop出top的那个左括号；对于右括号的视角就是匹配stack top的那个左括号，这种次序反转的识别，正好符合stack的特性

```python
def parChecker(symbolString):
    s = Stack()
    balanced = True
    index = 0
    while index < len(symbolString) and balanced:
        symbol = symbolSting[index]
        if symbol == "(":
            s.push(symbol)
        else:
            if s.isEmpty():
                balanced = False
            else:
                s.pop()
        index = index + 1

    if balance and s.isEmpty():
        return True
    else:
        return False
```

#### 栈的应用：十进制转换为二进制

十进制转换为二进制，采用的是除以2的算法，讲整数不断除以2，每次得到的余数就是由低到高的二进制位，但是展示给二进制则是从高到低，所以需要一个栈来反转次序

![](https://raingolee.github.io/img/statck_10.png)

```python
def divideBy2(decNumber):
    remstack = Stack()

    while decNumber > 0:
        #求余数
        rem = decNumber % 2
        remstack.push(rem)
        #整除
        decNumber = decNumber // 2

    binString = ""
    while not remstack.isEmpty():
        binString = binString + str(remstack.pop())
    return binString
```

十进制转换为二进制的算法，很容易可以扩展为转换到任意进制，只需要将除以2算法改为除以n算法就可以了
