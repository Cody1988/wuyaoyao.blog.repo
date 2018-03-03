---
title: weex应用debug
date: 2018-03-03 09:21:08
tags:
  - weex
  - debug
---
# 背景
[weex的官方文档](https://weex.apache.org/cn/guide/set-up-env.html)里面有很详细的关于weex的开发资料，但是在开发的过程中，按照文档来debug，无法正常进入，经过很长时间的调研以及查找，最终找到一个合适的方案，此处予以记录
# 创建weex应用
```bash
weex create weex-starter
```

# 安装依赖
```bash
npm i
```

# 运行
```bash
npm run dev & npm run serve
```

# debug	
```bash
weex debug
```

# 扫码
1. 使用weex playground扫描`weex debug`打开的页面上的二维码；
2. 将 `http:ip:8081/dist/index.js`生成二维码，并使用weex playground扫描；

# 打开debug
在网页上打开 `RemoteDebug` 选项，即可以debug了； 