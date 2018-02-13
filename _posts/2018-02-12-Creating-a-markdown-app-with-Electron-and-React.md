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
现在我们已经有了一个对于我们项目的使用React和Electron的模板，我们需要确信我们有一个稳定的构建环境对于开发和发布。

直到现在我们对于开发环境的构建是非常好的，但是最终我们要构建一个对于OS X, Windows and Linux的发布环境。

我也不喜欢我们必须在两个不同的命令行shell中分别运行React服务和Electron程序。

在做一些有调查后，我从一个[From React to an Electron app ready for production](https://medium.com/@kitze/%EF%B8%8F-from-react-to-an-electron-app-ready-for-production-a0468ecb1da3)的帖子帮助了我。

我们需要增加下面几个包在我们的项目中：

1. [electron-builder](https://www.electron.build/) - 一个完整的解决包和构建发布Electron在macOS, Windows and Linux上，并支持自动更新的库。我们将用这个包来发布程序。

2. [concurrently](https://github.com/kimmobrunfeldt/concurrently) - 同时运行命令。我们将使用这个包在一个命令中同时运行React和Electron。

3. [wait-on](https://github.com/jeffbski/wait-on) - 命令行实用程序和Node.js API对于等待文件、端口、sockets和HTTP（S）资源变得可用。我们将使用这个包在等待React服务后在开始运行Electorn app。

运行下面命令增加到这些包到项目中：
```
    yarn add electron-builder wait-on concurrently --dev
```
这些包只是对于开发是必须的，所以增加参数`--dev`，他们将自动增加到`devDependencies`中。

创建一个开发脚本
---------------
当开发这个app时，我们想创建一个开发脚本。这将帮助我们测试我们开发的新特性和调试和确保我们在编辑代码时不会破坏任何东西。

我们增加新甲苯在`package.json`中：
```
    "electron-dev": "concurrently \"BROWSER=none yarn start\" \"wait-on http://localhost:3000 && electron .\""
```
分析一下上面这行代码：
1. concurrently - 同时运行这些程序
2. BROWSER=none yarn start - 开始运行react程序并且不自动打开浏览器
3. wait-on http://localhost:3000 && electron . - 等待开发服务器启动后，启动Electron程序

运行下面命令行，将只开启Electron程序：
```
    yarn run electron-dev
```
![electron_react](/images/posts/electron/electron_react.png)


创建一个构建脚本
-------------
这是非常容易的。我们需要增加两个脚本在`package.json`:
* 编译Electron app之前，编译React app
```
    "preelectron-pack": "yarn build"
```
打包Electron app，这个脚本使用electron-builder：
```
    "electron-pack": "build --em.main=build/electron.js"
```

下一步，我们将制定编译属性，这是因为创建React和Electron之间有一个小冲突，因为两者都使用构建文件夹，来实现两个不同的目的。

为了解决这个冲突，我们需要手工配置electron-builder的正确文件夹。增加下面构建代码到`package.json`:
```
    "build": {
      "appId": "com.mook",
      "files": [
        "build/**/*",
        "node_modules/**/*"
      ],
      "directories": {
        "buildResources": "assets"
      }
    }
```

我们还增加homepage属性，允许打包Electron程序找到Javascript和CSS文件：
```
    "homepage": "./"
```

目前，你的`package.json`文件是这样的：
```
    {
      "name": "mook",
      "version": "0.1.0",
      "main": "public/main.js",
      "homepage": "./",
      "private": true,
      "scripts": {
        "start": "react-scripts start",
        "build": "react-scripts build",
        "test": "react-scripts test --env=jsdom",
        "eject": "react-scripts eject",
        "electron-start": "electron .",
        "electron-dev": "concurrently \"BROWSER=none yarn start\" \"wait-on http://localhost:3000 && electron .\"",
        "electron-pack": "build --em.main=build/main.js",
        "preelectron-pack": "yarn build"
      },
      "dependencies": {
        "react": "^15.6.1",
        "react-dom": "^15.6.1",
        "react-scripts": "1.0.13",
        "electron-is-dev": "^0.3.0"
      },
      "devDependencies": {
        "concurrently": "^3.5.0",
        "electron": "^1.7.6",
        "electron-builder": "^19.27.7",
        "wait-on": "^2.0.2"
      },
      "build": {
        "appId": "com.mook",
        "files": [
          "build/**/*",
          "node_modules/**/*"
        ],
        "directories": {
          "buildResources": "assets"
        }
      }
    }
```

最后一步将更新`public/main.js`。到目前为止，我们只支持应用程序的开发版本。我们的产品版本将无法使用`localhost:3000`，我们从发布文件夹中将用index.html文件替代。

首先，我们需要安装[electron-is-dev]，这将有助于我们确定电React是否在开发模式中运行。

安装[electron-is-dev]:
```
    yarn add electron-is-dev
```

更新`public/main`，使用[electron-is-dev]，

* 增加包到代码中：
```
    const isDev = require('electron-is-dev');
    const path = require('path');
```

* 更改`mainWindow.loadURL`函数：
```
    mainWindow.loadURL(isDev ? 'http://localhost:3000' : `file://${path.join(__dirname, '../build/index.html')}`);
```

上面代码是检测如果是开发模式，使用`localhost:3000`，否则`/build/index.html`。

`public/main.js`代码现在如下：
```
    const electron = require('electron');
    const app = electron.app;
    const BrowserWindow = electron.BrowserWindow;
    const isDev = require('electron-is-dev');
    const path = require('path');
    
    let mainWindow;
    
    function createWindow() {
      mainWindow = new BrowserWindow({width: 900, height: 680});
      mainWindow.loadURL(isDev ? 'http://localhost:3000' : `file://${path.join(__dirname, '../build/index.html')}`);
    
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

现在让我们运行下面脚本：
```
    yarn run electron-pack
```

当这个脚本运行后，你将会看到一个`dist`的文件夹。将能够发现这个app，比如mac上mook.app。

运行结果和开发模式一样：
![electron_react](/images/posts/electron/electron_react.png)

太好了，我们终于完成了基础构建工作！

增加主要功能
==================
现在我们可以开始增加markdown程序的代码了。

创建一个分割窗格的屏幕
---------------------
我们先增加[react-split-pane]组件。安装它：
```
    yarn add react-split-pane
```

增加下面JavaScript代码到`src/App.js`中：
* 导入[react-split-pane]:
```
    import SplitPane from 'react-split-pane';
```

用下面代码替换`render`函数。这个代码增加SplitPane元素渲染两个div，一个是编辑，一个是预览：
```
    render() {
        return (
            <div className="App">
                <SplitPane split="vertical" defaultSize="50%">
                    <div className="editor-pane">
                    </div>
                    <div className="view-pane">
                    </div>
                </SplitPane>
            </div>
        );
    }
```

再增加一些css到`src/App.css`中：
```
    .Resizer {
        background: #000;
        opacity: .4;
        z-index: 1;
        -moz-box-sizing: border-box;
        -webkit-box-sizing: border-box;
        box-sizing: border-box;
        -moz-background-clip: padding;
        -webkit-background-clip: padding;
        background-clip: padding-box;
    }
    
     .Resizer:hover {
        -webkit-transition: all 2s ease;
        transition: all 2s ease;
    }
    
     .Resizer.horizontal {
        height: 11px;
        margin: -5px 0;
        border-top: 5px solid rgba(255, 255, 255, 0);
        border-bottom: 5px solid rgba(255, 255, 255, 0);
        cursor: row-resize;
        width: 100%;
    }
    
    .Resizer.horizontal:hover {
        border-top: 5px solid rgba(0, 0, 0, 0.5);
        border-bottom: 5px solid rgba(0, 0, 0, 0.5);
    }
    
    .Resizer.vertical {
        width: 11px;
        margin: 0 -5px;
        border-left: 5px solid rgba(255, 255, 255, 0);
        border-right: 5px solid rgba(255, 255, 255, 0);
        cursor: col-resize;
    }
    
    .Resizer.vertical:hover {
        border-left: 5px solid rgba(0, 0, 0, 0.5);
        border-right: 5px solid rgba(0, 0, 0, 0.5);
    }
    .Resizer.disabled {
      cursor: not-allowed;
    }
    .Resizer.disabled:hover {
      border-color: transparent;
    }
```

如果你刷新app或者运行`yarn run electron-dev`，你将会看到当前窗口被分了两部分：
![split_pane](/images/posts/electron/split_pane.png)

你能够移动`separator bar`来缩放两个窗口。


创建editer和preview窗格
=====================
现在我们有了两个分割的窗格，我们需要在里面增加功能。我们要设置的窗格左边是editor右边是preview窗格。我们在editor里写markdown，右边实时显示改变。

创建editor窗格
---------------
让我们来创建editor窗格。我们使用[CodeMirror]。

安装React版本的[React-CodeMirror]。因为它有一个小问题[Code mirror value doesn’t update with state change](https://github.com/JedWatson/react-codemirror/issues/106)，所以我们需要安装`@skidding/react-codemirror`来解决这个问题：
```
    yarn add @skidding/react-codemirror
```
创建一个新文件`src/editor.js`，新的类名叫`Editor`，继承自React’s Component class：
```
    import React, { Component } from 'react';
    
    class Editor extends Component {
    }
    
    export default Editor;
```
这个类基本是封装[React-CodeMirror]的React组件[CodeMirror]。

下一步我们导入`@skidding/react-codemirror`和一些对CodeMirror组件和符号高亮、markdown模式的css文件。

我们还增加了render方法，返回CodeMirror元素和增加constructor方法。constructor方法允许我们从main文件里来用值来初始化CodeMirror。

我们设定CodeMirror组件对于markdown模式和主题为monokai：
```
    import React, { Component } from 'react';
    import CodeMirror from '@skidding/react-codemirror';
    
    require('codemirror/lib/codemirror.css');
    require('codemirror/mode/javascript/javascript');
    require('codemirror/mode/python/python');
    require('codemirror/mode/xml/xml');
    require('codemirror/mode/markdown/markdown');
    require('codemirror/theme/monokai.css');
    
    class Editor extends Component {
        constructor(props) {
            super(props);
        }
    
        render() {
            var options = {
              mode: 'markdown',
              theme: 'monokai',
            }
            return (
                <CodeMirror value={this.props.value} 
                    options={options} height="100%"/>
            );
        }
    }
    
    export default Editor;
```

在`src/App.js`文件里导入`editor.js`：
```
    import Editor from './editor.js';
```
在App类的constructor方法里初始化我们的editor：
```
    constructor(props) {
      super();
    
      this.state = {
        markdownSrc: "# Hello World",
      }
    }
```

在App的render方法里增加Editor组件并初始化markdownSrc值：
```
    render() {
        return (
          <div className="App">
            <SplitPane split="vertical" defaultSize="50%">
              <div className="editor-pane">
                <Editor className="editor" value={this.state.markdownSrc}/>
              </div>
              <div className="view-pane">
              </div>
            </SplitPane>
          </div>
        );
      }
```

此时的`src/App.js`文件：
```
    import React, { Component } from 'react';
    import logo from './logo.svg';
    import SplitPane from 'react-split-pane';
    import Editor from './editor.js';
    
    import './App.css';
    
    class App extends Component {
      constructor(props) {
        super();
    
        this.state = {
          markdownSrc: "# Hello World",
        }
      }
    
      render() {
        return (
          <div className="App">
              <SplitPane split="vertical" defaultSize="50%">
                  <div className="editor-pane">
                    <Editor className="editor" value={this.state.markdownSrc}/>
                  </div>
                  <div className="view-pane">
                  </div>
              </SplitPane>
          </div>
        );
      }
    }
    
    export default App;
```

更新css文件`src/App.css`，做下面修改：
1. 删除`text-align: center;`在.App章节，所以text不再居中。
2. 增加下面css，来拉伸editor到最大高度，给右边的文本增加一点padding。
```
    .editor-pane {
      height: 100%;
    }
    
    .CodeMirror {
      height: 100%;
      padding-top: 20px;
      padding-left: 20px;
    }
    
    .ReactCodeMirror {
      height: 100%;
    }
```

刷新app，或者运行`run electron-dev`，应该显示下面画面：
![editor](/images/posts/electron/editor.png)

创建preview窗格
----------------
我们在右边窗格创建一个对于左边编辑实时的预览。

为了这个目的，我们使用[React-Markdown]包：
```
    yarn add react-markdown
```

在`src\App.js`里导入它：
```
    import ReactMarkdown from 'react-markdown';
```

将`ReactMarkdown`组件增加到view-pane div：
```
    <div className="view-pane">
      <ReactMarkdown className="result" source={this.state.markdownSrc} />
    </div>
```

我们设置和`ReactMarkdown`组件相同的值`this.state.markdownSrc`。

现在你可以运行看看preview窗框：
```
    yarn run electron-dev
```
![preview_pane](/images/posts/electron/preview_pane.png)

我们可以看到text在preview窗框里，然而，你在左边写入内容后，右边的preview并不会翻译。

我们要做的是让在editor中的每个改变都在App类中传递给preview。

增加onMarkdownChange方法给`src\App.js`，用editor中的值更新markdownSrc的值。这个函数每当在editor中有改变时都会触发。

增加下面代码到`src\App.js`：
```
    constructor(props) {
      super();
    
      this.state = {
        markdownSrc: "# Hello World"
      }
    
      this.onMarkdownChange = this.onMarkdownChange.bind(this);
    }
    
    onMarkdownChange(md) {
      this.setState({
        markdownSrc: md
      });
    }
```
在render方法中，给editor元素增加下面代码：
```
    <Editor className="editor" value={this.state.markdownSrc} onChange={this.onMarkdownChange}/>
```

在`src/editor.js`文件里，绑定CodeMirror的onChange方法到父类的onChange方法：
```
    constructor(props) {
      super(props);
    
      this.updateCode = this.updateCode.bind(this);
    }
    
    updateCode(e) {
      this.props.onChange(e);
    }
```

在render方法中，修改CodeMirror元素：
```
    <CodeMirror
      value={this.props.value} onChange={this.updateCode}
      options={options} height="100%"
    />
```

此时`src/App.js`文件如下：
```
    import React, { Component } from 'react';
    import logo from './logo.svg';
    import SplitPane from 'react-split-pane';
    import Editor from './editor.js';
    import ReactMarkdown from 'react-markdown';
    
    import './App.css';
    
    class App extends Component {
      constructor(props) {
        super();
    
        this.state = {
          markdownSrc: "# Hello World"
        }
    
        this.onMarkdownChange = this.onMarkdownChange.bind(this);
      }
    
      onMarkdownChange(md) {
        this.setState({
          markdownSrc: md
        });
      }
    
      render() {
        return (
          <div className="App">
              <SplitPane split="vertical" defaultSize="50%">
                  <div className="editor-pane">
                    <Editor className="editor" value={this.state.markdownSrc} onChange={this.onMarkdownChange}/>
                  </div>
                  <div className="view-pane">
                    <ReactMarkdown className="result" source={this.state.markdownSrc} />
                  </div>
              </SplitPane>
          </div>
        );
      }
    }
    
    export default App;
```

此时`src/editor.js`文件如下：
```
    import React, { Component } from 'react';
    import CodeMirror from '@skidding/react-codemirror';
    
    require('codemirror/lib/codemirror.css');
    require('codemirror/mode/javascript/javascript');
    require('codemirror/mode/python/python');
    require('codemirror/mode/xml/xml');
    require('codemirror/mode/markdown/markdown');
    require('codemirror/theme/monokai.css');
    
    class Editor extends Component {
        constructor(props) {
            super(props);
    
            this.updateCode = this.updateCode.bind(this);
        }
    
        updateCode(e) {
            this.props.onChange(e);
        }
    
        render() {
            var options = {
              mode: 'markdown',
              theme: 'monokai',
            }
            return (
                <CodeMirror value={this.props.value} onChange={this.updateCode} options={options} height="100%"/>
            );
        }
    }
    
    export default Editor;
```

此时再次启动程序，你将能在左边editor中修改，实时的看到右边的变化。
![editor_markdown](/images/posts/electron/editor_markdown.png)

未来？
====

这里有一些还需要完成的功能：
1. 保存和打开文件
2. 在编辑时自动保存
3. 工具栏/控制窗框移动
4. 备份笔记到Github/Dropbox/etc
5. 支持分组保存笔记或者同意笔记
6. 支持数学方程在markdown
7. 更多惊人的功能！


可以在Twitter[@kazuarous](https://twitter.com/kazuarous)上查看更新进度，即将推出的功能和其他问题。


贡献
=====



代码地址：[GitHub repo](https://github.com/kazuar/mook)

本文翻译来自：[Creating a markdown app with Electron and React](https://medium.freecodecamp.org/heres-how-i-created-a-markdown-app-with-electron-and-react-1e902f8601ca)


[Electron]: https://electronjs.org/ "Electron"
[React]: https://reactjs.org/ "React"
[Mook]: https://github.com/kazuar/mook "Mook"
[boilerplate]: https://github.com/chentsulin/electron-react-boilerplate "boilerplate"
[create-react-app]: https://github.com/facebookincubator/create-react-app "create-react-app"
[electron-is-dev]: https://github.com/sindresorhus/electron-is-dev "electron-is-dev"
[react-split-pane]: https://github.com/tomkp/react-split-pane "react-split-pane"
[CodeMirror]: https://codemirror.net/ "CodeMirror"
[React-CodeMirror]: https://github.com/JedWatson/react-codemirror "React-CodeMirror"
[React-Markdown]: https://github.com/rexxars/react-markdown "React-Markdown"
