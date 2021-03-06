---
title: WIFI一键连接 iOS端 Configuration Profile 方式
date: 2017-06-15 19:39:00
tags: iOS WIFI Profile
---

{% blockquote %}
去年今日此门中,
人面桃花相映红.
人面不知何处去,
桃花依旧笑春风.
{% endblockquote %}

## 一、创建Configuration Profile

1.安装`Apple Configurator`工具

2.打开软件，工具栏：`文件` - `新建描述文件`

3.通用界面配置必备信息

（此处，配置文件的标识符相同，安装时，会覆盖安装）

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi1.png" height="300" width="400" />

4.左侧选择 Wi-Fi 项，进行 Wifi 信息配置

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi2.png" height="300" width="400" />

5.上述配置完成后，工具栏：`文件` - `存储`， 保存位置就会有一个 wifi.mobileconfig 文件

<img src="https://github.com/MoWangChen/Blog/raw/master/ScreenShot/wifi3.png" height="80" width="80" />

## 二、安装Configuration Profile
           
There are five ways to deploy configuration profiles:
```
 1.Using Apple Configurator 2, available in the App Store
 2.In an email message
 3.On a webpage
 4.Using over-the air configuration as described in Over-the-Air Profile Delivery and Configuration
 5.Over the air using a Mobile Device Management Server
```
```
https://developer.apple.com/library/ios/featuredarticles/iPhoneConfigurationProfileRef/Introduction/Introduction.html
```
     
上述文档中，苹果提供了五种方式安装配置文件。
	
测试时，使用了 加载网页的形式，自己创建了一个配置文件，委托 后端同学 放入后台服务，得到配置文件的下载链接。

   ```    
   [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"File_URL"]];
   ```

执行时，会调用系统safari加载下载链接，并自动转入系统设置，安装文件。安装完成，系统自动匹配wifi。
(测试：app里使用UIWevView控件加载下载链接，不会转入系统设置，无法完成安装）


