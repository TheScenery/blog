---
title: Blog Cookbook
date: 2018-05-26 14:05:46
tags: Hexo,Github,Blog,Cookbook
---
为了防止以后忘记怎么使用，很有必要在这里将基础步骤都记录下来
1. 原材料
``` bash
   Node.js
   Git
   Hexo
```
2. 基础环境配置
    
    - 安装[Node](https://nodejs.org/en/download/)
    - 安装[Git](https://git-scm.com/downloads)
    - 安装 `hexo` (建议全局安装，使用以下命令), [Hexo文档以及简介。](https://hexo.io/docs/)
    ``` bash
        npm install -g hexo-cli
    ```

3. 个性化定制

    Theme: [material](https://material.viosey.com/)

4. 基础常用命令
    
    - `hexo new [layout] title` 生成新的markdown源文件
    - `hexo generate` 从原始文件生成静态页面
    - `hexo server` 启动一个简单的本地server，可以方便的查看效果
    - `hexo depoly` 将生成好的静态页面上传

    所有的命令都可以简写，直接使用首字母就可以了。 例如： `hexo n [layout] title`

5. 一般流程
    
    - 生成新文章。 `hexo n`
    - 生成静态页面。`hexo g`
    - 启动个server看看效果。 `hexo server`
    - 觉得哪不爽了再改改， 重新生成下，看看效果。
    - 觉得还不错了，就可以发布了。 `hexo d`

6. Deploy相关

    - 配置下站点的`_config.yml`
    ```yml
        deploy:
        type: git
        repo: (repo url)
    ```
    - 安装个push到git的小工具 `npm install hexo-deployer-git --save`