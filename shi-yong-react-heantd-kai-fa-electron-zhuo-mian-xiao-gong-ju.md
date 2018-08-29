---
categories:
  - name: electron
    img: 'https://'
description: electron和nw.js有的一拼，后来看到钉钉和ice都使用electron，甚至社区也在使用electron开发了atom游戏的编辑器
---

# 使用React和Antd来开发桌面小工具

![](/assets/syrhk/electron-logo.png)

electron和nw.js有的一拼，后来看到钉钉和ice都使用electron，甚至社区也在使用electron开发了atom游戏的编辑器，于是也来学习一波.

在electron提供一个nodejs和浏览器结合体，称之为类nodejs环境，拥有使用系统权限的能力，可以使用文件，执行系统命令等等，显示的内容的确是html，下面就看我们如何开发一款小引用，首先看文件结构

![](/assets/syrhk/1.png)

这是原来没有react的项目，其中有一个`render.js`和 `main.js`。在electron中，有两大进程，一个是渲染进程，一个是主线程，渲染进程可以说在浏览器环境，而主进程则是在nodejs环境下，当然两个进程可以通信，稍后都会设计这部分内容的。

有了react项目，我们项目得改变一下了。

项目目录:

```txt

├── README.md
├── app.icns
├── dist
│   └── electron
├── icons.png
├── index.html
├── logo.png
├── logo.psd
├── package.json
├── releases
├── src
│   ├── main
│   ├── renderer
│   └── uploads
├── static
├── test
└── yarn.lock

```


## webpack 配置

---

### render 配置

renderer线程配置和普通网页配置其实差不多，首先查看

### main 配置


## 开发脚本dev-runner.js

---


## 编辑脚本build.js

---









