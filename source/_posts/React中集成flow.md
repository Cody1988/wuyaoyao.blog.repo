---
title: React中集成flow
date: 2018-05-08 10:35:07
tags:
  - flow
  - react
  - webpack
---

# 在React项目中使用flow
`flow`是Javascript的静态类型检测器。静态类型检测需不需要看个人以及团队的需要，个人在开发中发现，小项目一两个人的小团队，在开发中很难意识到静态类型检测的重要性，一旦团队人员扩张或者项目变庞大复杂，那么静态类型检测基本上属于必备品。

# 在项目中添加flow
此次实践，主要是在基于`webpack`配置的`React`项目中添加`flow`的配置过程

首先安装`flow-bin`
```bash
npm install --save-dev flow-bin
```

# 在`scripts`中添加`flow`

```json
"scripts": {
    "start": "npm run dev",
    "dev": "cross-env NODE_ENV=development webpack-dev-server --open --config webpack.dev.js",
    "build": "cross-env NODE_ENV=production webpack --config webpack.prod.js",
    "flow": "flow"
  },
```

# 初始化`flow`配置

```bash
flow init
```
此命令会在项目中创建一个`.flowconfig`配置文件，具体各项的含义可以查看[官方文档](https://flow.org/en/docs/config/)。至此如果你的`React`项目是使用`create-react-app`命令行创建的，那么到这一步以及完成`flow`的配置了，如果不是还需要继续配置。

添加`ignore`
```ini
[ignore]
.*/node_modules/.*
.*/dist/.*

[include]

[libs]

[lints]

[options]

[strict]
```

其它配置可以根据需要后面添加

# 配置babel

默认情况下，浏览器是无法运行`flow`的代码的，因此在运行之前需要从`JavaScript`中移除`flow`代码，完成此任务需要`babel-preset-flow`

安装`babel-preset-flow`

```bash
npm install --save-dev babel-preset-flow
```

配置`.babelrc`

```json
{
  "presets": [
    "flow",
    "env",
    "react"
  ]
}
```

# 运行`flow`

```bash
npm run flow
```





