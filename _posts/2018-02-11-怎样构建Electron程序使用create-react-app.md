---
layout: post
title: 怎样构建Electron程序，使用create-react-app，不需要配置webpack和不用ejecting
date: 2018-02-09 
tag: Electron
--- 

![electron-react](/images/posts/electron/electron-react.jpeg)

[Electron](https://electronjs.org/)
=============
Electron is GitHub’s framework for building cross-platform desktop apps in JavaScript.


[create-react-app](https://github.com/facebookincubator/create-react-app)
=============
React is Facebook’s JavaScript view framework.

开发步骤
==============
1. 运行`create-react-app`生成基本的react程序
2. 运行`npm install --save-dev electron`
3. 从[electron-quick-start](https://github.com/electron/electron-quick-start)找到`main.js`增加到`src`目录下，改名`electron-starter.js`
4. 修改`electron-starter.js`中的`mainWindow.loadURL`，使用`localhost:3000`

        const electron = require('electron');
        // Module to control application life.
        const app = electron.app;
        // Module to create native browser window.
        const BrowserWindow = electron.BrowserWindow;
        
        const path = require('path');
        const url = require('url');
        
        // Keep a global reference of the window object, if you don't, the window will
        // be closed automatically when the JavaScript object is garbage collected.
        let mainWindow;
        
        function createWindow() {
            // Create the browser window.
            mainWindow = new BrowserWindow({width: 800, height: 600});
        
            // and load the index.html of the app.
            mainWindow.loadURL('http://localhost:3000');
        
            // Open the DevTools.
            mainWindow.webContents.openDevTools();
        
            // Emitted when the window is closed.
            mainWindow.on('closed', function () {
                // Dereference the window object, usually you would store windows
                // in an array if your app supports multi windows, this is the time
                // when you should delete the corresponding element.
                mainWindow = null
            })
        }
        
        // This method will be called when Electron has finished
        // initialization and is ready to create browser windows.
        // Some APIs can only be used after this event occurs.
        app.on('ready', createWindow);
        
        // Quit when all windows are closed.
        app.on('window-all-closed', function () {
            // On OS X it is common for applications and their menu bar
            // to stay active until the user quits explicitly with Cmd + Q
            if (process.platform !== 'darwin') {
                app.quit()
            }
        });
        
        app.on('activate', function () {
            // On OS X it's common to re-create a window in the app when the
            // dock icon is clicked and there are no other windows open.
            if (mainWindow === null) {
                createWindow()
            }
        });
        
        // In this file you can include the rest of your app's specific main process
        // code. You can also put them in separate files and require them here.

5. 增加启动文件`"main": "src/electron-starter.js"`到`package.json`
6. 增加启动Electron`"electron": "electron ."`到`package.json`

        {
          "name": "electron-with-create-react-app",
          "version": "0.1.0",
          "private": true,
          "devDependencies": {
            "electron": "^1.4.14",
            "react-scripts": "0.8.5"
          },
          "dependencies": {
            "react": "^15.4.2",
            "react-dom": "^15.4.2"
          },
          "main": "src/electron-starter.js",
          "scripts": {
            "start": "react-scripts start",
            "build": "react-scripts build",
            "test": "react-scripts test --env=jsdom",
            "eject": "react-scripts eject",
            "electron": "electron ."
          }
        }

7. 启动Electron`npm run electron`
    ![react](/images/posts/electron/1.jpg)


指定loadURL在产品和开发模式里
==============
1. 在开发时，对于`mainWindow.loadURL (in electron-starter.js)`一个环境变量能够被指定，如果环境变量存在使用它，否则使用产品模式生成的静态HTML文件
    
    Linux & Mac
        
        "electron-dev": "ELECTRON_START_URL=http://localhost:3000 electron ."
        
    Windows
        
        "electron-dev": "set ELECTRON_START_URL=http://localhost:3000 && electron ."
        
    in `electron-starter.js`
    
        const startUrl = process.env.ELECTRON_START_URL || url.format({
                    pathname: path.join(__dirname, '/../build/index.html'),
                    protocol: 'file:',
                    slashes: true
                });
            mainWindow.loadURL(startUrl);


2. 有个问题，`create-react-app`默认启动`index.html`,启动Electron会失败。我们需要设置`homepage`属性在`package.json`

        "homepage": "./",
        

使用Foreman去管理React和Electron进程
=================
1. 启动和管理React服务和Electron进程
2. 等待React服务再去启动Electron

[Foremen](https://github.com/strongloop/node-foreman)是一个很好的进程管理工具。我们用一下方法增加它，

        
        npm install --save-dev foreman
        
增加`Procfile`文件

        react: npm start
        electron: npm run electron
        
处理1和2我们增加脚本文件`electron-wait-react.js`，作用是等待React服务启动后再启动Electron。

        const net = require('net');
        const port = process.env.PORT ? (process.env.PORT - 100) : 3000;
        
        process.env.ELECTRON_START_URL = `http://localhost:${port}`;
        
        const client = new net.Socket();
        
        let startedElectron = false;
        const tryConnection = () => client.connect({port: port}, () => {
                client.end();
                if(!startedElectron) {
                    console.log('starting electron');
                    startedElectron = true;
                    const exec = require('child_process').exec;
                    exec('npm run electron');
                }
            }
        );
        
        tryConnection();
        
        client.on('error', (error) => {
            setTimeout(tryConnection, 1000);
        });
        
修改`Procfile`

        react: npm start
        electron: node src/electron-wait-react
        
修改`package.json`，替换`electron-dev`

        "dev" : "nf start"
        
运行`npm run dev`


从React App中访问Electron
============
你想访问本地文件系统或者Electron的`ipcRenderer`，使用下面方法。

        const electron = window.require('electron');
        const fs = electron.remote.require('fs');
        const ipcRenderer  = electron.ipcRenderer;
        

代码地址：[GitHub repo](https://github.com/csepulv/electron-with-create-react-app)

本文翻译来自：[How to build an Electron app using create-react-app. No webpack configuration or “ejecting” necessary.](https://medium.freecodecamp.org/building-an-electron-application-with-create-react-app-97945861647c)