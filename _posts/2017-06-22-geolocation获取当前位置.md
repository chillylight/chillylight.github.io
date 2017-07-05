---
layout:     post
title:      geolocation地理定位
subtitle:   geolocation获取当前位置显示及计算两地距离
date:       2017-06-22
author:     chillylight
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - geolocation
    - 定位
    - H5定位
---


#### 获取当前经纬度

利用HTML5（以及基于JavaScript的地理定位API），可以很容易地在页面中访问位置信息，下面代码，就可以简单的获取当前位置信息：
```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
	<meta charset="UTF-8">
	<title>获取当前位置</title>
</head>
<body>
	<div id="location">
		你的位置将显示在这
	</div>
    <div id="distance">
            这是用来放距离的
    </div>
    <div id="map" style="width: 500px; height: 500px">
		这是用来放地图的
	</div>
	<script>
		/* 一旦浏览器加载页面，我们将调用函数getMyLocation */
		window.onload = getMyLocation;
		function getMyLocation() {
			/* 利用这个检查来确保浏览器支持地理定位API。如果navigator.geolocation对象存在，说明浏览器支持这个API */
			if (navigator.geolocation) {
				/* 如果浏览器支持这个API，则调用getCurrentPosition方法，并传入一个处理函数displayLocation. */
				navigator.geolocation.getCurrentPosition(displayLocation,,displayError);
			}else{
				/* 不支持地理定位时弹出提醒 */
				alert("你的浏览器不支持地理定位")
			}
		}

		/* 这个就是处理函数，浏览器得到一个位置时就会调用这个函数 */
		function displayLocation(position){
			/* 从position.coords对象得到位置的维度和经度 */
			var latitude = position.coords.latitude;
			var longitude = position.coords.longitude;
			var div = document.getElementById("location");
			/* 使用innerHTML将位置信息插入location<div>中 */
			div.innerHTML = "你的维度为："+latitude+"<br/>"+"你的经度为："+longitude;
		}
	</script>
</body>
</html>
```
用到地理定位时，并不是每次都能成功，所以可以创建一个小的诊断测试。
需要申明的是getCurrentPosition可以接收三个参数navigator.geolocation.getCurrentPosition(displaySuccess,displayError, options),其中成功回调，和失败回调是可选的。
在这里我们的诊断测试用的了失败回调函数,失败时调用displayError。
`navigator.geolocation.getCurrentPosition(displayLocation,displayError )`
现在要编写错误处理程序,geolocation将会向你的处理程序传入一个error对象，其中包含一个数值码，描述了它未能确定浏览器位置的原因。根据这个数值码还可以提供一个消息，包含有关这个错误的更多信息。
```
function displayError(error) {
    errorTypes = {
	0:"未知错误", /* 如果其他类型错误都不合适就会使用这个错误 */
    1:"用户拒绝使用定位", /* 你自己拒绝使用地理定位 */
	2:"位置不存在", /* 浏览器已经尽力了，但还是没能获取到你的位置 */
	3:"超时" /* 地理请求有一个内部超时设置，如果超出时间却还没有确定位置就好报出这个错误 */
    }
    var errorMessage = errorTypes[error.code];
    if (errorTypes == 0 || errorTypes ==2) {
	errorMessage = errorMessage+" "+error.message;
    }
    var div = document.getElementById("location");
	  div.innerHTML = errorMessage;
    }
```

#### 在百度地图上显示你的位置

获取到经纬度之后往往需要在地图上显示出来，这里调用百度地图,调用地图需要使用该公司的API接口，因此引入百度地图API接口。
`<script type="text/javascript" src="http://api.map.baidu.com/api?v=2.0&ak=密钥"></script>`
密钥得自己去[百度地图开发平台](http://lbsyun.baidu.com/)申请，不做商业用途不收费。
引入后在上面的displayLocation函数中增加下面代码：
```
    var bdPoint = new BMap.Point(longitude,latitude);
    var bm = new BMap.Map("map");
    bm.centerAndZoom(bdPoint,15);
    /* 添加位置标记 */
    var markergg = new BMap.Marker(bdPoint);
    bm.addOverlay(markergg);
```
便可得到如下地图，不过需要注意的是，台式浏览器上往往不能得到精确的位置，因为大多是根据ip来定位的，而且许多浏览器默认都禁止使用地理定位，不过使用新版的IE倒是可以。笔记本上有定位功能的开启后就较为精确，在有GPS的手机上，定位就更精确了。

![位置地图](http://images2015.cnblogs.com/blog/1101873/201706/1101873-20170623100144038-672730999.png)

以上地图来自台式电脑定位，误差很大。更多关于地图的操作可参考[百度地图web开发API文档](http://lbsyun.baidu.com/index.php?title=jspopular).

#### 计算你和别人之间的距离

在实际运用中，知道了自己的位置，知道了别人的位置，往往需要计算两个位置之间的距离。
因此接下来计算两点距离，要计算两个坐标之间的距离，基本都会用半正失公式，如下：
```
function computeDistance(startCoords,destCoords){
    var startLatRads=degreesToRadians(startCoords.latitude);
    var startLongRade=degreesToRadians(startCoords.longitude);
    var destLatRads=degreesToRadians(destCoords.latitude);
    var destLongRads=degreesToRadians(destCoords.longitude);
    var Radius=6371;
    var distance=Math.acos(Math.sin(startLatRads)*Math.sin(destLatRads)+Math.cos(startLatRads)*Math.cos(destLatRads)*Math.cos(startLongRade-destLongRads))*Radius;
    return distance;
    }
				
function degreesToRadians(degrees){
    var radians=(degrees*Math.PI)/180;
    return radians;
    }
```
以及对方的经纬度，这里随便写了一个
```
var ourCoords={
    latitude:30.2526,
    longitude:120.1656
};
```
然后还是在之前的displayLocation函数中继续增加下面代码：
```
/* 计算两点之间的距离 */
    var km=computeDistance(position.coords,ourCoords);
    var distance=document.getElementById("distance");
    distance.innerHTML="你的位置距离纬度为:"+ourCoords.latitude+"<br/>"+"经度为:"+ourCoords.longitude+"<br/>"+"的距离为"+km+"km";  
```
这样就可以得到两点之间的距离，我得到的是：

![距离](http://images2015.cnblogs.com/blog/1101873/201706/1101873-20170623105520554-194269987.png)

#### 补充
在iOS10 以上苹果对webkit定位权限进行了修改，所有定位请求的页面必须是https协议的。在http协议下通过html5原生定位接口会返回错误，无法正常定位。