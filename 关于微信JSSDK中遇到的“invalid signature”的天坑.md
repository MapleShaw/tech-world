![](/post-img/jssdk/cover.jpeg)

## 产品背景：

产品哥哥要开发一个在微信中打开的需要获取用户当前地理位置的功能的页面，于是便开始了与微信JSSDK的苍蝇与大便的故事。

## 技术背景：

Vue主打，router使用history模式，外加微信JSSDK套餐，实现定位功能。

## 正文：

wx.config()这块网上的信息太多了，基本的配置这里绕过，直接说这次遇到的最坑的问题 ---- invalid signature

排查了各种情况总是找不出原因，而且神奇的是在安卓上可以正常获取位置，就只是在ios上一直“invalid signature”，打印出来的当前url跟签名的url也明明是一致的，为什么还是签名有问题呢！！？？？

页面的结构如下：

SPA：

* 页面A
* 页面B

非常简单，整个应用从A进入，然后跳转到B，B需要获取位置，我也只是在B里面配置微信的JSSDK，然后就遇到了前面所说的问题。然后！！关键来了！！在某一次调试的时候，我就直接从B页面刷新了，然后就可以了！！！！！What the fuk？！！

从B刷新加载的每一次都是那么丝滑，而从A到B，每一次都被枪毙，所以抱着这个问题，我来到了这个新世界 ---- 关于html5-History模式在微信浏览器内的问题

原来，是酱紫？！！

IOS：微信IOS版，微信安卓版，每次切换路由，SPA的url是不会变的，发起签名请求的url参数必须是当前页面的url就是最初进入页面时的url

Android：微信安卓版，每次切换路由，SPA的url是会变的，发起签名请求的url参数必须是当前页面的url(不是最初进入页面时的)

坑爹啊！亏我把请求签名的url跟当前页面的url（location.href）对比明明是一样的！

## 解决方案：

全局存储进入SPA的url（window.entryUrl），Android不变，依旧是获取当前页面的url，IOS就使用window.entryUrl，然后签名，done！

	// 记录进入app的url，后面微信sdk
	if (window.entryUrl === '') {
		window.entryUrl = location.href.split('#')[0]
	}
	// 进行签名的时候
	url: isAndroid() ? location.href.split('#')[0] : window.entryUrl


题外话：貌似看到有人在处理微信分享的时候也有类似的问题，有时间可以好好研究下～