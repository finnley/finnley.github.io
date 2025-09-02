+++
title = 'HTML5语法与基础标签'
date = 2024-11-23T15:56:41+08:00
draft = true
categories = [ "HTML" ]
tags = [ "html" ]
+++

## 前置知识

### 互联网基本原理

- 在本地开发，在服务器共享

![alt text](/images/fe/image.png)

### HTTP协议

![alt text](/images/fe/image2.png)
![alt text](/images/fe/image3.png)

### 什么是前端、后端？

## HTML5基础入门

### 创建第一个网页

![alt text](image-1.png)
![alt text](image-2.png)
![alt text](image-3.png)
![](image-4.png)

### 浏览网页的方法

![alt text](image-6.png)
![alt text](image-8.png)

### 认识HTML骨架

![alt text](image-9.png)
![alt text](image-10.png)
![alt text](image-11.png)

![alt text](image-12.png)
![alt text](image-13.png)
![alt text](image-14.png)

### 字符集

![alt text](image-15.png)
![alt text](image-16.png)
![alt text](image-17.png)
![alt text](image-18.png)

![alt text](image-19.png)

### title、关键词及页面描述

![alt text](image-20.png)
![alt text](image-21.png)
![alt text](image-22.png)
![alt text](image-23.png)
![alt text](image-24.png)
![alt text](image-25.png)



### 认识标签
![alt text](image-26.png)

![alt text](image-27.png)

![alt text](image-30.png)
![alt text](image-31.png)
![alt text](image-32.png)

## HTML5基本标签

### 标题和段落标签

![alt text](image-33.png)
![ ](image-34.png)

![alt text](image-35.png)

### div标签

![alt text](image-36.png)
![alt text](image-37.png)
![alt text](image-45.png)

## HTML5特性

![alt text](image-38.png)
![alt text](image-39.png)
![alt text](image-40.png)
![alt text](image-41.png)

## 开发流程

![alt text](image-42.png)
![alt text](image-43.png)
![alt text](image-44.png)

## 列表标签

![alt text](image-46.png)

### 无序列表

![ ](image-47.png)
![alt text](image-49.png)
![ ](image-50.png)
![alt text](image-51.png)
![alt text](image-52.png)
![alt text](image-53.png)
![alt text](image-54.png)
![alt text](image-55.png)

### 有序列表

![alt text](image-56.png)
![alt text](image-57.png)
![alt text](image-58.png)
![alt text](image-59.png)
![alt text](image-60.png)
![alt text](image-61.png)

**无序列表的type属性**

> 说明：HTML5中被废弃，但是学习时可以作为了解。

* 无序列表有 type 属性，可以定义前导符号的样式，HTML5中被废弃，建议使用CSS替代

值                  | 描述               
-------             | ---
disc                | 默认值，实心圆
circle              | 空心圆
square              | 实心方块

代码1：
```html
<h1>购买商品</h1>
<ul type="circle">
    <li>牛奶</li>
    <li>面包</li>
    <li>咖啡</li>
</ul>
```
   
预览：
![alt text](image-72.png)


代码2：
```html
<h1>购买商品</h1>
<ul type="disc">
    <li>牛奶</li>
    <li>面包</li>
    <li>咖啡</li>
</ul>
```
   
预览：
![alt text](image-73.png)

代码3：
```html
<h1>购买商品</h1>
<ul type="square">
    <li>牛奶</li>
    <li>面包</li>
    <li>咖啡</li>
</ul>
```
   
预览：
![alt text](image-74.png)


### 定义列表
  
![alt text](image-62.png)

![ alt text](image-63.png)
![alt text](image-64.png)
![alt text](image-65.png)
![  ](image-66.png)
![alt text](image-67.png)

### 列表嵌套

**是什么？**
`<li></li>`标签内可以防止其他标签，包括放置 `<li></li>` 标签，这就形成了列表的嵌套。
![ ](image-70.png)

**示例**

代码：
```html
<ul>
    <li>
        <h2>上海市</h2>
        <ul>
            <li>松江区</li>
            <li>静安区</li>
            <li>闸北区</li>
        </ul>
    </li>
    <li>
        <h2>江苏省</h2>
        <ul>
            <li>盐城市</li>
            <li>苏州市</li>
            <li>南京市</li>
        </ul>
    </li>
</ul>
```

预览：
![alt text](image-71.png)

## 多媒体与语义化标签

### 图片标签

1、使用 `<img>` 标签在网页职工插入图片

例：<img src="images/gugong.jpg">

- `img` 是 image（图片）的缩写。 
- `src` 是 source（来源）的缩写
- `images/gugong.jpg` 是图片的存储目录和完整文件名。

**注意**

