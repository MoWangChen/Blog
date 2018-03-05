---
title: iOS变更工程文件目录引起missing file警告
date: 2017-05-15 18:35:00
tags: iOS 变更目录 警告
---

{% blockquote %}
斯人若彩虹,遇上方知有.
{% endblockquote %}

近期工作中做业务功能，创建类文件时，未指定文件目录，导致文件创建在根目录。显然很影响整个工程目录的结构，所以开始了文件的转移操作。

```
1.工程中，删除文件引用 Remove Reference

2.工程目录，类文件移动到指定目录中

3.工程中，拖动文件Copy items if needed
```

工程运行正常，但是出现了missing file警告，是由于Xcode自带的source control引起的

### 解决办法：

```
1.cd 到警告目录下

2.执行 git rm <文件名>

3.clean工程，重新运行
```

### 注意：

查资料，`Preference` - `Source Control` – 关闭`Enable Source Control` 可以解决此警告

经尝试，关闭`Source Control`可以解决此警告，但是工程文件目录下，原始目录文件又出现了，指定目录下也有一份文件。所以不建议使用此方法