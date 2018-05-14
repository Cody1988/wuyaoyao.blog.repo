---
title: React集成jest单元测试
date: 2018-05-14 10:43:27
tags:
  - React
  - jest
---

# jest
`jest`是`fb`开发的，可以很方便集成到`React`的单元测试方案。要在`create-react-app`命令行创建的`React`项目中集成`jest`很方便，基本上是开箱即用，可以看官方的`get start`文档。此篇文章主要记录的是我的一个`webpack`的`React`模板项目集成时的步骤。

# 安装

```bash
npm install -D jest babel-jest babel-core regenerator-runtime babel-preset-jest
```
# 添加`babel`配置

打开`.babelrc`文件，并添加`env`配置项：

```json
{
  "presets": [
    "stage-3",
    "flow",
    "react"
  ],
  "plugins": [
    "transform-decorators-legacy",
    "transform-class-properties",
    [
      "import",
      {
        "libraryName": "antd",
        "style": true
      }
    ]
  ],
  "env": {
    "test": {
      "plugins": [
        "transform-es2015-modules-commonjs"
      ]
    }
  }
}
```

# 添加`package.json`配置

在`package.json`的`scripts`中添加`test`配置项
```keyvalue
"test": "jest"
```

# 添加测试目录

在项目根目录下添加一个目录`__test__`，后面单元测试的文件都会放在这个目录下，`jest`默认会运行此目录下的所有的单元测试用例，也可以通过命令行指定运行特定的单元测试。

# 示例

新建一个文件`add.js`，并定义一个方法:

```javascript

export const add = (x,y) => {
  return x + y;
}

```

在测试目录，创建测试文件`add.spec.js`，并添加测试

```javascript
import { add } from '../render/utils/add';

test('add(3,2): ', () => {
  expect(add(3, 2)).toBe(5);
});
```

通过命令行，运行测试用例：

```bash
npm run test
```

也可以通过制定文件的方式运行，这样其它的测试用例就不会被运行

```bash
npm run test __test__/add.spec.js
```

运行结果：

<p align="center"><img src="/images/common/react_jest.jpg" /></p>



[jest官网传送门](https://facebook.github.io/jest/docs/en/getting-started.html)

[webpack-react-dva-antd-bolierplate传送门](https://github.com/Cody1988/webpack-react-dva-antd-bolierplate)