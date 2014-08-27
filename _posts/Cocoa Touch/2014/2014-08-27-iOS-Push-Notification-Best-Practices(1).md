---
layout: post
title: iOS Best Practice - Push Notification(1)
category: Cocoa Touch
tags: CocoaTouch iOS Best-Practices
---

[消息推送](https://developer.apple.com/notifications/) 是现在常用的一种及时通知用户相关信息的手段。

Apple 官方对消息推送的实现是有一些建议的：

- 确保你的应用不发送未经请求的消息或者垃圾消息给用户。如果你非法的使用消息推送可能会使你的应用被 App Store 下架。
- 在使用推送服务之前，你必须先让对应 app id 的应用开启推送服务，然后创建一个 Push SSL 证书用来连接 Apple Push Notification service。（=_=||）
- 你的应用不应该过量的使用 APN service的网络负载或者带宽，也不要给设备发送大量的推送消息。
- 消息推送服务是免费的。你的应用不需要缴纳其他的费用。（=_=||）
- 在发送推送消息的时候，要避免一个推送消息一个连接，尽量在发送消息的时候保持对 APN service 的连接。

Apple的这些建议大部分是很中肯的，但其实对现实并没有什么太大的帮助。

Apple 的 消息推送主要分为两个阶段，`device token` 共享 和 发送推送消息。下面我们就来看下 `device token` 共享阶段应该注意些什么？

### 优化推送注册时机

在 iOS 中要获取`device token`需要向系统进行注册：

```objective-c
UIApplication *app = [UIApplication sharedApplication];
[app registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge |
                                         UIRemoteNotificationTypeAlert |
                                         UIRemoteNotificationTypeSound)];
```

>TIPS:
>
> 上面的写法完全是为了显示好看，并无实际意义。写成一行完全可以。

那么需要在何时调用上面的方法哪？通常有一些人是在`application:didFinishLaunchingWithOptions:` 这个方法中调用。在此方法中调用的问题是，随着iPhone内存的不断增大，这个方法被调用的机会会越来越少。当用户在系统中修改了推送设置时，很难实时和准确的为用户请求新的token或者开启或关闭推送功能。所以建议是在 `applicationDidBecomeActive:` 方法中调用，这样可以保证每次app被激活的时候都会去向系统注册和申请新的token。

```objective-c
- (void)applicationDidBecomeActive:(UIApplication *)application
{  
    UIApplication *app = [UIApplication sharedApplication];
    [app registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge |
                                             UIRemoteNotificationTypeAlert |
                                             UIRemoteNotificationTypeSound)];
}
```

当我们向系统注册完成之后，系统就会返回给我们一个 device token。这样我们就可以用这个 device token 来给相应的设备推送消息啦。但是这里有个问题，就是当网络情况不好的时候系统是连接不上APNs的，这样device token可能会返回失败，我就拿不到一个有效的 token 了。所以为了避免这个问题，我们可以在注册的时候事先判断下当前的网络情况：

```objective-c
- (void)applicationDidBecomeActive:(UIApplication *)application
{  
    if([AppMangaer sharedManager].reachability.isReachable) {
        UIApplication *app = [UIApplication sharedApplication];
        [app registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge |
                                                 UIRemoteNotificationTypeAlert |
                                                 UIRemoteNotificationTypeSound)];
    }
}
```

> NOTE
> 
> 这里是用 Reachability 来判断当前的网络状态的。
> 
> AppManager 的单例会持有一个 Reachability 对象，大家可以按自己项目的实际情况来做判断。

我们现在已经的解决了没有网络的情况，这样我们就可以顺利的拿到`device token`了！！吗？

想的太天真！！iOS 有个一个很重要的特性就是在使用某些特定功能之前要经过用户的同意。

所以，你还需要过用户这关，如果用户不同意，你也是依然拿不到`device token`滴。

那我们就对用户听之任之吗？No！这里是有一些技巧的。

当你的 app 第一次去申请推送功能时，系统会去通知用户，让用户做选择是同意让你的 app 给他推送消息，还是不同意。如果这个时候用户不同意，那么事情就大条了。因为系统不会再次提示他，想要去让他再次能够接受你的推送就只能去系统设置里面修改了。一般的用户是找不到的。

所以，我们需要在开始的时候预估下用户现在是不是要接受。如果用户现在不想接收推送，我们就可以不去注册推送这个功能；如果想，我们再去注册。然后不想接收的用户，我们还可以每隔一段时间再去问一次。这样就可以以一个最低的成本来保证大部分用户都能够同意系统弹出的提示。

> NOTE
> 
> 这里有篇[文章](http://adriancrook.com/ios-best-practice-push-notifications-dialog/)，思想很接近。大家可以借鉴参考下。


### Device Token 存储和同步

上面我们讨论的是如果优化`device token`的获取，那么当拿到了一个合法的`device token`之后我们应该如何正确和高效的把它传到我们自己的服务器上哪？

首先我们来看下如何拿到这个`device token`:

```objective-c
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    if(deviceToken) {
        [self uploadDeviceToken:deviceToken complete:^(BOOL success){
        }];
    }
}
```

这样我们就可以把一个有效的`device token`传到服务器上啦。

但有一个问题是当你每次调用 `registerForRemoteNotificationTypes:` 的时候，`application:didRegisterForRemoteNotificationsWithDeviceToken:` 这个方法都会被调用返回给你一个`device token`。而且通常这个`device token`是不会变的。所以我们就需要把这个 `device token`保存下来，来防止频繁的像服务器发送无意义的请求：


```objective-c
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    if(deviceToken &&
       ![AppManager sharedManager].deviceToken isEqual:deviceToken]) {
        [self uploadDeviceToken:deviceToken complete:^(BOOL success){
            if (success) {
                [AppManager sharedManager].deviceToken = deviceToken;
            }
        }];
    }
}
```

> NOTE
> 
> [AppManager sharedManager].deviceToken 会把`device token`保存到 UserDefault 里面。
> 
> 要在上传`device token`成功之后才去保存到本地，这样可以在请求失败的时候，下次继续上传。

最后，再在 `application:didFailToRegisterForRemoteNotificationsWithError:` 方法里面收集下注册推送时候错误。这下就完美了。

到这个时候，我们已经把推送服务的注册，`device token`的获取，再到同步到服务器的整个过程都做了一次介绍和优化，保证了在`device token`共享阶段，能够实时的，稳定的，有效的获取`device token`并同步到服务器上。

下一篇会继续介绍如何优化发送推送消息，和其中的遇到的坑。
