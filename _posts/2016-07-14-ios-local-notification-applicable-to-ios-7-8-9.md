---
layout: blog
categories: blog
published: false
title: iOS Local notification (Applicable to iOS 7/8/9)
---
iOS 从4.0开始允许开发者使用本地推送(`UILocalNotification`).而在实际开发过程中，需根据实际系统的版本，编写不同的设置初始化代码.

## iOS 7以前版本使用本地推送

在iOS7以前，使用本地推送非常简单，只需要实例化一个`UILocalNotification`对象，并设置notification推送的日期，和推送消息的内容即可。参考代码如下：

        UILocalNotification *notification = [[UILocalNotification alloc] init];
        [notification setFireDate:[NSDate dateWithTimeIntervalSinceNow:1]];
        [notification setTimeZone:[NSTimeZone defaultTimeZone]];
        [notification setAlertBody:@"Network is reachable."];
        [[UIApplication sharedApplication] scheduleLocalNotification:notification];
        
## iOS 8以后版本使用本地推送

在iOS 8以后的环境中，第一次使用UILocalNotification的时候，按上述代码编写本地消息推送，但是在真机运行中，一直没有看到有实际的消息推送。而在iOS 7却十分正常，后来查阅了些资料后发现，原来iOS 8以后的版本需要注册推送消息，然后才能schedule推送消息。

iOS 8版本之后，新增了以下3个类

* UIUserNotificationAction / UIMutableUserNotificationAction
* UIUserNotificationCategory / UIMutableUserNotificationCategory
* UIUserNotificationSettings

和一个enum
* UIUserNotificationType

而在创建推送消息前，必须通过UIApplication 的registerUserNotificationSettings:注册推送消息. 详细的创建推送消息的过程如下：

1. 创建NotificationAction

		// create action
        UIMutableUserNotificationAction *action = [[UIMutableUserNotificationAction alloc] init];
        [action setIdentifier:@"your action"];
        [action setActivationMode:UIUserNotificationActivationModeForeground];
        
2. 创建Category

		// create category
        UIMutableUserNotificationCategory *category = [[UIMutableUserNotificationCategory alloc] init];
        [category setIdentifier:@"your category"];
        [category setActions:@[action] forContext:UIUserNotificationActionContextDefault];
        
3. 创建Settings

        // create settings
        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert categories:[NSSet setWithObject:category]];

4. 创建本地推送消息

        UILocalNotification *notification = [[UILocalNotification alloc] init];
        
        // iOS8版本后新增的方法, 设置为刚刚注册的UIMutableUserNotificationCategory的identifier
        [notification setCategory:@"your category"];

        [notification setFireDate:[NSDate dateWithTimeIntervalSinceNow:1]];
        [notification setTimeZone:[NSTimeZone defaultTimeZone]];
        [notification setAlertBody:@"Network is reachable."];
        [[UIApplication sharedApplication] scheduleLocalNotification:notification];

这样，就完成了最基本的本地消息推送.