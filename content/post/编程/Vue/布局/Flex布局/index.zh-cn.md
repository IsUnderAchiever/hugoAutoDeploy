---
title: Flex布局
description: Flex布局
date: 2023-04-16
slug: Flex布局
image: 202412212158979.png
categories:
    - CSS
---

> 推介[查看](https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title></title>
    <script src="https://cdn.staticfile.org/jquery/3.0.0/jquery.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        body {
            width: 100%;
            height: 100vh;
            /* flex布局 */
            display: flex;
            /* justify-content & align-items */
            /* 延横轴方向居中对齐 */
            /*justify-content: center;*/
            /* 靠右对齐 */
            /*justify-content: flex-end;*/
            /* 左右两端对齐，相等间距 */
            /*justify-content: space-between;*/
            /* 项目中间的间距为左右两端的2倍 */
            /*justify-content: space-around;*/
            /* 项目之间间距与项目容器之间间距相等 */
            /*justify-content: space-evenly;*/
            /* 默认，沿着交叉轴布局方向分布 */
            /*justify-content: flex-start;*/
            /* 交叉轴居中排列,此处需要将父元素设置一定高度才起作用 */
            /*align-items: center;*/
            /* 交叉轴底部对齐 */
            /*align-items: flex-end;*/
            /* 项目水平垂直居中 */
            /*justify-content: center;*/
            /*align-items: center;*/
            /* flex-direction & flex-wrap*/
            /* 默认为row */
            /*flex-direction: row-reverse;*/
            /* 按列分布 */
            /*flex-direction: column;*/
            /*flex-direction: column-reverse;*/
            /* flex-wrap默认为nowrap，项目会强行等分容器，且不换行 */
            /* 根据自身宽度进行排列 */
            flex-wrap: wrap;
        }
        /* 项目属性: order & flex & align-self */
        /*order 排列顺序*/
        /*flex-grow 项目在有剩余空间时，是否放大*/
        /*取值:默认0 1 auto, flex属性是flex- grow, flex-shrink与flex- basis三个属性的简写，*/
        /*用于定义项目放大，缩小与宽度。该属性有两个快捷键值，分别是auto(1 1 auto)等分放大缩小，与none(0 0 auto)不放大，但等分缩小，*/
        .test-div {
            width: 200px;
            height: 200px;
            background-color: #00FFFF;
            border: 1px solid black;
			text-align: center;
			line-height: 200px;
        }
        .test-div:first-child {
            background-color: red;
            /*order: 1;*/
            /*align-self: center;*/
            /*align-self: flex-end;*/
        }
        .test-div:nth-child(2) {
            background-color: gray;
            /*order: 0;*/
        }
        .test-div:nth-child(3) {
            background-color: greenyellow;
            /*order: 2;*/
        }
    </style>
</head>
<body>
<div class="test-div">1</div>
<div class="test-div">2</div>
<div class="test-div">3</div>
</body>
</html>
```
