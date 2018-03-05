---
title: 位移枚举
date: 2017-03-21 16:23:00
tags: iOS Enum
---

{% blockquote %}
多少祸端起于觊觎别人的生活.
{% endblockquote %}

## 位移枚举


### 目录

 - [问题引入](#1)
 - [问题解析](#2)
 - [概念讲解](3)
 - [问题解决办法(位移枚举)](#4)
 - [优化方法](#5)

### 问题引入

假定有一项考试,考试内容为辨别左、右两个方向,辨认正确得一分.写出考试和得分方法.

### 问题解析

{% codeblock lang:objc %}
@interface Test : NSObject

@property (nonatomic, assign) NSInteger count;

@end

@implementation Test

// 辨认方向
- (void)recognizeLeft:(BOOL)isLeft
{
    if (isLeft) {
        _count += 1;
    }
}

- (void)recognizeRight:(BOOL)isRight
{
    if (isRight) {
        _count += 1;
    }
}

// 得分
- (NSInteger)testGrade
{
    return _count;
}

@end

{% endcodeblock %}

上述为最直接的方法,先调用辨认方向的方法,后查看成绩.

如果此时左、右代表两大块;左细分为左上,正左,左下;同理,右细分为右上,正右,右下.并且左上,左下, 右上,右下,辨认分值为2分.

那此时,`recognizeLeft `, `recognizeRight `各自变换成细分的三个方法,虽然能解决,但需要增加很多辨认方法,不是一个好的解决方式.

此时,把左(isLeft),右(isRight) 从BOOL值,转成枚举值,可解决上面问题.

{% codeblock lang:objc %}
typedef NS_ENUM (NSInteger,LeftType){
    LeftTypeUnknown,    // 不辨认左方向
    LeftTypeTop,        // 左上方
    LeftTypeMiddle,     // 正左方
    LeftTypeBottom      // 左下方
};

typedef NS_ENUM (NSInteger,RightType){
    RightTypeUnknown,   // 不辨认右方向
    RightTypeTop,       // 右上方
    RightTypeMiddle,    // 正右方
    RightTypeBottom     // 右下方
};
{% endcodeblock %}

原来的`recognizeLeft `, `recognizeRight `修改:

{% codeblock lang:objc %}
// 辨认方向
- (void)recognizeLeft:(LeftType)leftType
{
    if (leftType == LeftTypeTop) {
        
        _count += 2;
        
    }else if (leftType == LeftTypeMiddle) {
    
        _count += 1;
        
    }else if (leftType == LeftTypeBottom) {
    
        _count += 2;
        
    }
}

- (void)recognizeRight:(RightType)rightType
{
    if (rightType == RightTypeTop) {
        
        _count += 2;
        
    }else if (rightType == RightTypeMiddle) {
    
        _count += 1;
        
    }else if (rightType == RightTypeBottom) {
    
        _count += 2;
    }
}
{% endcodeblock %}

这样,当辨认左上方时,若能辨认则传`LeftTypeTop`,不能则传`LeftTypeUnknown`或不调用该方法.其他各方向同理.这样存在的问题是,虽然方法数没增加,但是调用次数增多了.

下面,引入位移枚举的方法讲解.


### 概念讲解

位移枚举的定义:

{% codeblock lang:objc %}
typedef NS_OPTIONS (NSInteger,EnumType){
    EnumTypeValue1      = 1 << 0, // 0b00000001
    EnumTypeValue2      = 1 << 1, // 0b00000010
    EnumTypeValue3      = 1 << 2, // 0b00000100
    EnumTypeValue4      = 1 << 3  // 0b00001000
};
{% endcodeblock %}

### 问题解决办法(位移枚举)

{% codeblock lang:objc %}
typedef NS_OPTIONS (NSInteger,LeftType){
    LeftTypeUnknown     = 1 << 0,   // 不辨认左方向
    LeftTypeTop         = 1 << 1,   // 左上方
    LeftTypeMiddle      = 1 << 2,   // 正左方
    LeftTypeBottom      = 1 << 3    // 左下方
};

typedef NS_OPTIONS (NSInteger,RightType){
    RightTypeUnknown    = 1 << 0,   // 不辨认右方向
    RightTypeTop        = 1 << 1,   // 右上方
    RightTypeMiddle     = 1 << 2,   // 正右方
    RightTypeBottom     = 1 << 3    // 右下方
};
{% endcodeblock %}

{% codeblock lang:objc %}
// 辨认方向
- (void)recognizeLeft:(LeftType)leftType
{
    if (leftType & LeftTypeTop) {
        
        _count += 2;
        
    }
    if (leftType & LeftTypeMiddle) {
    
        _count += 1;
        
    }
    if (leftType & LeftTypeBottom) {
    
        _count += 2;
        
    }
}

- (void)recognizeRight:(RightType)rightType
{
    if (rightType & RightTypeTop) {
        
        _count += 2;
        
    }
    if (rightType & RightTypeMiddle) {
    
        _count += 1;
        
    }
    if (rightType & RightTypeBottom) {
    
        _count += 2;
    }
}
{% endcodeblock %}

这样,我们调用的时候,就可以先赋值好枚举变量,调用辨认方法就可以了.

{% codeblock lang:objc %}
// 某人,左上,正左,左下,右上,右下都能辨认
- (void)test
{
    LeftType leftType = LeftTypeTop | LeftTypeMiddle | LeftTypeBottom;
    [self recognizeLeft:leftType];
    
    RightType rightType = RightTypeTop | RightTypeBottom;
    [self recognizeRight:rightType];
}
{% endcodeblock %}


还是需要调用左,右两个方法,有了位移枚举,可以合并成一个方法.

### 优化方法

左、右枚举,合并

{% codeblock lang:objc %}
typedef NS_OPTIONS (NSInteger,OrientationType){
    OrientationTypeUnknown      = 1 << 0,    // 不辨认方向
    OrientationTypeLeftTop      = 1 << 1,    // 左上方
    OrientationTypeLeftMiddle   = 1 << 2,    // 正左方
    OrientationTypeLeftBottom   = 1 << 3,    // 左下方
    OrientationTypeRightTop     = 1 << 4,    // 右上方
    OrientationTypeRightMiddle  = 1 << 5,    // 正右方
    OrientationTypeRightBottom  = 1 << 6     // 右下方
};
{% endcodeblock %}

辨认方法合并

{% codeblock lang:objc %}
- (void)recognizeOrientation:(OrientationType)orientationType
{
    if (orientationType & OrientationTypeLeftTop) {
        
        _count += 2;
        
    }
    if (orientationType & OrientationTypeLeftMiddle) {
        
        _count += 1;
    
    }
    if (orientationType & OrientationTypeLeftBottom) {
        
        _count += 2;
        
    }
    if (orientationType & OrientationTypeRightTop) {
        
        _count += 2;
        
    }
    if (orientationType & OrientationTypeRightMiddle) {
        
        _count += 1;
        
    }
    if (orientationType & OrientationTypeRightBottom) {
        
        _count += 2;
        
    }
    if (orientationType & OrientationTypeUnknown) {
        
    }
}
{% endcodeblock %}

调用方法:

{% codeblock lang:objc %}
// 某人,左上,正左,左下,右上,右下都能辨认
- (void)test
{
    OrientationType orientationType = OrientationTypeLeftTop | OrientationTypeLeftMiddle | OrientationTypeLeftBottom | OrientationTypeRightTop | OrientationTypeRightBottom;
    [self recognizeOrientation:orientationType];
}

{% endcodeblock %}
