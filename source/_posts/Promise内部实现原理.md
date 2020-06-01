---
title: Promise内部实现原理
top: 0
date: 2018-04-29 10:07:01
thumbnail: /images/article/2.jpg
tags: js
categories: js
---
>Promise 是异步编程的一种解决方案：
从语法上讲，promise是一个对象，从它可以获取异步操作的消息；从本意上讲，它是承诺，承诺它过一段时间会给你一个结果。
promise有三种状态：pending(等待态)，fulfiled(成功态)，rejected(失败态)；状态一旦改变，就不会再变。创造promise实例后，它会立即执行。

<!-- more -->
```js

  const PENDING = 'pending'
  const FULFILLED = 'fulfilled'
  const REJECTED = 'rejected'

  function Promise_(fn) {
    let that = this
    that.status = PENDING
    that.value = null
    that.reason = null
    that.fullfilledCallback = []
    that.rejectedCallback = []

    function resolve(value) {
      if(value instanceof Promise) {
        return value.then(resolve, reject)
      }
      setTimeout( () => {
        if(that.status === PENDING) {
          that.status = PENDING
          that.value = value
          that.fullfilledCallback.forEach( (cb) => {
            cb(that.value)
          })
        }
      })
    }

    function rejected(error) {
      if (that.status === PENDING) {
        that.status = REJECTED
        that.reason = error
        that.rejectedCallback.forEach( cb => {
          cb(that.reason)
        })
      }
    }

    try {
      fn(resolve, rejected)
    } catch (error) {
      rejected(error)
    }

  }

  Promise_.prototype.then = function(fulfilled, rejected) {
    typeof fulfilled === 'function' && that.fullfilledCallback.psuh(fulfilled)
    typeof rejected === 'function' && that.rejectedCallback.psuh(rejected)
    return this
  }

```