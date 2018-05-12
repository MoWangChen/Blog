---
title: iOS NEHotspotHelper使用
date: 2017-07-08 11:27:00
tags: iOS Enum
---

{% blockquote %}
杀不死你的,终将使你变得更强大.            --《小巨人》
{% endblockquote %}


## 一、简介
首先放上苹果官方文档：

```
https://developer.apple.com/reference/networkextension/nehotspothelper
```

`NEHotspotHelper` 是 `NetworkExtension.framework` 中与wifi连接相关的一个功能类。

 - `+ supportedNetworkInterfaces`
 可以获取到当前扫描到的WIFI列表，包含SSID，加密方式，信号强度信息。
 - `+ registerWithOptions:queue:handler:`  
注册当前app成为一个wifi辅助管理者，可以对指定的wifi，进行密码导入，并作字符串标记。

二、使用步骤

第一部分（权限申请） 

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi4.png" height="150" width="700" />

1.向苹果官方邮箱发权限申请邮件，使用自己的开发者账号邮箱申请，即代表所在的开发团队申请

2.邮件内容需要简单介绍APP的使用场景，以及为什么要使用NEHotspotHelper。 

3.发送完，就会收到一封苹果的回复，这时候去访问提示的那个网址，填写对应的权限申请信息。

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi6.png" height="450" width="700" />

4.访问`https://developer.apple.com/contact/network-extension/`，登入自己开发账号，会有自己和所在开发团队的信息

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi7.png" height="450" width="1000" />

5.填写对应的App信息，然后send。 

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi8.png" height="1600" width="900" />

6.邮箱会收到信息确认邮件，核实一下刚才填写的信息。如果没问题，就等大约三周时间，等苹果官方回复。

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi9.png" height="400" width="700" />

7.我这次 6月15申请，7月1号收到申请通过的邮件

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi10.png" height="400" width="700" />

8.去开发者中心配置开发证书,把`Wireless Accessory Configuration`，`iCloud`配置进去。
注意：配置文件，必须新建，在之前已存在的修改，后面工程运行会提示证书权限不匹配。

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi11.png" height="500" width="650" />


第二部分（项目工程配置）

1.`Target` - `Capabilities`  开启`iCloud` 

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi12.png" height="500" width="700" />

2.开启`Wireless Accessory Configuration`

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi13.png" height="150" width="700" />

3.上述步骤完成，工程会自动生成一个 .entitlements 权限文件，需要手动添加一项:`com.apple.developer.networking.HotspotHelper` ，设置它的Bool值为YES

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi14.png" height="170" width="700" />

3.在项目中配置`info.plist`文件
`UIBackgroundModels` 数组中增加 `network-authentication`

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi15.png" height="600" width="700" />

4.使用NEHotspotHelper

```
// wifiArray是含有wifi SSID和密码信息的数组
- (void)settingSSID:(NSArray *)wifiArray
{
    if (!IOS9) {
        return ;
    }
    // 此处设置的内容会在WiFi列表中每个WiFi下边展示出来
    NSDictionary* options = [NSDictionary dictionaryWithObjectsAndKeys:@"🔑WiFi一键连接",kNEHotspotHelperOptionDisplayName, nil];
    //线程
    dispatch_queue_t queue = dispatch_queue_create("com.testcompany.wifi", 0);
    
    //注册HotspotHelper,returnType为YES才说明可用
    BOOL returnType = [NEHotspotHelper registerWithOptions:options queue:queue handler: ^(NEHotspotHelperCommand * cmd) {
        NEHotspotNetwork* network;
        NSLog(@"cmd----%@", cmd);
        NSLog(@"commandType:----%ld", (long)cmd.commandType);
        NSLog(@"network---%@",cmd.network);
        NSLog(@"networkList---%@",cmd.networkList);
        
        //type为扫描wifi列表或评估
        if (cmd.commandType == kNEHotspotHelperCommandTypeEvaluate || cmd.commandType ==kNEHotspotHelperCommandTypeFilterScanList) {
            
            NSMutableArray *netArray = [NSMutableArray arrayWithCapacity:wifiArray.count];
            for (network  in cmd.networkList) {
                
                for (SSIDInfo *wifiItem in wifiArray) {
                    
                    if ([network.SSID isEqualToString:wifiItem.ssid]) {
                        
                        //遍历查找网络 连接wife
                        [network setConfidence:kNEHotspotHelperConfidenceHigh];
                        [network setPassword:wifiItem.password];
                        [netArray addObject:network];
                    }
                }
            }
            NEHotspotHelperResponse *response = [cmd createResponse:kNEHotspotHelperResultSuccess];
            //设置网络
            [response setNetworkList:netArray.copy];
            //发送
            [response deliver];
        }
    }];
    
    NSLog(@"result :%d", returnType);
}
```

程序后台运行，进入系统WIFI设置页时，就会走`NEHotspotHelperHandler`回调，对代码中设置的网络，会进行密码填充和标记。

三、摸索中遇到的坑
    
   1.待添加。。。
   
   ```
- (void)settingSSID:(NSArray *)wifiArray
{
    if (!IOS9) {
        return ;
    }
    // 此处设置的内容会在WiFi列表中每个WiFi下边展示出来
    NSDictionary* options = [NSDictionary dictionaryWithObjectsAndKeys:@"🔑WiFi一键连接",kNEHotspotHelperOptionDisplayName, nil];
    //线程
    dispatch_queue_t queue = dispatch_queue_create("com.testcompany.wifi", 0);
    
    //注册HotspotHelper,returnType为YES才说明可用
    BOOL returnType = [NEHotspotHelper registerWithOptions:options queue:queue handler: ^(NEHotspotHelperCommand * cmd) {
        NEHotspotNetwork* network;
        NSLog(@"cmd----%@", cmd);
        NSLog(@"commandType:----%ld", (long)cmd.commandType);
        NSLog(@"network---%@",cmd.network);
        NSLog(@"networkList---%@",cmd.networkList);
        
        //type为扫描wifi列表或评估
        if (cmd.commandType == kNEHotspotHelperCommandTypeEvaluate || cmd.commandType ==kNEHotspotHelperCommandTypeFilterScanList) {
            
            NSMutableArray *netArray = [NSMutableArray arrayWithCapacity:wifiArray.count];
            for (network  in cmd.networkList) {
                
                for (SSIDInfo *wifiItem in wifiArray) {
                    
                    if ([network.SSID isEqualToString:wifiItem.ssid]) {
                        
                        //遍历查找网络 连接wife
                        [network setConfidence:kNEHotspotHelperConfidenceHigh];
                        [network setPassword:wifiItem.password];
                        [netArray addObject:network];
                    }
                }
            }
            NEHotspotHelperResponse *response = [cmd createResponse:kNEHotspotHelperResultSuccess];
            //设置网络
            [response setNetworkList:netArray.copy];
            //发送
            [response deliver];
        }
    }];
    
    NSLog(@"result :%d", returnType);
}
   ```




