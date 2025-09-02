+++
title = 'CSS3浮动与定位'
date = 2024-11-25T19:02:29+08:00
draft = true
categories = [ "CSS" ]
tags = [ "css" ]
+++

## 浮动

### 浮动的基本概念

![alt text](image-243.png)

![alt text](image-244.png)

![alt text](image-275.png)
![alt text](image-276.png)
![alt text](image-277.png)

![alt text](image-278.png)
 
![alt text](image-279.png)

### 使用浮动实现网页布局

![alt text](image-280.png)
![alt text](image-281.png)

### BFC规范和浏览器差异

![alt text](image-282.png)
![alt text](image-283.png)

代码：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .box{
            width: 400px;
            /* 注意这里没有设置 height，再给两个子盒子设置浮动之后，父盒子变形了 */
            border: 10px solid #000;
        }

        .box .c1 {
            width: 200px;
            height: 200px;
            background-color: orange;
            float: left;
        }

        .box .c2 {
            width: 200px;
            height: 200px;
            background-color: blue;
            float: left;
        }
    </style>
</head>
<body>
    <div class="box">
        <div class="c1"></div>
        <div class="c2"></div>
    </div>
</body>
</html>
```

预览：
![alt text](image-284.png)

原因是父盒子没有形成BFC。
![alt text](image-285.png)

![alt text](image-286.png)

![alt text](image-287.png)

### 清除浮动

![alt text](image-289.png)

代码：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        p{
            float:left;
            width: 100px;
            height: 100px;
            margin-right: 20px;
            background-color: orange;
        }
    </style>
</head>
<body>
    <div>
        <p></p>
        <p></p>
    </div>
    <div>
        <p></p>
        <p></p>
    </div>
</body>
</html>
```

预览：
![alt text](image-290.png)

实际效果显示在了一行上，原因是div没有设置任何的宽度和高度，没有形成BFC，也就没有出现预期的两行，所以后面的div也堆上去了。

这也就是没有清除浮动。

一种方式是给div设置高度，但有可能盒子的高度是动态的，我们不确定盒子有多高，比如用户的帖子，留言等，所以这时为盒子设置高度并不是很好的方式。

![alt text](image-291.png)

![alt text](image-292.png)

![alt text](image-293.png)

![alt text](image-294.png)

## 定位

### 相对定位

![alt text](image-288.png)
![alt text](image-295.png)
![alt text](image-296.png)

用途
![alt text](image-297.png)

### 绝对定位

![alt text](image-298.png)
![alt text](image-299.png)
![alt text](image-300.png)

![alt text](image-301.png)
![alt text](image-302.png)

**绝对定位的盒子垂直居中**
![alt text](image-303.png)

**堆叠顺序 z-index 属性**
![alt text](image-304.png)

**绝对定位的元素可以设置宽度和高度**

![ ](image-305.png)

### 固定定位

![alt text](image-306.png)

![alt text](image-307.png)

![alt text](image-308.png)

## 边框

### 边框的三要素

**border**
![alt text](image-309.png)

![alt text](image-310.png)
![alt text](image-311.png)


### 四个方向的边框

![alt text](image-312.png)
![alt text](image-313.png)
![alt text](image-314.png)
![alt text](image-315.png)

## 圆角

![alt text](image-319.png)
![alt text](image-320.png)
![alt text](image-321.png)

![alt text](image-322.png)
![alt text](image-323.png)

## 盒子阴影

![alt text](image-316.png)
![alt text](image-317.png)
![  ](image-318.png)