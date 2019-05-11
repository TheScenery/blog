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


## animation 使用方法
```css
animation: name duration timing-function delay iteration-count direction;
// 同样也可以每个都单独设置
animation-name // 规定需要绑定到选择器的 keyframe 名称
animation-duration // 规定完成动画所花费的时间，以秒或毫秒计，默认0，所以省略这一项往往会没有动画效果
animation-timing-function // 规定动画的速度曲线
animation-delay // 规定在动画开始之前的延迟
animation-iteration-count // 规定动画应该播放的次数,可以设置为无数次 infinite
animation-direction // 规定是否应该轮流反向播放动画， 可以设置一些反方向什么的。



// 特殊的
animation-fill-mode //动画执行前后的样式

animation-fill-mode: none //动画执行前后不改变任何样式
animation-fill-mode: forwards //保持目标动画最后一帧的样式
animation-fill-mode: backwards //保持目标动画第一帧的样式
animation-fill-mode: both //动画将会执行 forwards 和 backwards 执行的动作
```

animation-name keyframes？规定一些动画执行的关键帧，这个就可以比transition更加灵活的控制整个动画的过程
```css
@keyframes scale {
    from {
        trannsform: scale(1);
    }
    to {
        transform: scale(1.8);
    }
}


@keyframes scale {
    0% {
        trannsform: scale(1);
    }
    30% {
        transform: scale(1.8);
    }
    80% {
        transform: scale(3);
    }
}
```
可以定义执行过重的一些元素样式，既能from to指定两个节点，也可以使用百分比来详细的设置各个时间阶段的变化。

什么时候你执行动画？

    1. 元素第一次加载的时候开始执行
    2. 元素已经加载完毕，直接通过JS给起设置动画，则在刚设置完后执行
    3. 想办法触发浏览器的重新布局，比如添加删除class， 设置deplay:none --> display: flex.之类的变化。