- 图片必须存在于项目文件夹中，一般将图片保存在项目中的 `images` 子文件夹中。
- 图片路径必须正确。
- 图片并非真正插入到网页中，只是被引入到了网页中。所以将来上传服务器时，一定要将图片一起上传到服务器。故需要将图片放在项目目录下一起整体上传。

代码：
```html
<h1>北京景点</h1>
<p><img src="images/gugong.jpg" alt=""></p>
<p><img src="images/niaochao.jpg" alt=""></p>
```

预览：
![alt text](image-76.png)

2、<img> 标签 `alt` 属性

alt 属性是 alternate “替代品”的缩写，它是对图像的文本的描述，非强制选项。其作用是当由于某种原因导致无法加载图像，页面上会显示alt属性中的描述内容。另一个作用就是供视力不方便的人在使用网页朗读器是朗读alt中的描述内容。

例：<img src="images/gugong.jpg" alt="故宫角楼">

代码：
```html
<h1>北京景点</h1>
<p><img src="images/gugong1.jpg" alt="故宫角楼"></p>
<p><img src="images/niaochao1.jpg" alt="鸟巢体育场"></p>
```

预览：
![alt text](image-77.png)

3、<img> 标签 width、height 属性

width、height 可分别设置图片的宽度和高度，单位为像素，可省略。

例：<img src="images/gugong.jpg" alt="故宫角楼" width="300">

- 这里只设置width属性，没有设置 height 属性，如果省略了其中一个，则会按原始比例缩放图片。

4、网页支持的图片格式

![alt text](image-78.png)

5、相对路径和绝对路径

![alt text](image-79.png)
![alt text](image-80.png)

### 超链接 

![ ](image-81.png)

使用 <a> 标签制作超级链接

例：<a href="2.html">去第二个网页</a>

- a 是 anchor（锚）的首字母。
- href 是 hypertext reference 超文本引用

![alt text](image-82.png)

1、href属性支持相对路径和绝对路径

![alt text](image-83.png)

2、
![alt text](image-84.png)

3、
![alt text](image-85.png)

4、
![alt text](image-86.png)

### 页面内锚点

![alt text](image-87.png)
![ ](image-88.png)

代码：
```bash
<h1>旅游经历</h1>
<h2>北京</h2>
<p><img src="images/beijing/1.jpg" alt=""></p>
<p><img src="images/beijing/2.jpg" alt=""></p>
<p><img src="images/beijing/3.jpg" alt=""></p>
<p><img src="images/beijing/4.jpg" alt=""></p>
<!-- 在这里添加一个id为wuxi的锚点 -->
<h2 id="wuxi">无锡</h2>
<p><img src="images/wuxi/1.jpg" alt=""></p>
<p><img src="images/wuxi/2.jpg" alt=""></p>
<p><img src="images/wuxi/3.jpg" alt=""></p>
<p><img src="images/wuxi/4.jpg" alt=""></p>
```

然后浏览想直接跳转到无锡的锚点，只需要在url后面携带上“#wuxi”即可。

还有个特殊的锚点 “#top”，可以直接回到顶部。

![alt text](image-89.png)
![alt text](image-91.png)

### 音频

![alt text](image-92.png)
![alt text](image-93.png)
![alt text](image-94.png)
![alt text](image-95.png)


### 视频

![alt text](image-96.png)
![alt text](image-97.png)

## 语义化标签


### 区块标签
![alt text](image-98.png)

![alt text](image-99.png)

### 语义化标签

![alt text](image-100.png)
![alt text](image-101.png)
![alt text](image-102.png)

![alt text](image-103.png)

## 表单标签

![alt text](image-104.png)
 
### 表单的创建

![alt text](image-105.png)

### 基本控件

**单行文本框**
![alt text](image-106.png)

![alt text](image-107.png)

![alt text](image-108.png)

**单选按钮**
![alt text](image-109.png)
![alt text](image-110.png)

![alt text](image-111.png)
![alt text](image-112.png)

**复选框**
![alt text](image-113.png)

**密码框**

![alt text](image-114.png)

**下拉菜单**

![alt text](image-115.png)

**多行文本框**

![alt text](image-116.png)

**三种按钮**

![alt text](image-117.png)

**小结**

![alt text](image-118.png)

### HTML5新增表单控件

![alt text](image-119.png)

![alt text](image-120.png)

## 表格标签

### 认识表格


![alt text](image-121.png)
![alt text](image-122.png)
![alt text](image-123.png)


### 表格标签

![alt text](image-124.png)
![alt text](image-125.png)

![alt text](image-126.png)

![alt text](image-127.png)
![alt text](image-128.png)


### 单元格合并

![alt text](image-129.png)
![alt text](image-130.png)
![ ](image-131.png)

![alt text](image-132.png)

![alt text](image-133.png)
![alt text](image-134.png)

![alt text](image-135.png)

![alt text](image-136.png)

### 表格其他特性 

![alt text](image-137.png)

![alt text](image-138.png)