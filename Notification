1、消息本地推送：添加、更新、删除等操作都有；
LocalPush.h
[objc] view plain copy print?
//  
//  LocalPush.h  
//  HilinkPlatformSDK  
//  
//  Created by developer on 15/5/13.  
//  Copyright (c) 2015年 cocosbuilder. All rights reserved.  
//  
  
#ifndef HilinkPlatformSDK_LocalPush_h  
#define HilinkPlatformSDK_LocalPush_h  
  
@interface LocalPush : NSObject  
  
+(LocalPush*)sharedSingleton;  
  
- (void) RegisterAndOpenLocalNotification;  
- (void) GetPushInfoByPlatformServer;  
- (void) CreateLocalNotificationByInfo:(NSString *)key deleaTime:(NSInteger)deleaTime pushContent:(NSString *)pushContent;  
- (BOOL) CheckNotificationUpdate:(NSString *)key pushContent:(NSString *)pushContent;  
- (NSInteger) MakeTime:(NSInteger)pushHour Minute:(NSInteger)pushMiute;  
- (void) RemoveLocalNotificationByKey:(NSString *)key;  
- (void) RemoveAllLocalNotification;  
  
@end  
  
#endif  

LocalPush.mm
[objc] view plain copy print?
//  
//  LocalPush.m  
//  HilinkPlatformSDK  
//  
//  Created by developer on 15/5/13.  
//  Copyright (c) 2015年 cocosbuilder. All rights reserved.  
//  
  
#import <Foundation/Foundation.h>  
#import "LocalPush.h"  
#import "HLWebApi.h"  
  
@implementation LocalPush  
  
+(LocalPush*)sharedSingleton {  
    static LocalPush* sharedInstance = nil;  
    static dispatch_once_t predicate;  
      
    dispatch_once(&predicate, ^{  
        sharedInstance = [[self alloc] init];  
    });  
    return sharedInstance;  
}  
  
// 应用注册消息推送  
- (void)RegisterAndOpenLocalNotification  
{  
    NSDictionary *infoDict = [[NSBundle mainBundle] infoDictionary];  
    BOOL isPush = [[infoDict objectForKey:@"Push"] boolValue];  
    if (isPush)  
    {  
        UIApplication *app = [UIApplication sharedApplication];  
        if ([app respondsToSelector:@selector(isRegisteredForRemoteNotifications)])  
        {  
            //IOS8  
            //创建UIUserNotificationSettings，并设置消息的显示类类型  
            UIUserNotificationSettings *notiSettings = [UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeBadge | UIUserNotificationTypeAlert | UIRemoteNotificationTypeSound) categories:nil];  
              
            [app registerUserNotificationSettings:notiSettings];  
            [app registerForRemoteNotifications];  
              
        }  
        else  
        { // ios7  
            [app registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge                                       |UIRemoteNotificationTypeSound                                      |UIRemoteNotificationTypeAlert)];  
        }  
          
        [self GetPushInfoByPlatformServer];  
    }  
}  
  
//  第二步：接收本地推送  
- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification*)notification  
{  
    NSLog(@"receive");  
    //    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"iWeibo" message:notification.alertBody delegate:nil cancelButtonTitle:@"确定" otherButtonTitles:nil];  
    //    [alert show];  
    // 图标上的数字减1  
    application.applicationIconBadgeNumber -= 1;  
}  
  
// 获取推送的信息(根据自己需求的自由设定)  
- (void) GetPushInfoByPlatformServer  
{  
    HLWebApi *api = [[[HLWebApi alloc] init] autorelease];  
    NSString *pushInfo = [api getPushInfo];  
    NSArray *resultArr = [NSJSONSerialization JSONObjectWithData:[pushInfo dataUsingEncoding:NSUTF8StringEncoding] options:NSJSONReadingAllowFragments error:nil];  
    NSArray *psuhArray = [[NSArray alloc] init];  
    if (resultArr && ([resultArr count] != 0))  
    {  
        for (psuhArray in resultArr)  
        {  
            NSString *pushTitle = [psuhArray valueForKey:@"pushTitle"];  
            NSString *pushTime = [psuhArray valueForKey:@"pushTime"];  
            NSString *pushContent = [psuhArray valueForKey:@"pushContent"];  
            NSLog(@"pushTitle= %@\t pushTime= %@\t pushContent= %@ ", pushTitle, pushTime, pushContent);  
            NSInteger pushHour = [[pushTime substringWithRange:NSMakeRange(0,2)] integerValue];  
            NSInteger pushMiute = [[pushTime substringWithRange:NSMakeRange(3,2)] integerValue];  
              
            NSInteger deleaTime = [self MakeTime:pushHour Minute:pushMiute];  
            NSString *identifier = [[NSBundle mainBundle] bundleIdentifier];  
            NSString *key = [[NSString alloc ]initWithFormat:@"%@,%@", identifier, pushTime];  
              
            if ([self CheckNotificationUpdate:key pushContent:pushContent])  
            {  
                // 创建消息推送  
                [self CreateLocalNotificationByInfo:key deleaTime:deleaTime pushContent:pushContent];  
            }  
        }  
    }  
    else  
    {  
        // 没有推送消息，清除已有本地推送  
        [self RemoveAllLocalNotification];  
    }  
}  
  
