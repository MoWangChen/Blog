---
title: __Block浅谈
date: 2018-01-21 16:23:00
tags: iOS  block
---

{% blockquote %}
常将有日思无日,
莫待无时思有时.
{% endblockquote %}


很多同学在面试中都有可能被问到`__block`的问题, 与`__weak`的比较,或者是其作用.`__block`都是配合block体使用的,目的是让block内部能修改外部变量的值.

举个例子:

{% codeblock lang:objc %}
- (void)testBlock
{
    typedef void (^Block)(void);
    
    __block int i = 10;
    Block block = ^(){
        NSLog(@"inner block 1 -- %d", i);
        i = 20;
        NSLog(@"inner block 2 -- %d", i);
    };
    i = 30;
    block();
    NSLog(@"outer block -- %d", i);
}
{% endcodeblock %}

上述代码输出结果:

{% codeblock lang:objc %}
2018-01-21 15:43:03.378546+0800 Block[67824:5612019] inner block 1 -- 30
2018-01-21 15:43:03.378726+0800 Block[67824:5612019] inner block 2 -- 20
2018-01-21 15:43:03.378845+0800 Block[67824:5612019] outer block -- 20
{% endcodeblock %}

根据输出结果,`__block`确实起到作用,使变量在block内部被重新赋值.
`__block`修饰符是如何起作用的,通过下面的例子,了解如何不使用`__block`关键字使block内部修改外部变量值,来理解block的作用过程.

{% codeblock lang:objc %}
- (void)testBlock_variant
{
    typedef void (^Block)(void);
    
    int i = 10;
    int * i_block = &i;   // 定义指针变量 i_block 指向 i 变量的地址
    Block block = ^(){
        NSLog(@"inner block 1 -- %d", * i_block); // * i_block 取地址的值
        * i_block = 20; // 对 i_block对应地址的值进行修改
        NSLog(@"inner block 2 -- %d", * i_block);
    };
    i = 30;
    block();
    NSLog(@"outer block -- %d", i);
}
{% endcodeblock %}

上述代码输出结果:

{% codeblock lang:objc %}
2018-01-21 15:51:18.395184+0800 Block[67927:5624382] inner block 1 -- 30
2018-01-21 15:51:18.395344+0800 Block[67927:5624382] inner block 2 -- 20
2018-01-21 15:51:18.395448+0800 Block[67927:5624382] outer block -- 20
{% endcodeblock %}

与`__block`修饰效果一致.

[Demo下载地址 Block](https://github.com/MoWangChen/minutia-Experiment.git)

如有疑问或错误,欢迎大家批评指正.
