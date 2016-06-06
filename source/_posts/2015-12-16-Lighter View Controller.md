---
layout: post
title: 'Lighter View Controller'
date: '2015-12-16'
header-img: "img/home-bg.jpg"
tags:
     - iOS
     
author: '778477'
---


[Lighter-View-Controller 轻量级视图控制器](https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/)


视图控制器们(View Controllers)通常是最大的文件在我们的iOS工程中。而且 他们常常有很多不必要的代码。导致视图控制器往往是无法在其他项目中复用的。我们下面讲述的技巧可以让控制器们 更加轻量，有复用性，让不合理的代码出现在合理的位置

[example project on GitHub](https://github.com/objcio/issue-1-lighter-view-controllers)


### 分离出数据协议和其他接口协议
* 一个很有用的技巧就是将视图控制器中的 ```UITableViewDataSource``` 这部分代码分离出来。独立写一个类来维护，如果你都是这么处理的话，那么你这些类是可以以后在其他项目中复用的。
* 这样做还有一个好处就是，我们可以独立测试这个类。

举个例子：在示例项目中，```PhotosViewController``` 有这样几个方法

```
# pragma mark Pragma 

- (Photo*)photoAtIndexPath:(NSIndexPath*)indexPath {
    return photos[(NSUInteger)indexPath.row];
}

- (NSInteger)tableView:(UITableView*)tableView 
 numberOfRowsInSection:(NSInteger)section {
    return photos.count;
}

- (UITableViewCell*)tableView:(UITableView*)tableView 
        cellForRowAtIndexPath:(NSIndexPath*)indexPath {
    PhotoCell* cell = [tableView dequeueReusableCellWithIdentifier:PhotoCellIdentifier 
                                                      forIndexPath:indexPath];
    Photo* photo = [self photoAtIndexPath:indexPath];
    cell.label.text = photo.name;
    return cell;
}
```

让我们试着把这些代码放到 我们自己的类上。我们用闭包来填充数据(或许协议更合适，这个取决于你)

```
@implementation ArrayDataSource

- (id)itemAtIndexPath:(NSIndexPath*)indexPath {
    return items[(NSUInteger)indexPath.row];
}

- (NSInteger)tableView:(UITableView*)tableView 
 numberOfRowsInSection:(NSInteger)section {
    return items.count;
}

- (UITableViewCell*)tableView:(UITableView*)tableView 
        cellForRowAtIndexPath:(NSIndexPath*)indexPath {
    id cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier
                                              forIndexPath:indexPath];
    id item = [self itemAtIndexPath:indexPath];
    configureCellBlock(cell,item);
    return cell;
}

@end
```

这三个方法不再存在我们的视图控制器了，我们可以创建实例来承接table view的数据协议委托

```
void (^configureCell)(PhotoCell*, Photo*) = ^(PhotoCell* cell, Photo* photo) {
   cell.label.text = photo.name;
};
photosArrayDataSource = [[ArrayDataSource alloc] initWithItems:photos
                                                cellIdentifier:PhotoCellIdentifier
                                            configureCellBlock:configureCell];
self.tableView.dataSource = photosArrayDataSource;
```



而且，这种办法扩展其他接口协议也很方便。比如 另外的一个list数据协议是 ```UICollectionViewDataSource``` 由于一些需求实现，你们决定将 ```UITableView``` 替换为 ```UICollectionView```。实际上你的视图控制器不需要任何改动，你可以让你的类都支持这两个数据协议。



### 逻辑处理放在Model
 这里还是一个例子，这些代码在view controller里，功能是返回一个列表，上面的数据是用户的活跃度
 
 
```
- (void)loadPriorities {
  NSDate* now = [NSDate date];
  NSString* formatString = @"startDate <= %@ AND endDate >= %@";
  NSPredicate* predicate = [NSPredicate predicateWithFormat:formatString, now, now];
  NSSet* priorities = [self.user.priorities filteredSetUsingPredicate:predicate];
  self.priorities = [priorities allObjects];
}
```


当这些代码被放置到 User类的扩展中时，view controller就看起来比较清爽了。

```
- (void)loadPriorities {
  self.priorities = [self.user currentPriorities];
}
```

```User+Extensions.m:```

```
- (NSArray*)currentPriorities {
  NSDate* now = [NSDate date];
  NSString* formatString = @"startDate <= %@ AND endDate >= %@";
  NSPredicate* predicate = [NSPredicate predicateWithFormat:formatString, now, now];
  return [[self.priorities filteredSetUsingPredicate:predicate] allObjects];
}
```

当然在实际项目中，有些代码不是那么轻松可以转移到Model当中的。所以我们要创建 "数据仓库"(Store Class)

### 创建仓库类

我们有些代码是从文件中加载数据并处理数据的，这些代码大概是这样的：

```
- (void)readArchive {
    NSBundle* bundle = [NSBundle bundleForClass:[self class]];
    NSURL *archiveURL = [bundle URLForResource:@"photodata"
                                 withExtension:@"bin"];
    NSAssert(archiveURL != nil, @"Unable to find archive in bundle.");
    NSData *data = [NSData dataWithContentsOfURL:archiveURL
                                         options:0
                                           error:NULL];
    NSKeyedUnarchiver *unarchiver = [[NSKeyedUnarchiver alloc] initForReadingWithData:data];
    _users = [unarchiver decodeObjectOfClass:[NSArray class] forKey:@"users"];
    _photos = [unarchiver decodeObjectOfClass:[NSArray class] forKey:@"photos"];
    [unarchiver finishDecoding];
}
```

其实 View Controller 没有必要关心数据是如何处理的，我们创建 数据仓库来处理这些。使这些代码分离出来，这样我们就可以复用这个仓库类了。
独立测试并且View Controller的代码量又减少了一些。这些数据仓库负责数据处理，持久化和数据库交互。我们也可以叫这些"仓库"为服务层或资料库

### 将网络服务逻辑转移到Model

这是一个类似的优化逻辑：不要把网络交互放在View Controller。把这些逻辑独立放置在其他类中，可以调用方法并设置回调处理。
这样做的好处是你可以格外处理所有你的数据和错误在这个类上。而不会让View Controller变的臃肿

### 结论

我们看到有很多技巧可以让View Controller更加精简。我们努力让这些技巧在实际开发中更加实用。
我们只有一个目的：写出可维护的代码。知道这些模式之后，我们有更加深刻的认识来讨论如何将笨重的View Controller精简优化。



### 扩展阅读
* [Cocoa Core Competencies: Controller Object](https://developer.apple.com/library/mac/documentation/General/Conceptual/DevPedia-CocoaCore/ControllerObject.html)
* [Writing high quality view controllers](http://subjective-objective-c.blogspot.de/2011/08/writing-high-quality-view-controller.html)
* [Programmers Stack Exchange: Model View Controller Store](https://programmers.stackexchange.com/questions/184396/mvcs-model-view-controller-store)
* [Programmers Stack Exchange: How to avoid big and clumsy UITableViewControllers on iOS
](https://programmers.stackexchange.com/questions/177668/how-to-avoid-big-and-clumsy-uitableviewcontroller-on-ios)



