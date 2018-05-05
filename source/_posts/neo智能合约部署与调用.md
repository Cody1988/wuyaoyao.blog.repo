---
title: neo智能合约部署与调用
date: 2018-05-05 15:22:11
tags:
  - 区块链
  - neo
  - 智能合约
---
# Neo 智能合约部署
使用官方`WooLong`的demo来部署第一个智能合约

> 说明：  
> 在使用`WooLong`之前，应该对程序稍微做一些修改，哪怕只是改一下字符串；这个我在测试的时候遇到一个坑，直接使用官方的demo，发布后根据`ScriptHash`查找发现作者版本都不是我填写的那样，后来才知道，`ScriptHash`是根据合约脚本的二进制码产生的。如果不修改，那么最终根据`ScriptHash`查找的可能不是你部署的合约。

## 创建项目并编译
创建Neo项目，将`WooLong`的代码拷贝到demo中，并稍作修改。
<p align="center"><image src="/images/neo智能合约调用/create-neo-contract-project.jpg"></p>

## 打开neo-gui开发者版本
对于开发者，官方建议使用coz提供的开发版[neo-gui](https://github.com/CityOfZion/neo-gui-developer)，clone代码下来，并通过`visual studio 2017`编译项目，然后按照官方文档同步`testnet`的区块，推荐使用离线同步包。[参考文档](http://docs.neo.org/zh-cn/network/syncblocks.html)。

> 来自文档的坑  
> 那么这里又有了一个坑，coz在github中提供的这个`neo-gui-developer`代码，在调用智能合约的时候，会无法添加参数，因此我们需要到`neo`[官方的neo-gui](https://github.com/neo-project/neo-gui)，将项目下面`UI`目录下的以`ParametersEditor`开头的几个文件全部都拷贝进来。


## 打开部署智能合约界面，部署合约

<p align="center"><image src="/images/neo智能合约调用/depoly.jpg"></p>

点击[部署]按钮部署合约

## 拷贝ScriptHash

```hash
0x9499e029baba1221f903476b23a4d59866bb76e1
```

## 试运行并调用

部署完成并拷贝`ScriptHash`后，会弹出`调用合约`界面，先点击`试运行`,如果没有报错，就可以点击`调用`，这是在`neo-gui`-> `交易记录`中会产生一个新的交易记录，记录开始是`未确认`状态，稍后会返回已经确认的节点数目。
<p align="center"><image src="/images/neo智能合约调用/depoly-invoke.jpg"></p>

这时，查看一下`gas`，你会发现发布合约消耗了一些`gas`。

# 智能合约调用
打开`高级`->`调用合约`->`函数调用`，填写`ScriptHash`并查找，会查找到我们之前发布的合约。

填写调用合约需要的参数。
<p align="center"><image src="/images/neo智能合约调用/set-params.jpg"></p>

填写完成后，点击`调用`，查看交易记录，有一条新的`未确认`。

至此，合约的部署和发布算是完成了，其中还有很多细节未完善，只是重点说了自己在爬坑的过程中消耗时间最多的几个小细节，大的方向，各位还是先看官方文档，然后慢慢爬坑吧。
