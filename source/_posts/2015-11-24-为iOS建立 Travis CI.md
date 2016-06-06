---
layout: post
title: '为iOS建立 Travis CI'
date: '2015-11-24'
header-img: "img/home-bg.jpg"
tags:
     - iOS
     
author: '778477'
---

[为iOS建立 Travis CI](http://objccn.io/issue-6-5/)

我给自己的GMYHotSpotView项目 关联了这个 持续集成的工具 .travis.yml文件编写如下

```
language: objective-c
script: xcodebuild -workspace GMYHotSpotView.xcworkspace -scheme GMYHotSpotView
```


[Building an Objective-C Project](https://docs.travis-ci.com/user/languages/objective-c/)

我后面遇到的一个比较麻烦的问题是：

```
Check dependencies
Code Sign error: No code signing identities found: No valid signing identities.
```

最后修改.travis.yml文件，主要是新增了 ** CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO **

```
## http://lint.travis-ci.org/
language: objective-c
xcode_workspace: GMYHotSpotView.xcworkspace
xcode_scheme: GMYHotSpotView
script: xcodebuild -workspace GMYHotSpotView.xcworkspace -scheme GMYHotSpotView CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO
```

![CI](https://raw.githubusercontent.com/778477/778477.github.io/master/img/travis-ci1.png)


指示灯是可以拿来使用的  [![Build Status](https://travis-ci.org/778477/GMYHotSpotView.svg?branch=master)](https://travis-ci.org/778477/GMYHotSpotView)


~~GitHub上还有一个很完善的YAML配置 https://github.com/BoltsFramework/Bolts-iOS/blob/master/.travis.yml~~ **这个没啥卵用**



> [构建iOS持续集成平台（三）——CI服务器与自动化部署 @infoQ](http://www.infoq.com/cn/articles/build-ios-continuous-integration-platform-part3)


> http://stackoverflow.com/questions/27671854/travis-ci-fails-to-build-with-a-code-signing-error

> http://stackoverflow.com/questions/27671854/travis-ci-fails-to-build-with-a-code-signing-error