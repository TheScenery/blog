---
title: 微前端框架之qiankun初探
date: 2022-06-25 11:14:24
tags: 微前端， qiankun
---

Techniques, strategies and recipes for building a <b>modern web app</b> with <b>multiple teams</b> that can <b>ship features independently</b>. ———— [Micro Frontends](https://micro-frontends.org/)

`微前端是可以用来构建能够让多个团队独立交付项目代码的现代web app 技术，策略以及实践方法`

![](/images/qiankun/verticals-headline.png)

微前端架构旨在解决单体应用在一个相对长的时间跨度下，由于参与的人员、团队的增多、变迁，从一个普通应用演变成一个巨石应用(Frontend Monolith)后，随之而来的应用不可维护的问题。这类问题在企业级 Web 应用中尤其常见。

![生动形象的例子](/images/qiankun/three-teams.png)

## 微前端的核心思想

```txt
Be Technology Agnostic(技术不可知主义)
Isolate Team Code(隔离团队之间的代码)
Establish Team Prefixes(建立团队自己的前缀)
Favor Native Browser Features over Custom APIs(原生浏览器标准优先于框架封装的API)
Build a Resilient Site(构建高可用的网络应用)
```

## 微前端的价值

```txt
技术栈无关
独立开发、独立部署
增量升级
独立运行时
```

## qiankun (乾坤)是什么？

`In Chinese, qian(乾) means heaven and kun(坤) earth. qiankun is the universe.`

乾坤是一个基于 [single-spa](https://github.com/single-spa/single-spa) 的微前端实现库，旨在帮助大家能更简单、无痛的构建一个生产可用微前端架构系统。

`简单`  `解耦/技术栈无关`

### 特性
```txt
基于 single-spa 封装
HTML Entry

技术栈无关
样式隔离
JS 沙箱
资源预加载
umi 插件
```


## 如何工作的

![主应用&微应用](/images/qiankun/master-slave.jpeg)

### 使用方法
主应用安装乾坤框架

```bash
yarn add qiankun # 或者 npm i qiankun -S
```

主应用中注册微应用

```javascript
import { registerMicroApps, start， loadMicroApp } from 'qiankun';

// 自动匹配
registerMicroApps([
  {
    name: 'react app', // app name registered
    entry: '//localhost:7100',
    container: '#yourContainer',
    activeRule: '/yourActiveRule',
  },
  {
    name: 'vue app',
    entry: { scripts: ['//localhost:7100/main.js'] },
    container: '#yourContainer2',
    activeRule: '/yourActiveRule2',
  },
]);

start();

// 手动加载
loadMicroApp({
  name: 'app',
  entry: '//localhost:7100',
  container: '#yourContainer',
});
```

微应用导出生命周期钩子

```javascript
/**
 * bootstrap 只会在微应用初始化的时候调用一次，下次微应用重新进入时会直接调用 mount 钩子，不会再重复触发 bootstrap。
 * 通常我们可以在这里做一些全局变量的初始化，比如不会在 unmount 阶段被销毁的应用级别的缓存等。
 */
export async function bootstrap() {
  console.log('react app bootstraped');
}

/**
 * 应用每次进入都会调用 mount 方法，通常我们在这里触发应用的渲染方法
 */
export async function mount(props) {
  ReactDOM.render(<App />, props.container ? props.container.querySelector('#root') : document.getElementById('root'));
}

/**
 * 应用每次 切出/卸载 会调用的方法，通常在这里我们会卸载微应用的应用实例
 */
export async function unmount(props) {
  ReactDOM.unmountComponentAtNode(
    props.container ? props.container.querySelector('#root') : document.getElementById('root'),
  );
}

/**
 * 可选生命周期钩子，仅使用 loadMicroApp 方式加载微应用时生效
 */
export async function update(props) {
  console.log('update props', props);
}
```
微应用暴露出额外的一些信息，修改配置

```javascript
const packageName = require('./package.json').name;

module.exports = {
  output: {
    library: `${packageName}-[name]`,
    libraryTarget: 'umd',
    jsonpFunction: `webpackJsonp_${packageName}`,
  },
};
```

### 实现原理


## 一些限制和不足