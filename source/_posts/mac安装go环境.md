---
title: mac安装go环境
date: 2018-12-14 14:12:39
tags:
  - go
---


# mac 安装go

到官方下载 `pkg` 包，点击安装，安装完成后，会自动将 `/usr/local/go` 添加到环境变量，另外需要添加 `GOPATH`,`GOROOT` 到配置环境。

```bash
> vi .bash_profile
# go env
export GOPATH=/Users/cody/go
export GOROOT=/usr/local/go
```


# eclipse go 环境配置

`help -> install new software`, 在打开的窗口中，添加 `GoClipse` 的插件地址 [ http://goclipse.github.io/releases/ ]( http://goclipse.github.io/releases/ )

参考文档 [ref](https://github.com/GoClipse/goclipse/blob/latest/documentation/Installation.md#installation)

# eclipse 报错

```error
Eclipse Invalid Go environment, GOROOT is empty.
```

出现原因为，安装完 `GoClipse` 插件之后，未配置环境变量。

# eclipse 配置 go 环境变量

`Preferences -> go` 中配置 `go` 的 `GOROOT` 目录，以及 `go` 的 `GOPATH`

# 安装 go 其它工具

`Preferences -> go -> tools` 所有的 `download` 按钮点击一下，下载必要的工具

# 坑

新建项目，必须要在 `src` 下创建 `main` 目录，并且要有一个 `go` 文件的包名为 `main`

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("hello world111")
}

```








