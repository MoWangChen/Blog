---
title: iOS 代码纠错案例
date: 2018-03-14 14:23:00
tags: iOS 
---

代码纠错:

{% codeblock lang:objc %}

- (void)setModel:(InfoModel *)model
{
    self.model = model;
    UIImage *image = [UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:model.url]]];
    [self.imageView setImage:image];
}

{% endcodeblock %}

--- 分割线 ---

{% codeblock lang:objc %}

- (void)viewDidLoad
{
    __weak typeof(self) weakSelf = self;
    self.nextController.block = ^(NSString *identifier){
        NSDictionary *parameters = @{@"parameter": identifier};
        [[ZHNetEngine engine] postWithURL:weakSelf.url parameters:parameters success:(^)(NSURLSessionTask *task, id responseData){
            // do something
        } fail:(^)(NSError *error, id responseData){
            // do something
        }];
        _nextPop = YES;
    };
}

{% endcodeblock %}