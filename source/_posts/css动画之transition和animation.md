---
title: css动画之transition和animation
date: 2019-05-07 22:00:03
tags:
---
transition和animation是两个在css中主要用来处理动画的属性，今天用到了，将使用过程中的一些疑惑和问题记录在这里。

## transition 使用方法
```css
transition: property duration timing-function delay;
/*
    property 过渡效果变化的属性，即这个属性的变化是一个过度效果
    duration 这个过度效果持续的时间
    timing-function 变化速度的时间函数
    delay 延迟多久之后这个效果才开始发生变化
*/

// 还可以分开为四个属性单独设置
transition-property
transition-duration // 默认值0
transition-timing-function // 默认值 ease
/* 
    一般都会用一些内置的时间函数
    transition-timing-function: ease
    transition-timing-function: ease-in
    transition-timing-function: ease-out
    transition-timing-function: ease-in-out
    transition-timing-function: linear
    transition-timing-function: cubic-bezier(0.1, 0.7, 1.0, 0.1)
    transition-timing-function: step-start
    transition-timing-function: step-end
    transition-timing-function: steps(4, end)

    transition-timing-function: ease, step-start, cubic-bezier(0.1, 0.7, 1.0, 0.1)

    transition-timing-function: inherit 
*/
transition-delay // 默认值0
```

    一些小问题
    1. 那么问题来了，什么时候会触发这个效果呢？  
    设置的property的发生变化则触发这个效果。
    2. 发生的变化必须是确定的状态，才会触发。 
    比如height变为fitContent 则不会触发。遇到这种情况，可以是一个max-height来进行代替，将max-height设置为一个fitContent永远达不到的值，就可以用来模模拟自适应高度的变化了。

[Height Demo](https://jsfiddle.net/TheScenery/fmwqy4pr/24/)


