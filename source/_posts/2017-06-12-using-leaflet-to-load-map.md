---
layout: post
title: "使用Leaflet插件在浏览器加载地图"
date: 2017-06-12 12:00:00
comments: true
categories: 
- Toolkit
tags:
- JavaScript
- Leaflet
- Map
- 腾讯地图
---

在公司实习的第一个项目就与地图和数据可视化相关，这让从来没有接触过地图API、加载工具以及相关开发的我一开始都有些手足无措。之后技术路线慢慢定下来，在浏览器中加载腾讯地图的瓦片地图，并在该地图上添加热力图等可视化图表。

在经过一番简单的调研之后，在浏览器加载地图无非是采用相应地图公司开发的JavaScript API，或者加载在线/离线瓦片地图。后者由于要使用第三方的插件或者自己开发的工具进行加载，所以实现起来较为复杂，而前者实现起来就比较傻瓜了。本文中主要使用腾讯地图作为实例使用地图，同时也会简单提及其他公司开发的地图。

<!-- more -->

#### 使用JavaScript API加载腾讯地图

使用[JavaScript API](http://lbs.qq.com/javascript_v2/index.html)在浏览器加载腾讯地图，方式非常简单，最重要就是在页面中加载相应的API的js文件，并初始化地图组件即可：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
		<title>简单地图</title>
		<style type="text/css">
			#container{
			    min-width:600px;
			    min-height:767px;
			}
		</style>
		<script charset="utf-8" src="http://map.qq.com/api/js?v=2.exp"></script>
		<script>
		window.onload = function(){
			function init() {	//初始化地图函数，自定义函数名init
		        //定义map变量，调用qq.maps.Map()构造函数，获取地图显示容器
		    	var map = new qq.maps.Map(document.getElementById("container"), {
             		// 地图的中心地理坐标：
             		center: new qq.maps.LatLng(39.916527,116.397128),
              	zoom:8                                                 
             });
		   }
	      //调用初始化函数地图
	      init();
		}
		</script>
	</head>
	<body>
		<!--定义地图显示容器-->
		<div id="container"></div>
	</body>
</html>
```

#### 瓦片地图

目前的流行的互联网瓦片地图种类有两种，一种是栅格地图瓦片，另一种是矢量地图瓦片。

栅格地图瓦片较为传统，是将矢量数据渲染成为256×256像素大小的图片，例如百度地图、高德地图、天地图和腾讯地图等，这些地图可以使用leaflet框架进行瓦片的加载。矢量地图瓦片是以json格式分块传输，在浏览器端利用前端框架将地图展示出来，mapbox和腾讯地图中的3D地图就是矢量地图瓦片，支持三维旋转等。

#### 使用leaflet.js

各个地图公司的产品本身拥有还比较完善的API和相关的插件和工具，比如除了地图加载之外的标志叠加、热力图叠加等。但是地图瓦片的加载目前比较流行和完善的工具就是[leaflet.js](http://leafletjs.com/)工具,它通过下载特定地图产品的瓦片（一般是256x256px大小），将相应的x、y坐标对应起来，展示在html文档的容器中。之前在网上搜资料，腾讯地图瓦片的资料也是凤毛麟角，跟公司地图部门的同学了解之后便开始了这个地图应用的开发。下面就用leaflet工具加载腾讯地图瓦片作为例子来展示如何进行地图瓦片的操作。

废话不多说，先上代码：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
		<title>简单地图</title>
		<style type="text/css">
			#container{
			    min-width:600px;
			    min-height:767px;
			}
		</style>
		<script charset="utf-8" src=""></script>
	</head>
	<body>
		<!--定义地图显示容器-->
		<div id="mapid"></div>
		<script>
			var mymap = L.map('mapid').setView([39.90469, 116.40717], 12);
			L.tileLayer('https://rt{s}.map.gtimg.com/realtimerender?z={z}&x={x}&y={y}&type=vector&style=0', {
	          subdomains: ["0","1","2","3"],
	          tms: true,
	          maxZoom:18,
	          minZoom:1,
	          detectRetina: true,
	          attribution: '&copy 滴滴出行'
	   		}).addTo(mymap);
		</script>
	</body>
</html>
```

上面代码中展示的就是在div容器中加载地图瓦片的实例，其中，最重要的就是腾讯地图瓦片的下载地址**[https://rt{s}.map.gtimg.com/realtimerender?z={z}&x={x}&y={y}&type=vector&style=0](https://rt{s}.map.gtimg.com/realtimerender?z={z}&x={x}&y={y}&type=vector&style=0)**，其中{s}是指subdomains，是地图瓦片的集群编号，可以传入字符串或数组，此处我们传入的是[“0”, “1”, “2”, “3”]；其次是{x}、{y}、{z}三个参数分别指的是瓦片的横坐标、纵坐标以及缩放等级。地图实例通过L.map(容器ID)来进行声明，通过L.tileLayer().addTo(地图实例)方法来进行加载，该方法中需要传入一个地图瓦片URL和一个参数对象作为参数。其中参数对象中的参数配置可以参考leaflet官网的文档进行配置。

地图加载完毕之后，可以在加载好的地图图层上继续添加所需要的标志、热力图等等。我们这里展示leaflet工具自带的热力图加载方法：

```javascript
L.heatLayer(data,{
	minOpacity: 0.6,
	maxZoom: 18,
	max: 500,
	radius: 12,
	blur: 0,
	gradient: {
	  1: '#00ffe4', 
	  0.7: '#0078ff', 
	  0.5:'#4285ff'
	}
}).addTo(mymap);
```

同样是在一样的容器里添加图层，参数有热力数据和参数对象两个，其中热力数据的数据结构是[纬度, 经度, 热度]该热力图可以通过设置透明度（minOpacity）、渐变色（gradient）、扩展度（blur）来改变热力图的样式。

在本文最基础的使用leaflet工具来加载腾讯地图瓦片之后，相信会接触更多有关地图网页应用的项目，之后也会不断地更新地图相关技术博客文章。