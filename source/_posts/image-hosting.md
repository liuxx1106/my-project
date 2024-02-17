---
date: 2023-09-12 21:00:00
title: Hexo博客搭建之图床的最佳实践
tags: [图床, 博客搭建]
categories: [博客搭建]
index_img: https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/image-hosting.png   # 封面图
# banner_img: /img/post_banner.jpg  # 文章顶部大图
---

### 前言：为什么使用图床？

相较于冗长乏味的文字堆砌，读者往往更喜欢图文并茂、生动有趣的文章。因此好的图片往往能起到画龙点睛的作用。
我们知道，在hexo中（在一半web项目中也适用），添加图片一般有两种形式：

- 存在项目public目录下
- 存在远程仓库

当我们有大量的图片时，如果存在本地目录下，不仅造成项目臃肿，而且加载速度也不快，
所以，我们会考虑把图片存在远程服务器，也就是所说的图床。

### 方案选择

#### 方案一：Github仓库 + Picgo + jsDelivr CDN

由于最近国内将jsDelivrCDN给墙了，无法通过其对Github进行加速，这种方案目前暂时不可行，此处

介绍下具体操作即可：

1. 创建Github公共仓库，名字随便，注意一定要是公共的，否则别人无法访问到图片资源。

2. 下载 Picgo

> PicGo: 一个用于快速上传图片并获取图片 URL 链接的工具

3. 将Github创建仓库信息配置到Picgo中。打开Picgo，点击图床设置，选择Github图床，填写如下信息。

![picgo配置github](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/image-hosting1.png)

仓库名写你之前创建好的仓库名

分支填现有的分支即可

其中的Token可以在Github - Settings - Personal access tokens 中生成，只选择repo选项就可以

自定义域名这里填写：<https://cdn.jsdelivr.net/gh/用户名/仓库名，因为我们采用了jsDelivr> CDN进行加速（目前用不了）

最后保存为默认图床即可

#### 方案二：阿里云对象存储 + Picgo

谈到云服务器首先想到的就是阿里云了，目前阿里云是国内最好的云上服务解决方案提供商。

我们的项目中接触和应用和比较多，那么今天我们就先来实践下阿里云对象存储oss

1. 登录阿里云网站->进入工作台页面->搜索对象存储->点击立即开通->管理控制台->bucket列表

![picgo配置aliyunoss1](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/image-hosting2.png)

2. 进入页面，点击创建bucket.

- Bucket名称和地域必填。地域选择一个距离自己近一些的地方。
- 读写权限选择为公共读，其余均默认

![picgo配置aliyunoss2](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/image-hosting3.png)

点击网页右上角的头像，再点击AccessKey管理，进入该页面
点击创建AccessKey，将创建号的key复制保存下来，之后在Picgo上需要用到

![picgo配置aliyunoss3](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/image-hosting4.png)

3. 打开Picgo，点击图床设置，选择阿里云OSS，填写如下信息。

![picgo配置aliyunoss4](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/image-hosting5.png)

### 总结

结合个人喜好和使用经验，我还是更推荐方案三作为我们图床实践的最佳选择

就算后期图片访问量变大，也仅需要充一点钱，大概一年40GB只需要9块钱，非常划算。

### 结合Typora使用

当我们成功搭建好图床之后，每次写Markdown文档时，都需要先截图，再保存，然后手动打开Picgo完成上传，最后将图片地址复制到Markdown文档中。

如何做到更加高效地上传图片到图床呢？

用Typora写笔记，只需要先截图，再粘贴到Markdown文档，即可直接跳过上传操作，Typora帮我们自动完成。

操作：打开Typora的设置，点击图像，进行如下设置

![picgo配置aliyunoss5](https://murphy-blog.oss-cn-hangzhou.aliyuncs.com/image-hosting6.png)

自此，便可以方便快捷地用Hexo写图文并茂的博客啦~
