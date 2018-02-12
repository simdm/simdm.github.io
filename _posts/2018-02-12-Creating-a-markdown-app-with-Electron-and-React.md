---
layout: post
title: 用Electron和React做一个markdown程序
date: 2018-02-12
tag: Electron
--- 

这篇文章一步一步教你使用[Electron]和[React]创建一个基本的markdown程序。


我将描述为什么、怎样创建它，我把这个markdown程序叫做[Mook]。


Mook的代码在[Mook]。


动机
========
我为什么开始这个项目，有两个原因。

首先，最近我看到越来越多的用Javascript开发的印象深刻和感兴趣的东西，我现在也想用[Electron]作一些东西了。

写JavaScript总是让我感到奇怪，我总是试图避免它。每当我试图用JavaScript做一些事情时，我总觉得我在敲打键盘，不论做什么工作。

然而，最近我发现自己越来越喜欢JavaScript，因为它突然觉得是一个很好的工具，可以用来解决我正在做的问题。

其次，无论何时我都在使用笔记软件，我常常觉得一些功能没有了，我能够发现一些功能在一个软件上，很可能其他功能在另一个软件上才有，所以我一直再寻找一个新的更好的笔记软件。

因为这些原因我决定花些时间来学习JavaScript，并决定用[Electron]来做一个新的markdown软件。


需求
========
我认为的markdonw软件的需求如下（实际更多，开始先这些）：
1. 编辑和预览窗口。
2. 分割屏幕，能动态的调整编辑和预览窗口。
3. 支持代码块和代码高亮显示。
4. 支持保存和同步笔记到Github。
5. 一个层次的笔记本和markdown笔记。
6. 支持Latex/数学方程式在编辑器上。
7. 使用主题能够分组不同的笔记。
8. 能够分享这些笔记在Github和其他笔记平台，如Dropbox、Google Docs等。

技术栈
========
对于这个项目我做了两个决定。

例如，我是否应该使用[boilerplate]？我使用React, AngularJS, Riot, Vue还是其他？我要用那个库？等等。

在最后，我决定避免使用boilerplate，因为我想构建这个app的基础和学习这个过程。

听很多朋友们都在说[React]，而且追求时尚的小朋友们也都在用，所以我打算做个app用[React]。

对这个app构建环境
===============
因为我们使用[React]，我将要开始构建一个新的基本的React应用并且增加Electron到里面。

我们将开始我们的项目使用[create-react-app]。

准备环境
------------
[create-react-app]是一个容易的方式，用来构建React基础的配置。

首先，确认你有最新的node和npm版本安装在你的电脑上，运行一下命令：
```
    node -v
    npm -v
    yarn --version
```
例如这个例子，会输出机器上的版本号：
```
    node = v8.4.0
    npm = 5.3.0
    yarn = 1.0.1
```
使用[create-react-app]创建React app
-----------------
安装[create-react-app]
```
    npm install -g create-react-app
```
创建一个新项目，并进入这个目录：
```
    create-react-app mook
    cd mook
```
我们的目录结构像是这样（去除了`node_modules`目录），
```
    tree -I "node_modules"
    .
    ├── README.md
    ├── package.json
    ├── public
    │   ├── favicon.ico
    │   ├── index.html
    │   └── manifest.json
    ├── src
    │   ├── App.css
    │   ├── App.js
    │   ├── App.test.js
    │   ├── index.css
    │   ├── index.js
    │   ├── logo.svg
    │   └── registerServiceWorker.js
    └── yarn.lock
    
    2 directories, 13 files
```
现在我们有了基本的React app，让我们使用`package.json`中的`start`脚本。
```
    yarn run start
``` 
这是将打开一个浏览器，出现下面的页面：

![react_start](/images/posts/electron/react_start.png)

安装[Electron]
----------------
Electron允许我们构建一个跨平台的应用。

安装Electron包：
```
    yarn add electron --dev
```
打开`package.json`，如果一切OK，你将会看到Electron包在`devDependencies`章节里。

使用下面方法，更新`package.json`文件：

1. 增加下列脚本：
```
    "electron-start": "electron ."
```
    
2. 增加`main`属性，并制定它为Electron的启动文件（这个文件需要自己新建）：
```
    "main": "public/main.js"
```
   
`package.json`文件如下：
```
    {
      "name": "mook",
      "version": "0.1.0",
      "main": "public/main.js",
      "private": true,
      "dependencies": {
        "react": "^15.6.1",
        "react-dom": "^15.6.1",
        "react-scripts": "1.0.13"
      },
      "scripts": {
        "start": "react-scripts start",
        "build": "react-scripts build",
        "test": "react-scripts test --env=jsdom",
        "eject": "react-scripts eject",
        "electron-start": "electron ."
      },
      "devDependencies": {
        "electron": "^1.7.6"
      }
    }
```

下一步我们增加一些Electron的事件([Electron’s events](https://github.com/electron/electron/blob/master/docs/api/app.md))来控制应用的生命周期。我们实现下面的事件：

1. [ready](https://github.com/electron/electron/blob/master/docs/api/app.md#event-ready) - 当Electron运行完成时的初始化运行。在这个代码里，将会运行`createWindow`方法，创建一个本地`http://localhost:3000`浏览器的窗口，设置about面板的属性和`mainWindow`为null在close里。

2. [activate](https://github.com/electron/electron/blob/master/docs/api/app.md#event-activate-macos) - 当程序被激活时运行。所以离着我们要呼叫`createWindow`函数，创建一个新的窗口。

3. [window-all-closed](https://github.com/electron/electron/blob/master/docs/api/app.md#event-window-all-closed) - 当窗口关闭时触发。这将关闭所有平台上的应用程序，除了Mac，它只会关闭窗口，但要执行`quit`程序退出。

public/main.js代码：
```
    const electron = require('electron');
    const app = electron.app;
    const BrowserWindow = electron.BrowserWindow;
    
    let mainWindow;
    
    function createWindow() {
      mainWindow = new BrowserWindow({width: 900, height: 680});
      mainWindow.loadURL('http://localhost:3000');
    
      app.setAboutPanelOptions({
        applicationName: "Mook",
        applicationVersion: "0.0.1",
      })
    
      mainWindow.on('closed', () => mainWindow = null);
    }
    
    app.on('ready', createWindow);
    
    app.on('window-all-closed', () => {
      if (process.platform !== 'darwin') {
        app.quit();
      }
    });
    
    app.on('activate', () => {
      if (mainWindow === null) {
        createWindow();
      }
    });
```

确认React仍然在后台运行，如果不是运行起来：
```
    yarn run start
```
打开一个新的命令行窗口，在该目录下执行：
```
    yarn run electron-start
```

运行之后，下面窗口会显示出来：
![electron_react](/images/posts/electron/electron_react.png)
如果React没有在后台运行，Electron app将会出现一个空白的窗口。


创建稳定的开发和构建过程
=========================





代码地址：[GitHub repo](https://github.com/kazuar/mook)

本文翻译来自：[Creating a markdown app with Electron and React](http://kazuar.github.io/markdown-app/)


[Electron]: https://electronjs.org/ "Electron"
[React]: https://reactjs.org/ "React"
[Mook]: https://github.com/kazuar/mook "Mook"
[boilerplate]: https://github.com/chentsulin/electron-react-boilerplate "boilerplate"
[create-react-app]: https://github.com/facebookincubator/create-react-app "create-react-app"

