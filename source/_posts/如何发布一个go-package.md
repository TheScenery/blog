---
title: 如何发布一个go package
date: 2022-01-22 00:16:40
tags:
---

如何将自己所写的Go package开源并提供给别人使用？

### 1. 首先找到在代码托管平台创建一个仓库（以Github为例）

![create-repository](/images/publishGoPackage/createRepository.png)

### 2. 添加自己的package代码并且提交

### 3. 回到github该仓库中创建一个release
![create-release](/images/publishGoPackage/createRelease.png)

### 4. 填写release相关信息，表明版本号
![release-info](/images/publishGoPackage/releaseInfo.png)

### 5. 发布成功！可以使用`go get`命令来使用你刚刚发布好的包了
``` shell
go get -u github.com/TheScenery/BoltCompact
```