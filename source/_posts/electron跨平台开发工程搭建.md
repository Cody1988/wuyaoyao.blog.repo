---
title: electron跨平台开发工程搭建
date: 2018-05-09 15:09:35
tags:
  - 跨平台
---
# `electron` 工程搭建
`electron`官网有一个很不错的`first app`[开始文档](https://electron.org.cn/demo.html)，可以从这里开始。
首先创建一个文件夹`electron-starter`，作为工程目录，然后初始化工程

```bash
mkdir electron-starter & cd electron-starter
npm init
npm i -D electron
```
这样一个就初始化了一个工程，并安装上了`electron`作为开发依赖包，在初始化工程时，需要做一些简单的配置，这些配置最终可以在`package.json`中体现。

```json
{
  "name": "electron-starter",
  "version": "1.0.0",
  "description": "a electron starter",
  "main": "index.js",
  "scripts": {
    "start": "electron ."
  },
  "author": "Cody1988",
  "license": "ISC",
  "devDependencies": {
    "electron": "^2.0.0"
  }
}

```

# 打包
`electron`程序打包有很多工具，目前使用比较广泛的是[electron-builder](https://github.com/electron-userland/electron-builder)，它可以打包`mac`,`win`,`linux`的程序安装包，并且有很多配置项，可以很好的满足我们的需求，并且所有的需求都是通过配置就可以完成的。

安装`electron-builder`
```bash
npm i -D electron-builder
```
相关配置
```json
 "scripts": {
    "start": "npm run dev",
    "dev": "cross-env NODE_ENV=development electron .",
    "dist": "electron-builder",
    "dist-win": "electron-builder --win -ia32",
    "dist-all": "electron-builder -mwl",
    "pack": "electron-builder --dir",
    "postinstall": "electron-builder install-app-deps",
    "clean": "rm -rf ./dist ./render/dist"
  },
  "build": {
    "appId": "com.cody.electron.starter",
    "dmg": {
    },
    "win": {
      "target": "nsis"
    }
  }
```
此配置中，可以通过`npm run dist`打当前系统的安装包，`npm run dist-all`打全平台的安装包。如果有其他的定制化需求，可以[参考文档](https://www.electron.build/)

# 调试
`electron`程序分为两个部分，`主进程`以及`渲染进程`，在`electron`的窗口中，可以通过`DevTools`调试`渲染进程`中的`H5`以及`js`；而调试`主进程`主要通过`命令行开关`或者`开发工具`提供的外部调试器。

# VSCode调试
在`.vscode`目录下，添加配置`launch.js`并使用以下配置

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Main Process",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceRoot}",
      "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron",
      "windows": {
        "runtimeExecutable": "${workspaceRoot}/node_modules/.bin/electron.cmd"
      },
      "args" : ["."]
    }
  ]
}
```
添加好配置之后，在`VSCode`左侧有个小虫子的debug按钮，点击进入，点击顶部的`debug`按钮，即可以调试`electron`的主进程了。

[VSCode调试主进程](https://electronjs.org/docs/tutorial/debugging-main-process-vscode)




# 重要文档
[环境配置](https://electronjs.org/docs/tutorial/development-environment)

[应用程序打包](https://electronjs.org/docs/tutorial/application-packaging)

[开发工具扩展程序](https://electronjs.org/docs/tutorial/devtools-extension)