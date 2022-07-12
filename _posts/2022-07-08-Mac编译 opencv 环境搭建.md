---
layout: post
title: Mac编译opencv 环境搭建
tag: opencv
date: 2022-07-08 13:35:00.000000000 +09:00
---

### 安装 opencv

首先安装 homebrew 运行：

Homebrew 3.5.4

```
brew install opencv
```

### 配置 xcode

xcode 13.4.1

```
Library Search Paths 添加:
/opt/homebrew/opt/opencv/lib

Header Search Paths 添加:
/opt/homebrew/opt/opencv/include/**

library 中添加 /opt/homebrew/opt/opencv/lib 下全部的 .a 文件
```

openCV lib 可能遇到签名问题我们可以在 Xcode Signing & Capabilities -> Hardened Runtime 勾选 Disable Library Validation
