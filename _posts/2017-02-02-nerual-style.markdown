---
layout: post
title: "一个简单的nerual-style程序"
---
Nerual-style是一个简单的卷积神经网络应用的例子, 可以将两张图片分别作为content和style合成一张图片

## Input
Content Image:

![](https://js-image-assets.s3-ap-northeast-1.amazonaws.com/nerual-style/1-content.jpg)

Style Image:

![](https://js-image-assets.s3-ap-northeast-1.amazonaws.com/nerual-style/1-style.jpg)
## Output
![](https://js-image-assets.s3-ap-northeast-1.amazonaws.com/nerual-style/output.jpg)

本文参考的是[anishathalye/neural-style](https://github.com/anishathalye/neural-style), 大家也可以参考的简化版本[sjjianshen/nerual-style](https://github.com/sjjianshen/nerual-style)

# Steps
+ Loading content images and style images and resize style image according to the size of content image

{% highlight python %}
        content_image = scipy.misc.imread('1-content.jpg').astype(np.float)
        style_image = scipy.misc.imread('1-style.jpg').astype(np.float)
        style_image = scipy.misc.imresize(style_image, float(content_image.shape[1]) / style_image.shape[1])
{% endhighlight %}

+ 利用卷积神经网络从content和style图片中提取特征

这里我们用的是vgg net预先训练好的参数，vgg网络模型可以从此处下载imagenet-vgg-verydeep-19.mat](http://www.vlfeat.org/matconvnet/models/beta16/imagenet-vgg-verydeep-19.mat)

提取content图片特征

![](https://js-image-assets.s3-ap-northeast-1.amazonaws.com/nerual-style/content.png)

提取style特征

![](https://js-image-assets.s3-ap-northeast-1.amazonaws.com/nerual-style/style.png)

+ 使用平均分布随机初始化一张和content图片同样size的图片计算loss

![](https://js-image-assets.s3-ap-northeast-1.amazonaws.com/nerual-style/train.png)

![](https://js-image-assets.s3-ap-northeast-1.amazonaws.com/nerual-style/loss.png)

+ 循环优化随机图片像素值使Loss变小，最终获得mixed的图片
