---
layout: post
title: 'ALAssetsLibrary'
date: '2015-09-04'
header-img: "img/home-bg.jpg"
tags:
     - iOS
     
author: '778477'
---

# ALAssetsLibrary
一个资料库实例在与用户没有交互的情况下，能获取到相片和摄影资源

* 在 iOS8及其以后，使用 Photos framework代替 Assets Libray framework。 Photos framework提供了更多的新功能和强大的性能。

这个资料库包含了 来着iTunes或拍摄 的已保存相册。你可以使用资料库来检索所有资料分组 和 保存相片和影像到已经存在的相册中。

## Accessing Assets
authorizationStatus 返回照片数据授权权限

* ALAuthorizationStatusNotDetermined  未授权
* ALAuthorizationStatusRestricted     访问受限
* ALAuthorizationStatusDenied  拒绝访问
* ALAuthorizationStatusAuthorized 授权访问

## Managing Notifications
disableSharedPhotoStreamSupport 关闭分享照片流，无视分享照片流的更新通知消息

## Finding Assets
assetForURL:resultBlock:failureBlock: 使用一个文件的详细url(理解为访问路径)表示文件标示符来访问该文件

* 注意该方法是异步的，当文件被访问时，会询问用户是否授权应用访问相册。 如果允许，resultBlock回调将执行。如果用户拒绝failureBlock回调将执行

## Enumerating Assets
enumerateGroupsWithTypes:usingBlock:failureBlock: 遍历资源分组下的所有资源

* 注意该方法是异步的，关于遍历资源需要用户授权访问。特殊注意事项，如果访问失败原因为 **ALAssetsLibraryAccessGloballyDeniedError**是因为用户没有启用地理信息服务

## Saving Assets
- [ ] writeVideoAtPathToSavedPhotosAlbum:completionBlock:
- [ ] videoAtPathIsCompatibleWithSavedPhotosAlbum:
- [ ] writeImageToSavedPhotosAlbum:orientation:completionBlock:
- [ ] writeImageDataToSavedPhotosAlbum:metadata:completionBlock:
- [ ] writeImageToSavedPhotosAlbum:metadata:completionBlock:

## Managing Asset Groups
 addAssetsGroupAlbumWithName:resultBlock:failureBlock: 新建资源分组到资源库中
 
 * name为新建资源组名字，**不可重名**。类型为ALAssetsGroupAlbum，可读写。同为异步方法，需要用户授权
 
groupForURL:resultBlock:failureBlock:使用资源分组url来访问该资源组
 
 * 同为异步方法

## Constants
具体的资源类型、回调定义，权限类型，通知信息，错误信息。请参见 ALAssetsLibrary Class Reference(iOS 8.3 Documentation)
 




> 如有任何知识产权、版权问题或理论错误，还请指正。
>
> 转载请注明原作者及以上信息。
