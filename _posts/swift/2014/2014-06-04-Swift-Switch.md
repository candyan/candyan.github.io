---
layout: post
title: 初探Swift—Switch
category: Swift
tags: Swift iOS
---

Swift 在条件判断上依然沿用了传统的 `if` 和 `switch`。其中 `if` 语句跟传统用法一样这里就不做过多的介绍了。但在 `switch` 语句上 Swift 增加了一些有意思的新特性，下面我们来看下Swift中的 `Switch` 可以怎么用：

###经典用法

```javascript
let peopleID = 4148
switch peopleID {
case 4148:
    println(You are Canydan.)
default:
    println(I don't know who you are.)
}
```

上面的例子就是一个最经典的 `switch` `case` `default` 的写法，在大部分的语言中 `switch` 都是这样用的。

> TIPS
>
> default 关键字一定要写到最后，是不能省略的呦！

###Fall Through

在C, C++, Objective-C... 中 `case` 是可以落空的，例如：

```objectivec
NSInteger peopleID = 4148;
switch (peopleID) {
    case 4148:
    case 4739:
      NSLog("You are Candyan.");
      break;
    default:
      NSLog("I don't know who you are.");
      break;
}
```

但是在 Swift 中 `case` 是不可以落空的，所以你不可以写一个没有执行语句的 `case` 条件。那如何使不同的条件执行相同的语句哪？请看下面的例子：

```javascript
let peopleID = 4148
switch peopleID {
case 4148, 4739:
    println(You are Canydan.)
default:
    println(I don't know who you are.)
}
```

Swift 可以在 同一个 `case` 中使用 `,` 来区隔不同的条件值。

> NOTE
>
> 正因为 Swift 的 `case` 是不能落空的，所以一般情况下，写 `case` 的时候是不用像C或者Objective-C一样使用 break 来确保后面的 `case` 不被执行的。 

当然了前面只是默认情况，如果你非要Fall Through也不是不可以的。有 `fallthrough` 关键字来满足你。

```javascript
let integerToDescribe = 5
var description = "The number \(integerToDescribe) is"
switch integerToDescribe {
case 2, 3, 5, 7, 11, 13, 17, 19:
    description += " a prime number, and also"
    fallthrough
default:
    description += " an integer."
}
println(description)
// prints "The number 5 is a prime number, and also an integer.
```

> TIPS
>
> `fallthrough` 的行为和 C 语言是一样的，并不会去检查下一个case的数值，而直接进入到下个case。

最后，当你想忽略一个 `case` 的时候，可以用 `break`，但不能神马都不写呦。

```javascript
let peopleID = 4148
switch peopleID {
case 4148, 4739:
    println(You are Canydan.)
default:
   break;
}
```

### 范围匹配（Range Matching）

得力于 Swift 的 `Close Range` 特性， 其 `Switch` 是可以做范围条件匹配的。例如：

```javascript
let count = 3_000_000_000_000
let countedThings = "stars in the Milky Way"
var naturalCount: String
switch count {
case 0:
    naturalCount = "no"
case 1...3:
    naturalCount = "a few"
case 4...9:
    naturalCount = "several"
case 10...99:
    naturalCount = "tens of"
case 100...999:
    naturalCount = "hundreds of"
case 1000...999_999:
    naturalCount = "thousands of"
default:
    naturalCount = "millions and millions of"
}
```
### 元组（Tuples）

当然，你也可以用元组来作为 switch 的判断条件，例如在一个二维直角坐标系里面， 来判断一个点是不是在你给定的区域里面：

```javascript
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    println("(0, 0) is at the origin")
case (_, 0):
    println("(\(somePoint.0), 0) is on the x-axis")
case (0, _):
    println("(0, \(somePoint.1)) is on the y-axis")
case (-2...2, -2...2):
    println("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    println("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}
```

> NOTE
>
> 跟 C 不太一样，Swift 的 `Swtich` 允许 在不同的case里面有相同的值。例如上面的里面，如果somePoint是(0, 0)，那么其实下面的所有 case 都是匹配的。但在 Swift中 `Switch` 是顺序匹配的，所以当检查到第一个 case 满足之后后面的都会被忽略掉。

> TIPS
>
> 其中 `_` 表示可以是任意的值。可以理解为这个 `case` 并不 care 这个位置是啥，是啥都可以。


### Where 和 数值绑定


在Swift中，`switch` 的 `case` 可以获取一个匹配上的数值到一个变量中，例如在刚才的坐标系里面：

```javascript
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    println("on the x-axis with an x value of \(x)")
case (0, let y):
    println("on the y-axis with a y value of \(y)")
case let (x, y):
    println("somewhere else at (\(x), \(y))")
}
// prints "on the x-axis with an x value of 2”
```

> NOTE
>
> 上面的例子里面没有 `default`， 是因为 `case let (x, y)` 已经可以匹配到任意的一个坐标了，这样就不需要 `default` 来补全这个 `swtich` 了。

当然，我们可以不把条件数值直接写出来，用 `where` 也可以来做相应的匹配。只要 `where` 后面的结果返回的是 `true`，那么这个case就会被执行，例如：

```javascript
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where (y == 0 && x == 0):
    println("original point.")
case let (x, y) where y == 0:
    println("on the x-axis with an x value of \(x)")
case let (x, y) where x == 0:
    println("on the y-axis with a y value of \(y)")
case let (x, y) where x == y:
    println("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    println("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    println("(\(x), \(y)) is just some arbitrary point")
}
```

从上面的介绍可以看的出来，Swift的 `switch` 语句要比 C, C++, Objective-C 强大很多也好用很多。
最后，Swift 的 `switch` 给我的感觉就是一个好看的、封装好的 if-else ╮(╯_╰)╭。