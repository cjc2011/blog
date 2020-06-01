---
title: js链式调用及delay函数实现
date: 2018-03-13 10:40:19
tags: js
thumbnail: /images/article/1.jpg
categories: js 
---

今天在网上看到一道JavaScript的面试题，感觉涉及到的知识点还是挺多的，在此记录一番。
<!-- more -->
先看下原题目

```js
  Person("Li");
  // 输出： Hi! This is Li!
  Person("Jerry").eat("dinner").eat("supper");
  // 输出：
  // Hi This is Jerry!
  // Eat dinner~
  Person("Dan").sleep(10).eat("dinner");
  // 输出：
  // Hi! This is Dan!
  // 等待10秒..
  // Wake up after 10
  // Eat dinner~
  Person("Cjc").eat("dinner").fristsleep(5).eat("supper");
  // 输出：
  // 等待5s
  // Hi! This is Cjc!
  // Eat dinner~
  // Eat supper~
```

第一眼看过去，这几个方法的都是需要链式调用来实现，而链式调用的核心是在函数中**retrun this**

大概思路有了，下面就来实现具体功能

```js
  // 首先来创建一个Person类
  function Person(name) {
    console.log(`Hi This is ${name}`)
  }
  // 在Person类原型链上扩展eat方法
  Person.prototype.eat = function(food) {
    console.log(`Eat ${food}`)
    return this
  }
  // 在浏览器中测试 依次输出
  // Hi This is Cjc!
  // Eat dinner~
  new Person('cjc').eat('dir')

```

简易版的链式调用实现了，但是看下题目中Person前面没有new运算符，这时候想起来设计模式中的工厂模式，在实例化对象外面包裹一层function，继续来改造下

```js
  function Person(name) {
    return new Main(name)
  }
  function Main(name) {
    console.log(`Hi This is ${name}`)
  }
  Main.prototype.eat = function(food) {
    console.log(`Eat ${food}`)
    return this
  }
  // 在浏览器中测试 依次输出
  // Hi This is Cjc!
  // Eat dinner~
  // 依旧OK
  Person('cjc').eat('dir')
```

下面我们继续来看slee延迟函数的实现，延迟函数需要通过定时器来实现，但是由于setTimeout是一个异步线程，对js事件线程不熟悉的同学可以参考这两个文章视频[再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)和[可视化Event Loop](https://www.youtube.com/watch?time_continue=1&v=8aGhZQkoFbQ)，而异步线程里的所有事件都需要浏览器主线程事件执行完毕后才会去执行，这样一来sleep函数可能就不会按照预期的顺序来执行，所以这时候就需要我们自己实现一个事件队列来控制函数的调用顺序和时机，下面看具体代码实现：

```js
  function Person(name) {
    return new Main(name)
  }
  function Main(name) {
    // 事件队列存储在queue数组内
    this.queue = []
    // 初始化输出name也为一个事件 push到queue中
    this.queue.push( () => {
      console.log(`Hi This is ${name}`)
      this.next()
    })
    // 防止第一次执行this.next方法时事件队列未添加完整
    setTimeout( () => {
      this.next()
    })
  }
  Main.prototype.next = function() {
    let fn = this.queue.shift()
    if (typeof fn === 'function') {
      fn()
    }
  }
  Main.prototype.eat = function(food) {
    this.queue.push( () => {
      console.log(`Eat ${food}`)
      this.next()
    })
    return this
  }
  Main.prototype.sleep = function(time) {
    this.queue.push( () => {
      setTimeout( () => {
        console.log(`Wake up after ${time}`)
        this.next()
      }, time * 1000)
    })
    return this
  }

  // 在浏览器端测试 依次输出
  // This is cjc
  // 等待3秒
  // Wake up after 3
  Person('cjc').sleep(3).eat('dirner')

```

这段代码的增加了queue数组来存储事件列队，在每个事件函数最后添加this.next()，通过next函数依次从队列中取出函数不断的来调用，需要注意的是在Main函数中最后调用的this.next需要异步执行，否则在初始化调用next方法时事件列队为空。

这时基本的功能已经都实现了，还有一个fristsleep函数，将所有的事件都延迟执行，有了上面的铺垫这个实现起来就很简单了，我们只需在添加fristsleep函数内向queue添加的push方法改为unshift，这样就把fristsleep添加到事件列队的第一位，下面看具体代码实现

```js
  // add code
  Main.prototype.fristsleep = function(time) {
    this.queue.unshift( () => {
      setTimeout( () => {
        console.log(`Wake up after ${time}`)
        this.next()
      }, time * 1000)
    })
    return this
  }
```

以上就是本篇文章的全部内容，在这个题目中分别涉及到了工厂模式，this上下文，浏览器事件队列相关的知识点，链式调用平时的使用还是很频繁的比如Promise的then方法和catch方法，下一篇文章我们来具体的说一说Promise的实现。
