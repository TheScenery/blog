---
title: 微前端框架之qiankun初探
date: 2022-06-25 11:14:24
description: 微前端是可以用来构建能够让多个团队独立交付项目代码的现代web app 技术，策略以及实践方法
categories: ['充电']
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
import { registerMicroApps, start, loadMicroApp } from 'qiankun';

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

#### 加载方式——HTML Entry

qiankun使用独立的处理库[import-html-entry](https://github.com/kuitos/import-html-entry#readme)


```typescript
export async function loadApp<T extends ObjectType>(
  app: LoadableApp<T>,
  configuration: FrameworkConfiguration = {},
  lifeCycles?: FrameworkLifeCycles<T>,
): Promise<ParcelConfigObjectGetter> {
  const { entry, name: appName } = app;
  const appInstanceId = genAppInstanceIdByName(appName);

  const markName = `[qiankun] App ${appInstanceId} Loading`;
  if (process.env.NODE_ENV === 'development') {
    performanceMark(markName);
  }

  const {
    singular = false,
    sandbox = true,
    excludeAssetFilter,
    globalContext = window,
    ...importEntryOpts
  } = configuration;

  // get the entry html content and script executor
  /*
  1. template 处理后的html
  2. execScripts 执行脚本
  3. assetPublicPath 公共路径
  */
  const { template, execScripts, assetPublicPath } = await importEntry(entry, importEntryOpts);
  ///...
}

```

#### JS沙箱实现方式

qiankun中采取了两种方式来支持JS沙箱

1. 不支持Proxy的，则简单的讲window备份一个，使用完后恢复
2. 支持Proxy的，使用Proxy后的window作为全局对象，get从window上获取，set不往window上设置

```typescript
export function createSandboxContainer(
  appName: string,
  elementGetter: () => HTMLElement | ShadowRoot,
  scopedCSS: boolean,
  useLooseSandbox?: boolean,
  excludeAssetFilter?: (url: string) => boolean,
  globalContext?: typeof window,
) {
  let sandbox: SandBox;
  if (window.Proxy) {
    sandbox = useLooseSandbox ? new LegacySandbox(appName, globalContext) : new ProxySandbox(appName, globalContext);
  } else {
    sandbox = new SnapshotSandbox(appName);
  }
  //...
}

```

通过Snapshot方式实现沙箱

```typescript
export default class SnapshotSandbox implements SandBox {
  proxy: WindowProxy;

  name: string;

  type: SandBoxType;

  sandboxRunning = true;

  private windowSnapshot!: Window;

  private modifyPropsMap: Record<any, any> = {};

  constructor(name: string) {
    this.name = name;
    this.proxy = window;
    this.type = SandBoxType.Snapshot;
  }

  active() {
    // 记录当前快照
    this.windowSnapshot = {} as Window;
    iter(window, (prop) => {
      this.windowSnapshot[prop] = window[prop];
    });

    // 恢复之前的变更
    Object.keys(this.modifyPropsMap).forEach((p: any) => {
      window[p] = this.modifyPropsMap[p];
    });

    this.sandboxRunning = true;
  }

  inactive() {
    this.modifyPropsMap = {};

    iter(window, (prop) => {
      if (window[prop] !== this.windowSnapshot[prop]) {
        // 记录变更，恢复环境
        this.modifyPropsMap[prop] = window[prop];
        window[prop] = this.windowSnapshot[prop];
      }
    });

    if (process.env.NODE_ENV === 'development') {
      console.info(`[qiankun:sandbox] ${this.name} origin window restore...`, Object.keys(this.modifyPropsMap));
    }

    this.sandboxRunning = false;
  }
}

```
使用snapshot创建沙箱环境的基本原理为，使用之前对当前上下文做快照，使用完成之后再从快照恢复上下文。因此，这种方式创建的沙箱其实在生效的过程中，所有对于上下文的操作其实都是在window上进行的，也正是这种原理，决定了使用这种方式创建的沙箱，同时间只能存在一个激活的，不能同时开启，映射到微前端中也就意味着同时只能激活一个微应用。

通过Proxy实现JS沙箱

```TYPESCRIPT
export default class ProxySandbox implements SandBox {
  //...
  const proxy = new Proxy(fakeWindow, {
      // set时给fakeWindow设置
      set: (target: FakeWindow, p: PropertyKey, value: any): boolean => {
        if (this.sandboxRunning) {
          this.registerRunningApp(name, proxy);
          // We must kept its description while the property existed in globalContext before
          if (!target.hasOwnProperty(p) && globalContext.hasOwnProperty(p)) {
            const descriptor = Object.getOwnPropertyDescriptor(globalContext, p);
            const { writable, configurable, enumerable } = descriptor!;
            if (writable) {
              Object.defineProperty(target, p, {
                configurable,
                enumerable,
                writable,
                value,
              });
            }
          } else {
            // @ts-ignore
            target[p] = value;
          }

          if (variableWhiteList.indexOf(p) !== -1) {
            // @ts-ignore
            globalContext[p] = value;
          }

          updatedValueSet.add(p);

          this.latestSetProp = p;

          return true;
        }

        if (process.env.NODE_ENV === 'development') {
          console.warn(`[qiankun] Set window.${p.toString()} while sandbox destroyed or inactive in ${name}!`);
        }

        // 在 strict-mode 下，Proxy 的 handler.set 返回 false 会抛出 TypeError，在沙箱卸载的情况下应该忽略错误
        return true;
      },

      // get时先从FakeWindow上获取，如果获取不到则去Window上获取
      get: (target: FakeWindow, p: PropertyKey): any => {
        this.registerRunningApp(name, proxy);

        if (p === Symbol.unscopables) return unscopables;
        // avoid who using window.window or window.self to escape the sandbox environment to touch the really window
        // see https://github.com/eligrey/FileSaver.js/blob/master/src/FileSaver.js#L13
        if (p === 'window' || p === 'self') {
          return proxy;
        }

        // hijack globalWindow accessing with globalThis keyword
        if (p === 'globalThis') {
          return proxy;
        }

        if (
          p === 'top' ||
          p === 'parent' ||
          (process.env.NODE_ENV === 'test' && (p === 'mockTop' || p === 'mockSafariTop'))
        ) {
          // if your master app in an iframe context, allow these props escape the sandbox
          if (globalContext === globalContext.parent) {
            return proxy;
          }
          return (globalContext as any)[p];
        }

        // proxy.hasOwnProperty would invoke getter firstly, then its value represented as globalContext.hasOwnProperty
        if (p === 'hasOwnProperty') {
          return hasOwnProperty;
        }

        if (p === 'document') {
          return document;
        }

        if (p === 'eval') {
          return eval;
        }

        const value = propertiesWithGetter.has(p)
          ? (globalContext as any)[p]
          : p in target
          ? (target as any)[p]
          : (globalContext as any)[p];
        /* Some dom api must be bound to native window, otherwise it would cause exception like 'TypeError: Failed to execute 'fetch' on 'Window': Illegal invocation'
           See this code:
             const proxy = new Proxy(window, {});
             const proxyFetch = fetch.bind(proxy);
             proxyFetch('https://qiankun.com');
        */
        const boundTarget = useNativeWindowForBindingsProps.get(p) ? nativeGlobal : globalContext;
        return getTargetValue(boundTarget, value);
      },
    });

    this.proxy = proxy;
    //...
}
```

#### 样式隔离

创建appWrapper时做了特殊处理

1. 使用strictStyleIsolation使用shadowDOM
2. 使用experimentalStyleIsolation，在appElement上添加前缀遍历所有css添加前缀

```typescript
function createElement(
  appContent: string,
  strictStyleIsolation: boolean,
  scopedCSS: boolean,
  appInstanceId: string,
): HTMLElement {
  const containerElement = document.createElement('div');
  containerElement.innerHTML = appContent;
  // appContent always wrapped with a singular div
  const appElement = containerElement.firstChild as HTMLElement;
  if (strictStyleIsolation) {
    if (!supportShadowDOM) {
      console.warn(
        '[qiankun]: As current browser not support shadow dom, your strictStyleIsolation configuration will be ignored!',
      );
    } else {
      const { innerHTML } = appElement;
      appElement.innerHTML = '';
      let shadow: ShadowRoot;

      if (appElement.attachShadow) {
        shadow = appElement.attachShadow({ mode: 'open' });
      } else {
        // createShadowRoot was proposed in initial spec, which has then been deprecated
        shadow = (appElement as any).createShadowRoot();
      }
      shadow.innerHTML = innerHTML;
    }
  }

  if (scopedCSS) {
    const attr = appElement.getAttribute(css.QiankunCSSRewriteAttr);
    if (!attr) {
      appElement.setAttribute(css.QiankunCSSRewriteAttr, appInstanceId);
    }

    const styleNodes = appElement.querySelectorAll('style') || [];
    forEach(styleNodes, (stylesheetElement: HTMLStyleElement) => {
      css.process(appElement!, stylesheetElement, appInstanceId);
    });
  }

  return appElement;
}
```

## Demo演示
[Qiankun微前端演示项目](https://github.com/TheScenery/micro-frontend-basement)

![QiankunDemo](/images/qiankun/qiankunDemo.png)