- (void) CreateLocalNotificationByInfo:(NSString *)key deleaTime:(NSInteger)deleaTime pushContent:(NSString *)pushContent  
{  
    UILocalNotification *notification = [[[UILocalNotification alloc] init] autorelease];  
    //设置10秒之后  
    NSDate *pushDate = [NSDate dateWithTimeIntervalSinceNow:deleaTime];  
    if (notification != nil)  
    {  
        // 设置推送时间  
        notification.fireDate = pushDate;  
        // 设置时区  
        notification.timeZone = [NSTimeZone defaultTimeZone];  
        // 设置重复间隔  
        notification.repeatInterval = kCFCalendarUnitDay;  
        // 推送声音  
        notification.soundName = UILocalNotificationDefaultSoundName;  
        // 推送内容  
        notification.alertBody = pushContent;  
        //显示在icon上的红色圈中的数子  
        notification.applicationIconBadgeNumber = 1;  
        //设置userinfo 方便在之后需要撤销的时候使用  
        NSDictionary *info = [NSDictionary dictionaryWithObject:key forKey:@"key"];  
        notification.userInfo = info;  
    }  
      
    //添加推送到UIApplicatio  
    UIApplication *app = [UIApplication sharedApplication];  
    [app scheduleLocalNotification:notification];  
}  
  
// 判断推送消息是否有更新  
- (BOOL) CheckNotificationUpdate:(NSString *)key pushContent:(NSString *)pushContent  
{  
    UIApplication *app = [UIApplication sharedApplication];  
    //获取本地推送数组  
    NSArray *localArr = [app scheduledLocalNotifications];  
    NSString *deleteKey = [[NSString alloc] init];  
    if (localArr)  
    {  
        for (UILocalNotification *noti in localArr)  
        {  
            NSDictionary *dict = noti.userInfo;  
            if (dict)  
            {  
                NSString *inKey = [dict objectForKey:@"key"];  
                NSString *inBody = noti.alertBody;  
                if ([inKey isEqualToString:key])  
                {  
                    if ([inBody isEqualToString:pushContent])  
                    {  
                        return NO;  
                    }  
                }  
                deleteKey = inKey;  
            }  
        }  
    }  
    // key相同而内容不同没有去刷新它，偷个懒直接删除重建  
    if (deleteKey) {  
        [self RemoveLocalNotificationByKey:deleteKey];  
    }  
    return YES;  
}  
  
- (NSInteger) MakeTime:(NSInteger)pushHour Minute:(NSInteger)pushMiute  
{  
    //获取当前时间  
    NSDate *now = [NSDate date];  
    NSCalendar *calendar = [NSCalendar currentCalendar];  
    NSUInteger unitFlags = NSYearCalendarUnit | NSMonthCalendarUnit | NSDayCalendarUnit | NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;  
    NSDateComponents *dateComponent = [calendar components:unitFlags fromDate:now];  
    NSInteger hour = [dateComponent hour];  
    NSInteger minute = [dateComponent minute];  
    NSInteger deleaTime = (pushHour - hour)*3600 + (pushMiute - minute)*60;  
    if (deleaTime < 0)  
    {  
        deleaTime += 24*3600;  
    }  
    return deleaTime;  
}  
  
// 根据key删除推送  
- (void) RemoveLocalNotificationByKey:(NSString *)key  
{  
    UIApplication *app = [UIApplication sharedApplication];  
    //获取本地推送数组  
    NSArray *localArr = [app scheduledLocalNotifications];  
    //声明本地通知对象  
    UILocalNotification *localNoti = [[UILocalNotification alloc] init];  
      
    if (localArr)  
    {  
        for (UILocalNotification *noti in localArr)  
        {  
            NSDictionary *dict = noti.userInfo;  
            if (dict)  
            {  
                NSString *inKey = [dict objectForKey:@"key"];  
                if ([inKey isEqualToString:key])  
                {  
                    if (localNoti)  
                    {  
                        [localNoti release];  
                        localNoti = nil;  
                    }  
                    localNoti = [noti retain];  
                    break;  
                }  
            }  
        }  
          
        //判断是否找到已经存在的相同key的推送  
        if (!localNoti)  
        {  
            //不存在 初始化  
            localNoti = [[UILocalNotification alloc] init];  
        }  
          
        if (localNoti)  
        {  
            //不推送 取消推送  
            [app cancelLocalNotification:localNoti];  
            [localNoti release];  
            return;  
        }  
    }  
}  
  
- (void) RemoveAllLocalNotification  
{  
    UIApplication *app = [UIApplication sharedApplication];  
    NSArray *localArr = [app scheduledLocalNotifications];  
    if (localArr)  
    {  
        for (UILocalNotification *noti in localArr)  
        {  
            [noti release];  
            noti = nil;  
        }  
    }  
}  
@end  

2、如果需要远程服务器推送，还需要申请推送证书并生产公钥密码给远程服务器推送，远程推送要客户端获取设备的token给服务器，服务器根据Device Token推送相应的消息。
