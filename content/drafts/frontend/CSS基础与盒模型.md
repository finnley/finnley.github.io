+++
title = 'CSS基础与盒模型'
date = 2024-11-25T11:49:06+08:00
draft = true
categories = [ "CSS" ]
tags = [ "css" ]
+++

## CSS 简介

![alt text](image-139.png)
![alt text](image-140.png)
![alt text](image-141.png)
![alt text](image-142.png)
![alt text](image-143.png)
![alt text](image-144.png)
![alt text](image-145.png)
![alt text](image-146.png)
![alt text](image-147.png)
![alt text](image-148.png)
![alt text](image-149.png)

### CSS3书写位置

![alt text](image-150.png)
![alt text](image-151.png)
![alt text](image-152.png)
![alt text](image-153.png)

### CSS3基本语法

![alt text](image-154.png)
![alt text](image-155.png)
![alt text](image-156.png)
![alt text](image-157.png)

## 选择器

### 标签选择器

![alt text](image-158.png)
![alt text](image-159.png)
![alt text](image-160.png)

### id选择器

**id属性**

![alt text](image-161.png)
![alt text](image-162.png)


### class选择器
![alt text](image-163.png)
![alt text](image-164.png)
![alt text](image-165.png)
![alt text](image-167.png)

**原子类**
![alt text](image-168.png)
![alt text](image-169.png)
![alt text](image-170.png)

### 复合选择器

![alt text](image-171.png)

**后代选择器**
![alt text](image-172.png)
![alt text](image-174.png)
![alt text](image-175.png)

**交集选择器**
![alt text](image-176.png)

**并集选择器**
![alt text](image-177.png)
![alt text](image-178.png)

### 伪类

![alt text](image-179.png)
![alt text](image-180.png)

### 元素关系选择器

![alt text](image-181.png)
![alt text](image-182.png)
![alt text](image-183.png)

![alt text](image-184.png)
![alt text](image-185.png)
![alt text](image-186.png)

### 序号选择器

![alt text](image-187.png)

**:first-child**
![alt text](image-188.png)
![alt text](image-189.png)

**:last-child**
![alt text](image-190.png)

**:nth-child(?)**
![alt text](image-191.png)
![alt text](image-192.png)
![alt text](image-193.png)
![alt text](image-194.png)

**:nth-of-type(?)**

![alt text](image-195.png)
这里没有任何一个p被选中，它是失效的。

想要被选中需要使用下面用法：
![alt text](image-196.png)
![alt text](image-197.png)

![alt text](image-198.png)

### 属性选择器

![alt text](image-199.png)

### CSS3新增伪类

![alt text](image-200.png)

### 伪元素
![alt text](image-201.png)

![alt text](image-203.png)

![alt text](image-204.png)

![alt text](image-205.png)

### 层叠性和选择器权重计算

![alt text](image-206.png)

![alt text](image-207.png)

![alt text](image-209.png)
![alt text](image-210.png)
![alt text](image-211.png)


## 文本与字体属性 

### 常用文本样式属性

**color**

![alt text](image-212.png)
![alt text](image-213.png)
![alt text](image-214.png)
![alt text](image-215.png)
![alt text](image-217.png)

**font-size**

![alt text](image-218.png)

**font-weight**

![alt text](image-219.png)

**font-style**
![alt text](image-220.png)

**text-decoration**

![alt text](image-221.png)

### 字体属性

![alt text](image-222.png)
![alt text](image-223.png)
![alt text](image-225.png)

![alt text](image-226.png)
![alt text](image-227.png)

![alt text](image-228.png)

### 段落和行属性
![alt text](image-229.png)
![alt text](image-230.png)
![alt text](image-232.png)

![alt text](image-233.png)

**单行文本垂直居中**
![alt text](image-235.png)

**font合写属性**
![alt text](image-237.png)

 
### 继承性

![alt text](image-238.png)

![alt text](image-239.png)

![alt text](image-240.png)

![alt text](image-241.png)

![alt text](image-242.png)

## 盒模型

### 盒模型基本概念

**认识盒模型**

![alt text](image-246.png)
![alt text](image-247.png)

![alt text](image-248.png)
整个网页就可以看成是很多盒模型构成的

![alt text](image-249.png)

代码:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .box1 {
            /* 宽度，快捷键 w200 */
            width: 200px;
            /* 高度，快捷键 h200 */
            height: 200px;
            /* 背景颜色，快捷键 bgc */
            background-color: gold;
            /* 边框，快捷键 bd */
            border: 10px solid red;
            /* 内边距 */
            padding: 10px;
        }
    </style>
</head>
<body>
    <div class="box1">
        文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文字文
    </div>                             
</body>
</html>
```
预览：
![alt text](image-250.png)

**width和height属性**
![alt text](image-251.png)

![alt text](image-252.png)

**padding**
![alt text](image-253.png)
![alt text](image-254.png)
![alt text](image-255.png)
![alt text](image-256.png)
![alt text](image-257.png)
![alt text](image-258.png)
![alt text](image-259.png)
![alt text](image-260.png)
![alt text](image-261.png)

**margin**
![alt text](image-262.png)
![alt text](image-263.png)
![alt text](image-264.png)

![alt text](image-265.png)

![alt text](image-266.png)

**盒模型计算**

![alt text](image-267.png)
![alt text](image-268.png)
![alt text](image-269.png)


**box-sizing**

![alt text](image-270.png)
![alt text](image-271.png)

### 行内元素和块级元素

**display**
![alt text](image-272.png)
![alt text](image-273.png)

**行内元素和块级元素的相互转换**

![alt text](image-274.png)

**元素的隐藏**

![alt text](image-245.png)


