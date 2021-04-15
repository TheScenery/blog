---
title: JavaScript内存泄漏
date: 2021-04-13 20:31:28
tags: JavaScript, 内存泄漏
---

JavaScript内存泄漏，通俗来讲就是JavaScript申请的变量不能被GC回收。

### 浏览器GC回收机制

标记清除或者引用计数，而现代浏览器则都为标记清除类似的机制。

标记清除顾名思义是一种分两阶段对对象进行垃圾回收的算法。

第一阶段：标记。从根结点出发遍历对象，对访问过的对象打上标记，表示该对象可达。

第二阶段：清除。对那些没有标记的对象进行回收，这样使得不能利用的空间能够重新被利用。

因此，造成内存泄漏原因无非为，意外的造成了变量从根结点一直能够访问到这些变量。

### 内存泄漏可能的原因(无非为不小心能把变量挂到根结点的方式)

1. 意外写到了根结点上的变量

    这种一般是由于手残导致的，一般不太会发生。写的时候定义变量注意都加上 var let const

    ```javascript
    //别写这种代码就行
    function() {
        a = 123;
    }

    ```
2. 全局的事件监听

    事件监听一定记得及时取消，只要监听和摘取成对出现，一般不会出现太大的问题。

3. setTimeOut，setInterval和requestAnimationFrame

    用完记得及时取消，保证成对出现
    ```javascript
    setTimeOut                clearTimeout
    setInterval               clearInterval
    requestAnimationFrame     cancelAnimationFrame

    ```

4. 闭包？并不会引起内存泄漏

    经过实际验证，现代浏览器中，必报并不会引起内脆泄漏。应该是个传说中的某个浏览器的BUG