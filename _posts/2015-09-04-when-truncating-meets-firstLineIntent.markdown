---
layout:     post
title:      "When Truncating meets firstLineIndent."
subtitle:   "Success is not built on success. It's built on failure. It's built on frustration. Sometimes its built on catastrophe."
date:       2015-09-04 15:58:00
author:     "Nickolas"
header-img: "img/post-bg-05.jpg"
---



首先”恭喜”下, 你踩到了UILabel的一个坑.  
经过试验发现,多行文本做截断再设置首行缩进的话会发生首行就被截断的奇怪问题. [这篇文章](http://www.cocoabuilder.com/archive/cocoa/323722-nsattributedstring-mysteriously-truncated-too-soon.html)中也有朋友遇到了这个问题, 因为拿不到UIKit的源码, 只能猜测是UILabel的一个bug.

>I believe it’s just a quirk of the way the text system divides responsibility, which is in turn a consequence of the archeology of text capabilities. Paragraph styles  let you specify *one* line (with truncation) or *multiple* lines (without truncation). There is a separate option that truncates the last line of a multiple-line paragraph

猜测是UILabel只支持一行文本的截断或者不截断的多行文本...

我用了个比较hack的方法处理firstLineIndent, 通过在头部使用空格做出缩进的效果. 思路是使用较小的字体,填充缩进区域,字体越小效果越精确.

    {% highlight objective-c %}
    // 使用@“ “做firstLineIntent
    CGSize stringBoundingBox = [@“ “ sizeWithFont:[UIFont systemFontOfSize:10]];
    CGFloat unitWidth = stringBoundingBox.width;
    NSInteger spaceNum = (theWidthToIndent) / unitWidth;
    NSString *firstLineIndentString = [@“” stringByPaddingToLength:spaceNum withString:@“ “ startingAtIndex:0];
    NSMutableAttributedString *stringWithIntent = [originalString stringByAppendingString:fisrtLineIndentString];
    {% endhighlight %}

如果你需要比较复杂的多行文本样式, 还是老老实实的自己写UILabel. 或者使用[开源的代替方案](https://github.com/TTTAttributedLabel/TTTAttributedLabel).
