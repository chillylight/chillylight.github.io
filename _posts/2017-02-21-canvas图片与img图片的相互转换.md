---
layout:     post
title:      canvas图片与img图片的相互转换
subtitle:   canvas图片与img图片的转换
date:       2017-02-21
author:     chillylight
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - canvas转图片
    - 图片转canvas
---

### canvas图片与img图片的相互转换

最近在一个项目中，遇到了一个问题，需要把生成的canvas形式的二维码转换为图片，可以长按识别，保存等。查找了一些资料归纳总结了一些知识。
默认在jq库里进行，引入jquery.qrcode.min.js库，将canvas图片转化为img图片，代码如下，

```
<body>
    <div id="cans"></div>
    <div id="img"></div>
</body>
<script>
//生成canvas形式的二维码
$("#cans").qrcode({
    width:150,
    height:150,
    text:'http://www.cnblogs.com/dxzg/p/6424855.html'//需要生成的内容
    });
    
//从 canvas 提取图片 image  
function convertCanvasToImage(canvas) {  
    //新Image对象，可以理解为DOM  
    var image = new Image();  
    // canvas.toDataURL 返回的是一串Base64编码的URL
    // 指定格式 PNG  
    image.src = canvas.toDataURL("image/png");  
    return image;  
}  

//获取网页中的canvas对象  
var mycans=$('canvas')[0];   
//调用convertCanvasToImage函数将canvas转化为img形式   
var img=convertCanvasToImage(mycans);  
//将img插入容器 
$('#img').append(img); 
</script>
```

同理也可以将图片转换为canvas，转换函数如下：

```
// 把image 转换为 canvas对象  
function convertImageToCanvas(image) {  
    // 创建canvas DOM元素，并设置其宽高和图片一样   
    var canvas = document.createElement("canvas");  
    canvas.width = image.width;  
    canvas.height = image.height;  
    // 坐标(0,0) 表示从此处开始绘制，相当于偏移。  
    canvas.getContext("2d").drawImage(image, 0, 0);    
    return canvas;  
}
```