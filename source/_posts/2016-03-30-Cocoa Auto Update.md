---
layout: post
title: 'Cocoa auto update'
date: '2016-03-30'
header-img: "img/home-bg.jpg"
tags:
     - Cocoa
     
author: '778477'
---


# Cocoa auto update

我们在进行Mac 桌面应用开发的时候，需要维护应用的更新。我们希望说每一次的新版本发布,用户都能够自动更新下载版本并且替换掉老版本。


首先需要一个文件服务器来存放 对外发布的软件压缩包。这里还维护一份releaseInfo.json对外描述已有的版本信息

```
{
  "app": "MyApp",
  "version": "1.0.0",
  "channels": ["release"],
  "entries": [
    {
      "os": "osx",
      "architectures": ["x86-64"],
      "osversion": " >= 10.6 ",
      "appversion": "1.0.0",
      "path": "MyApp-1.0.0-osx.tar.gz",
      "format": "gz"
    }
  ]
}

```

客户端定时check版本信息，发现服务端有版本号大于自身版本号的时候 触发更新下载逻辑。 这里使用`NSURLSession`即可，下载过程不在这里描述了。

当我们自行下载完安装包之后，如何帮助用户自动安装呢?

我们下载完成之后，可用使用`NSTask`来调用一个 shell 脚本，让它来帮助我们结束老版本的进程，同时将下载好的新版本解压、拷贝内容到老版本的路径下、重新唤起App。 这个时候，app运行就是新版本的样式和逻辑

附上具体的shell脚本：
 
**注意需要传入的参数：tarball指的下载的安装包路径，destination指的当前app的bundlePath**


```
function abort() {
    echo "Update script aborted."
    echo "Removing temporary directory..."
    rm -rf $tempdir
    echo "Removing tarball..."
    rm -rf "$tarball"
    echo "Relaunching bundle..."
    open "$destination"
    rm -f ~/.auto-update.lock
    exit 1
}

tarball=$1
destination=$2

lockfile ~/.auto-update.lock

# Step 1. Wait until all processes from within the bundle are closed

processes=$(echo "$(ps ax)" | grep -v "$0")
# Escape the destination into a regexp that matches it
regexp=$(echo "$destination" | sed 's/[^[:alnum:]_-]/\\&/g')
# Filters entries matching the regexp, and do some magic to preserve the trailing newline
matches=$(echo "$processes" | awk "/$regexp/ { print \$1 }"; echo .)
matches=${matches%.}
# Count matches
count=`printf "%s" "$matches" | wc -l`
if [[ $count -gt 0 ]]; then
    for pid in $matches
    do
        echo "$pid"
        kill -9 "$pid"
    done
fi


# Step 2. Check if the downloaded tar is empty, if so abort
if [ ! -s "$tarball" ]
then
    abort
fi

# Step 3. Extract the new contents
echo "Creating temporary directory..."
tempdir=`mktemp -d /tmp/auto-update.XXXXX`
echo "Extracting new content from tarball..."
tar -xf "$tarball" -C "$tempdir"

# Step 4. Check if the extraction worked, if not abort
if [ $? -ne 0 ]
then
    abort
fi

# Step 5. Remove the old bundle directory
echo "Removing bundle..."
rm -rf "$destination"/*
echo "Moving new content into place..."
mv -f $tempdir'/'$(ls $tempdir | head -n 1)'/'* "$destination"'/'
echo "Make sure destination is not quarantined..."
xattr -d com.apple.quarantine "$destination"
echo "Removing temporary directory..."
rm -rf $tempdir
echo "Removing tarball..."
rm -rf "$tarball"

# Step 6. (Re)launch the destination bundle
echo "Relaunching bundle..."
open "$destination"

echo "Done."
rm -f ~/.auto-update.lock
```


# 参考资料
* https://github.com/Automattic/auto-update
* https://github.com/Automattic/auto-update-server



