---
title: 解析 crash 堆栈
date: 2016-04-09 20:26:06
tags:
---

本文介绍了如何解析 iOS 的 crash 堆栈，分别使用了 `symbolicatecrash` 来自动解析整个堆栈，以及使用 `atos` 来解析单个地址的符号。
在 iOS 开发中，解决 crash 问题是比较常见的工作。其中能够解析出符号当然是定位问题的开始。
实际工作中，也有看到很多人其实会卡在解析符号这里，遇到这种情况，可以按照本文中的做法解决。
<!-- more -->

### 使用 symbolicatecrash 解析堆栈

`symbolicatecrash` 是 Xcode 自带的 crash 符号解析工具，可以自动搜索本地符号表，解析整个 crash 堆栈。

#### 确认 Xcode 环境

首先，需要确认 Xcode 的环境，执行以下代码，获取当前 Xcode 的目录。

```
/usr/bin/xcode-select -print-path
```

结果应该是：

```
/Applications/Xcode.app/Contents/Developer/
```

如果结果不是上述的路径，则指定一下路径：

```
sudo /usr/bin/xcode-select -switch /Applications/Xcode.app/Contents/Developer/
```

#### 准备好解析堆栈符号的工具：symbolicatecrash

需要先找到 symbolicatecrash 所在的路径，以 `Xcode 7.3` 版本为例，执行：

```
find /Applications/Xcode.app -name symbolicatecrash -type f
```

将会返回：

```
/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash
```

可以做一个快捷方式：

```
alias symbolicatecrash='/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash'
```

需要先配置好 `DEVELOPER_DIR`，否则会报错。如下：

```
export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer/
```

#### 开始解析
准备好 dSYM 文件和 app 文件，可以存放在任何位置，只要 mac 系统的 spotlight 能够找到就行。
接着执行
```
symbolicatecrash xxx.crash
```

就可以解析符号了。

#### 找不到符号的解决方法

首先，需要确认一下符号表是不是正确的。可以通过以下方式看看符号文件和堆栈是否是对应的 (判断 uuid 是否相同)：
```
dwarfdump --uuid MyApp.app/MyApp
dwarfdump --uuid xxx.app.dSYM/Contents/DWARF/Resources/MyApp
grep "0x.*com.wison.xxx .*<" NoSymbolsTestxxx.crash
```
如果不一样，那么说明崩溃堆栈和符号文件对应不上，很可能是搞错版本，或者打包的时候有问题导致符号文件生成不正确。
如果输出一样的 uuid，那么就是对应的，此时 `symbolicatecrash` 应该可以正常解析符号。
如果还是不能正确解析，那么很可能是 mdfind 自动查找的问题。
Xcode 找符号文件的时候，是通过 mdfind 来找的，比如：

```
mdfind 'com_apple_xcode_dsym_uuids = *'
```

该命令会把当前环境下的所有符号文件找出来。
如果你的符号文件不在此列表中，说明 mdfind 找不到我们的符号，

那么就在执行 `symbolicatecrash` 的时候显式指定dSYM文件的路径：

```
symbolicatecrash xxx.crash xxx.dSYM/Contents/Resources/DWARF/MyApp  
```
如果还是不能解析，试一试把 App 文件也指定：

```
symbolicatecrash xxx.crash xxx.dSYM/Contents/Resources/DWARF/MyApp MyApp.app/MyApp
```

### 使用 atos 解析单个符号

有时候我们需要解析单个地址的符号，比如 `lr` 寄存器的地址对应的符号，就需要用到 `atos`
用法如下：
```
 atos  -arch [armv7 or arm64] -o [BinaryFile or dSYMFile] -l loadAddress address
```
其中
`-arch` 指定二进制的架构，比如 armv7，armv7s，arm64 等等。
`-o` 指定符号文件，可以是 dSYM 文件，也可以是包含了符号表的可执行文件。
`-l` 是加载地址，由于 Xcode 默认打开 PIE 选项，所以加载地址每次都不一样，所以需要指定，可以在 crash 堆栈的 Binary Image 那段看到应用的加载地址。
最后一个参数是需要解析符号的地址。
