---
title: hexo的next主题优化定制
date: 2017-05-01
tags: 
	- hexo
	- next
categories:
    - hexo
keywords: hexo,next,主题优化,next定制,页面阅读区宽度,紧凑首页
description: 首页每个博文展示之间的宽度太宽？我怎么样才能调窄一些让它看起来紧凑一些？看着net的布局中内容展示区太窄，如何定制next主题阅读器宽度？
---
刚开始使用next主题，需要慢慢折腾，在折腾过程中，遇到的问题记录下来，做个备忘。

调整首页博文之间的间距
---
### 博文间的距离 ###
想让首页的博文之间的间距缩小一些，看着紧凑些，如此处理呢？其实只需要修改themes\你的主题\source\css\_variables\base.styl文件内的$post-eof-margin-top和$post-eof-margin-bottom这两个值，影响的是博文之间分割线的上下间距，如果想紧凑就调小，如果想宽松就是放大。

### 博文描述与全文阅读之间的间距 ###
看着博文描述与全文阅读按钮之间的间距太宽？这里也是可以修改的，需要修改的文件路径是themes\你的主题\source\css\_common\components\post\post-button.styl，查找并调整。

	.post-button {
		margin-top: 60px; //默认60个像素，可以根据自己的需求调整大小
### 调整博文作者行和博文描述的间距 ###
感觉博文作者行与博文描述间的具体太宽？或者你感觉还不够？怎么办呢？其实只需要修改themes\你的主题\source\css\_common\components\post\post-meta.styl文件内的

	.posts-expand .post-meta {
		margin: 3px 0 60px 0; 
		//修改这里的第三个值的大小就可以实现修改博文作者行与博文描述间的距离
		//第一个值是干什么的？你也好奇？其实只是修改博文作者行与上面博文标题的间距的，你可以调整试一下
		color: $grey-dark;
		font-family: $font-family-posts;
		font-size: 12px;
		text-align: center;

调整博文阅读区域
---
我感觉博文阅读区域太窄了，看着不舒服，怎么办？其实很简单的，只需要修改themes\你的主题\source\css\_schemes\Pisces\_layout.styl内的

	//这一部分调整侧边栏博客描述以及导航信息
	.header {
		position: relative;
		margin: 0 auto;
		width: 70%; //直接设置百分比，你想调整多少就多少

	//这一部分调整博文以及目录信息
	.container .main-inner {
		width: 70%;

> 这里针对Pisces主题的设置，原来的设计思路是显示器小于1600像素时阅读区展示700px，大于1600像素时阅读区展示900px。其实这里需要在行宽以及阅读舒适度方面做一个折中，毕竟单独的设置宽度太宽，阅读起来也不是很友好不是？ 就个人而言，太宽有时候会看串行啊。

在侧边栏添加邮箱
--
在**主题配置文件** source/_data/next.yml 中添加social配置：

	# Social links
	social:
	  Email: mailto:gamesdoa@gmail.com
	
	social_icons:
	  enable: true
	  Email: envelope

其他的还在慢慢折腾，有其他问题随时补充。