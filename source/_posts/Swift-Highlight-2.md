---
title: Swift Highlight 2 ：范型
date: 2016-04-05 00:27:41
tags:
categories: Swift Highlight
---

Swift 相比 Objective-C 的一项改进就是引进了范型(Generic Type)，有了范型，我们可以通过指定容器中对象的类型，让编译器可以做更多的类型检查。

<!-- more -->

比如：
```
var words: [String] = ["Hello", "Swift"]
words.append("2.2")
words.append(2.2) // compile error
```
上面的代码中，`words` 的类型为 `[String]`，即一个包涵 String 类型的数组，所以后面对这个数组插入数字的时候，会编译错误。
同时，由于数组里面只有 String 对象，所以取出的元素直接就是 String 对象，调用时不用做额外的类型检查：
```
var greeting = ""
for word in words {
    greeting = greeting.stringByAppendingString(word)
}
```
相比 Objective-C，由于没有范型的支持，所以使用时需要做类型判断：
```
NSMutableArray *words = [@[@"Hello", @"Swift"] mutableCopy];
[words addObject:@"2.2"];
[words addObject:@2.2]; // compile success
NSString *greeting = @"";
for (NSString *word in words) {
    if ([word isKindOfClass:[NSString class]]) {
        greeting = [greeting stringByAppendingString:word];
    } else {
        // not a string, do nothing
    }
}
```

2015 年的一次更新中，Objective-C 引入了范型语法，推荐以后都用这种方式，但这只是静态的编译期检查，如果运行中出现类型错误，是不会报错的。使用了范型语法的代码如下：

```
NSMutableArray<NSString *> *words = [@[@"Hello", @"Swift"] mutableCopy];
[words addObject:@"2.2"];
[words addObject:@2.2]; // warning：类型不正确
NSString *greeting = @"";
for (NSString *word in words) {
    greeting = [greeting stringByAppendingString:word];
}    
```
