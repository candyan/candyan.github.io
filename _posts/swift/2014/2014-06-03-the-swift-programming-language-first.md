---
layout: post
title: The Swift Programming Language
category: blog
description: 语法指南 —— 基础篇
tags: Swift iOS
---

#语法指南 —— 基础篇

Swift是开发 iOS 和 OS X app 的一门新语言。尽管如此，Swift的一些部分还是和你开发C和Objective-c的经验十分相似的。

Swift提供了一套自己的C和Objective—C基本类型，其中包括 `Int` 对应 `integers`；`Double` 和 `Float` 对应浮点类型；`Bool` 对应Boolean类型；和 `String` 对应 文本数据。Swift对于两种基础的集合类型 `Array` 和 `Dictionary` 也提供了强大的版本，在 Collection Types 中有详尽的描述。

和C语言一样，Swift使用变量来保存数值并且通过一个标识名来使用这个数值。Swift也广泛使用数值是不可变的变量。这就是众所周知的常量，并且他比C语言中的常量更加的强大。在Swift中，随处可见的常量使你的代码更加安全，并且当你需要使用不需要改变的数值时，常量的使用可以使你的意图更加清晰。

除了熟悉的类型，Swift也有一些在Objective-C中没有的高级类型。这些包括：`元组（tuples）`，他可以帮助你创建和传递一组数值。元组可以让一个只有单一返回值的方法返回一组数量可变的数值。

Swift也介绍了任意类型（Optional），其可以处理数值的缺省。任意类型是说：要么“它是一个数值并且等于x”，要么“它根本不是一个数值”。任意类型有些像对Objective-C的指针使用 `nil`，但是它是对任何类型都是有效的不光是对象。任意类型比Objective-C的 `nil`指针更加安全和生动，并且它是Swift许多强大特性的核心。

任意类型让Swift成为了一门类型安全的语言。Swift能够帮助你理清楚代码中所使用数据的类型。如果你们的代码期望的是一个 `String`，类型安全机制会防止你错误的传递一个 `Int`类型给他。这使你在开发阶段可以尽可能早的发现并修复错误。

###**常量和变量**

常量和变量把一个名字（例如：maximumNumberOfLoginAttempts 或者 welcomeMessage）和特定类型的数值（例如：数字 10 或者 字符串 "Hello"）联系到一起。常量的数值一旦被赋值就不可以再改变了，但变量可以在赋值之后任意的改变。

###**常量和变量的声明**

常量和变量必须在使用之前先声明。你可以使用 `let` 关键字来声明一个常量，`var` 关键字来声明一个变量。下面的例子会告诉你怎样使用常量和变量在跟踪一个用户的尝试登录的次数：

```javascript
let maximumNumberOfLoginAttempts = 10
var currentLoginAttempt = 0
```

这段代码可以理解为:

“声明了一个叫做 maximumNumberOfLoginAttempts 的常量并且赋值为 `10` 。然后声明了一个叫做 currentLoginAttempt 的变量并且初始化为 `0`。”

在这个例子中，允许尝试登录的最大次数被声明为常量，因为最大值是不会改变的。当前登录的数量被声明为变量，以为这个数值在登录失败后一定会增加。

你也可以在一行中声明多个变量或者常量，并用逗号隔开：

```javascript
var x = 0.0, y = 0.0, z = 0.0
```

> 提示
>
>如果你代码中存储的数值不会改变，一定要用 `let` 关键字把它声明为常量。只用当数值需要改变时才使用变量。

###**类型注释**

当你声明一个常量或者变量是，你可以提供一个类型注释来清楚的表明这个变量或者常量可以存储的类型。在常量或变量名后面使用冒号来添加一个类型注释，并跟随这个一个空格和所使用的变量类型。

下面这个例子给一个叫做 welcomeMessage 的变量提供了一个类型注释，来指示这个变量可以存储 `String`数值：

```javascript
var welcomeMessage: String
```

冒号在声明中的意思是“...是...类型”，所以这个代码可以理解为：

“声明了一个叫做 welcomeMessage 的变量，它的类型是 `String`。”

“`String`类型”的意思是“能够存储任何 `String` 数值。”

这个 welcomeMessage 变量现在可以被赋值为任何的字符串数值了：

```javascript
welcomeMessage = "Hello"
```

> 提示
>
>在实践中，需要你写类型注释的时候是很少的。如果你给一个变量或常量赋予一个已经定义的初始值的话，Swift可以推断出常量或变量的类型。在上面 welcomeMessage 的例子中，没有提供任何初始值，所以需要给 welcomeMessage 一个类型注释而不能从初始值里推断。
