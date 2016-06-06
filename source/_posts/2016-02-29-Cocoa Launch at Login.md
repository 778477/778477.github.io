---
layout: post
title: 'Cocoa Launch at Login'
date: '2016-02-29'
header-img: "img/home-bg.jpg"
tags:
     - Cocoa
     
author: '778477'
---


# Cocoa Launch at Login 

从[Mac Developer Library](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLoginItems.html)中查看文档：

有两种方式可以添加登录启动项：使用Service Management framework 或 使用 shared file list

这两种方式有差别：

 * 使用Service Management framework 在系统的登录项中是不可见的。只有卸载App才能移除登录项
 
 * 使用 shared file list 在系统的登录项中是可见的。用户可以直接在面板上控制他们。（If you use this API, your login item can be disabled by the user, so any other application that communicates with it it should have reasonable fallback behavior in case the login item is disabled.） 原文还有一句大意是指这个API有隐患，所以在OS X 10.10系统上 API被大量Deprecated
 

下面具体介绍一下 这两种的实现

## Adding Login Item Using the Service Management Framework

应用要包含一个 Helper Target，类型也是应用。设置路径为Contents/Library/LoginItems。设置这个Helper Target info.plist LSBackgroundOnly 为 YES。

使用 ``` SMLoginItemSetEnabled ```方法授权Helper，需要传入两个参数：```CFStringRef```指的是Helper Target的bundle identifier. ``` Boolean ``` 表示期望状态。 传入 ```true``` Helper 会每次在用户登录的时候启动。 传入``` false ``` 来注销这个登录项，不会在用户登录的时候启动。 这个方法会返回一个bool值，如果是ture的话，说明期望状态设置成功。如果是false的话，可能存在多个helper

比如 一个公司发布过很多App，但其中的helper bundle identifier都是相同的。 这样会导致在同一个电脑上只有一个App(greatest bundle version number)能成功注册登录项。

这个方式，我未实践过。贴几个相关的博文吧：

[The Launch At Login Sandbox Project](http://blog.timschroeder.net/2012/07/03/the-launch-at-login-sandbox-project/)

[Adding a preference to launch sandboxed app on login](http://rhult.github.io/articles/sandboxed-launch-on-login/)

[在SandBox沙盒下实现程序的开机启动](http://www.tanhao.me/pieces/590.html/)



## Adding Login Items Using a Shared File List

```
- (BOOL)isLaunchAtStartup {
    // See if the app is currently in LoginItems.
    LSSharedFileListItemRef itemRef = [self itemRefInLoginItems];
    // Store away that boolean.
    BOOL isInList = itemRef != nil;
    // Release the reference if it exists.
    if (itemRef != nil) CFRelease(itemRef);
    
    return isInList;
}

- (IBAction)toggleLaunchAtStartup:(id)sender {
    // Toggle the state.
    BOOL shouldBeToggled = ![self isLaunchAtStartup];
    // Get the LoginItems list.
    LSSharedFileListRef loginItemsRef = LSSharedFileListCreate(NULL, kLSSharedFileListSessionLoginItems, NULL);
    if (loginItemsRef == nil) return;
    if (shouldBeToggled) {
        // Add the app to the LoginItems list.
        CFURLRef appUrl = (__bridge CFURLRef)[NSURL fileURLWithPath:[[NSBundle mainBundle] bundlePath]];
        LSSharedFileListItemRef itemRef = LSSharedFileListInsertItemURL(loginItemsRef, kLSSharedFileListItemLast, NULL, NULL, appUrl, NULL, NULL);
        if (itemRef) CFRelease(itemRef);
        [[self.statusMenu itemAtIndex:0] setTitle:@"取消开机自启动"];
    }
    else {
        // Remove the app from the LoginItems list.
        LSSharedFileListItemRef itemRef = [self itemRefInLoginItems];
        LSSharedFileListItemRemove(loginItemsRef,itemRef);
        if (itemRef != nil) CFRelease(itemRef);
        [[self.statusMenu itemAtIndex:0] setTitle:@"设置开机自启动"];
    }
    CFRelease(loginItemsRef);
}

- (LSSharedFileListItemRef)itemRefInLoginItems {
    LSSharedFileListItemRef res = nil;
    
    // Get the app's URL.
    NSURL *bundleURL = [NSURL fileURLWithPath:[[NSBundle mainBundle] bundlePath]];
    // Get the LoginItems list.
    LSSharedFileListRef loginItemsRef = LSSharedFileListCreate(NULL, kLSSharedFileListSessionLoginItems, NULL);
    if (loginItemsRef == nil) return nil;
    // Iterate over the LoginItems.
    NSArray *loginItems = (__bridge NSArray *)LSSharedFileListCopySnapshot(loginItemsRef, nil);
    for (id item in loginItems) {
        LSSharedFileListItemRef itemRef = (__bridge LSSharedFileListItemRef)(item);
        CFURLRef itemURLRef;
        if (LSSharedFileListItemResolve(itemRef, 0, &itemURLRef, NULL) == noErr) {
            // Again, use toll-free bridging.
            NSURL *itemURL = (__bridge NSURL *)itemURLRef;
            if ([itemURL isEqual:bundleURL]) {
                res = itemRef;
                break;
            }
        }
    }
    // Retain the LoginItem reference.
    if (res != nil) CFRetain(res);
    CFRelease(loginItemsRef);
    CFRelease((__bridge CFTypeRef)(loginItems));
    
    return res;
}

```

[launch on startup](http://bdunagan.com/2010/09/25/cocoa-tip-enabling-launch-on-startup/)


** 主要特别注意的是以上方法大都是 Deprecated in OS X V10.10 所以如果你要开发Deployment Target 10.10 以上的App的话不推荐使用这个方法。显然这些API在之后的OS X更新版本中，将会无法调用。导致你的App出现编译问题。 **

当然介于这个方式比较简单，你仍然想使用的话。你可以设置Deployment Target为 10.9 来使用这些在 10.10被废弃的API。是没有问题的。我就是这么干的，😊


## Deprecated APIs

In previous versions of OS X, it is possible to add login items by sending an Apple event, by using the CFPreferences API, and by manually editing a property list file. These approaches are deprecated.

If you need to maintain compatibility with versions of OS X prior to v10.5, the preferred approach is to use Apple events; for details, see LoginItemsAE. Using the CFPreferences API is an acceptable alternative. You should not directly edit the property list file on any version of OS X.

早些 OS X版本的API 和实现方式，现在也许行不通了。我就不翻译了。 
 
 

