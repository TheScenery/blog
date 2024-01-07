---
title: Flex布局整理
date: 2018-06-16 15:06:27
description: Flex布局，即为“弹性布局”，只要有两个基本的概念，容器、成员。
categories: ['充电']
---
Flex布局，即为“弹性布局”，只要有两个基本的概念，容器、成员。
![middleware-data-flow](/images/flex/flexbox.png)

使用flex布局模型，首先需要将Container设置为`display: flex`， 注意， 设定了flex之后，成员的`float、clear`和`vertical-align`属性将失效

Flex Container主要有以下属性可以设置：
1. flex-direction
2. justify-content
3. align-items
4. flex-wrap
5. align-content
6. flex-flow

flex-direction: 主要用来决定成员在主坐标轴上的排列顺序，有以下四个取值：

``` css
    row: 水平方向排列，从左边开始。 默认值
    row-reverse: 水平方向排列， 从右边开始
    column: 垂直方向排列，从上面开始
    column-reverse: 垂直方向排列， 从下面开始
```
justify-content：决定在主坐标轴上，里面的成员如何分布。可选值非常多：但是，常用的只有这几个：
```
    center: 居中对齐
    space-between: 均匀排列每个元素,首个元素放置于起点，末尾元素放置终点
    space-around: 均匀排列每个元素, 每个元素周围分配相同的空间
    space-evenly: 均匀排列每个元素, 每个元素之间的间隔相等
    flex-start: 从行首起始位置开始排列
    flex-end: 从行尾位置开始排列
```

align-items: 决定成员在副坐标轴如何分布， 可选值一样很多，常用的几个：
```
    center: 局中对齐
    stretch： 占满高度。 默认值
    flex-start： 起点对齐
    flex-end: 终点对齐
```
flex-wrap： 定义flex成员是否允许在副坐标轴上折行：
```
    nowrap: 不折行
    wrap： 自动折行，向后插行
    wrap-reverse： 自动折行，向前插行
```
align-content：决定，如果有多行时，他们之间如何对齐。类似于`justfy-content`的取值，只不过是定义的是在副坐标轴上多行之间的分布方式。

flex-flow：`flex-direction` 和 `flex-wrap` 的简写

Flex成员的属性：
1. flex-grow
2. flex-shrink
3. flex-basis
4. flex
5. align-self

flex-grow： 放大因子， 默认值为0， 既即使有剩余空间，也不放大。取值自然数

flex-shrink： 所小因子，默认值1，所以空间不够时，会缩小， 取值也为自然数

flex-basis： 指定了 flex 元素在主轴方向上的初始大小， 默认值为auto， 即原本大小

flex: 这是一个简写属性，可以同时设置flex-grow, flex-shrink与flex-basis。

align-self: 覆盖 align-items 的值, 即自己设置自己的在副坐标轴方向对齐方式。如果任何 flex 元素的侧轴方向 margin 值设置为 auto，则会忽略 align-self。
