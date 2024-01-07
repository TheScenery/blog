---
title: '奇奇怪怪的JavaScript'
date: 2024-01-07T10:21:58+08:00
description: 《你不知道的JavaScript》读书笔记
categories: ['充电']
tags: ['JavaScript']
draft: true
---

# Yoy Don't Know JavaScript 

1. typeof用来判断类型，但是 `typeof null === object`

    目前JavaScript之中共有七种内置的类型：`null, undefined, boolean, number, string, object, symbol`, 我们可以使用typeof来判断一个值的类型，其会返回一个字符串。

    ```javascript
    typeof undefined === 'undefined' //true
    typeof true === 'boolean' //true
    typeof 42 === 'number' //true
    typeof '42' === 'string' //true
    typeof { count: 42 } === 'object' //true
    typeof Symbol() === 'symbol' //true

    //奇怪的地方来临了
    typeof null === 'null' //false ???
    typeof null === 'object' //true
    ```
    事实上这是一个由来已久的BUG，可能永远也不会被修复了，涉及到太多的WEB系统，“修复”了它可能会使得很多系统无法正常工作。
    ```javascript
    let a;
    (!a && typeof a === 'object') //That's null.
    ```
2. 数字的语法。`42.toFixed(3) //Uncaught SyntaxError: Invalid or unexpected token`
3. 强制类型转换
    ```javascript
    [] == ![] // -> true

    (![] + [])[+[]] +
    (![] + [])[+!+[]] +
    ([![]] + [][[]])[+!+[] + [+[]]] +
    (![] + [])[!+[] + !+[]];
    // -> 'fail'
    ```
