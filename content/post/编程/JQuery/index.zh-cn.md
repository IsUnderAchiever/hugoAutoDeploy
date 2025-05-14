---
title: JQuery
description: JQuery
date: 2023-04-19
slug: JQuery
image: 202412212153256.png
categories:
    - JQuery
---

## JQuery
#### 初识语法
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>练习</title>
    <script src="/js/jquery-3.0.0.min.js"></script>
    <script src="/js/vue2.7.0.min.js"></script>
    <style>
        .one {
            width: 300px;
            height: 300px;
            background-color: red;
        }
    </style>
</head>
<body>
    <div id="app"></div>
    <script>
        $(document).ready(function () {
            console.log("aaa");
        });
        // 以上写法可简写成如下写法
        $(function () {
            console.log("bbb");
        });
        $(function () {
            // 支持链式编程，如下
            // $(".one").mouseover(function () {
            //     $(".one").css("backgroundColor", "green").next().css()
            // });
            $("#app").addClass("one");
            $(".one").mouseover(function () {
                $(".one").css("backgroundColor", "green")
            });
            $(".one").mouseout(function () {
                $(".one").css("backgroundColor", "red")
            });
        });
        // jquery对象转DOM对象
        // var $app = $("#app"); //jquery对象
        // var app = $app[0]; // DOM对象
        // var $app = $("#app"); //jquery对象
        // var app = $app.get(0); // DOM对象
    </script>
</body>
</html>
```
#### 类CSS选择器
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>练习</title>
    <script src="/js/jquery-3.0.0.min.js"></script>
    <script src="/js/vue2.7.0.min.js"></script>
</head>
<body>
    <h2>哈哈哈</h2>
    <h2 class="top" name="qaq">啊啊啊</h2>
    <span>呵呵</span>
    <div id="aaa">
        <p>哈哈</p>
        <div>呵呵</div>
        <span>aa</span>
        <p>bb</p>
        <p>哈哈</p>
        <div>cc</div>
        <p>哈哈</p>
    </div>
    <script>
        $(function () {
            // 并集选择器
            $("h2,span").css("color", "red");
            // 交集选择器
            // 选择class为top的h2标签
            $("h2.top").css("color", "green");
            // 全局选择器
            // $('*').css(...)
            // 后代选择器
            // $("#aaa p")
            // 子选择器
            // $("#aaa>p)
            // 相邻选择器
            // 这里选择的是id为aaa内的p标签，p标签后紧邻着的div
            $("#aaa>p+div").css('color', 'red');
            // 同辈选择器
            // 这里选择的是id为aaa内的div标签,div标签之后的所有的同辈的p标签
            // 请注意！！！
            // 选择的是div之后的p标签，所以第一个p标签并不会被选中
            // $("#aaa>div~p").css("color", "blue");
            // 属性选择器
            // 选择的是页面上具有class属性的标签
            $("[class]").css("color", "blue");
            // 选择的是页面上具有class属性的h2标签
            $("h2[class]").css("color", "blue");
            // 选择的是页面上class属性为top的h2标签
            $("h2[class='top']").css("color", "blue");
            // 选择的是页面上class属性以op结尾的h2标签
            $("h2[class$='op']").css("color", "blue");
            // 选择的是页面上class属性以t开头的h2标签
            $("h2[class^='t']").css("color", "blue");
            // 选择的是页面上class属性包含t的h2标签
            $("h2[class*='t']").css("color", "blue");
            // 多属性条件选择
            // 选择的是页面上包含class属性，且name属性以q开头 以q结尾的标签
            $("h2[class][name^='q'][name$='q']").css("color", "blue")
        })
    </script>
</body>
</html>
```
#### 过滤选择器
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>练习</title>
    <script src="/js/jquery-3.0.0.min.js"></script>
    <script src="/js/vue2.7.0.min.js"></script>
</head>
<body>
    <div>div1</div>
    <span>span</span>
    <div>div2</div>
    <div>div3</div>
    <p hidden>aaa</p>
    <script>
        $(function () {
            // 基本过滤选择器
            // 选择第一个div
            $("div:first").css("color", "red");
            // 选择最后一个div
            $("div:last").css("color", "blue");
            // 选择第偶数个div(从0开始)
            $("div:even").css("color", "red");
            // 选择第奇数个div(从0开始)
            $("div:odd").css("color", "red");
            //
            // 选择索引为1(从0开始)
            $("div:eq(1)").css("color", "red");
            // 选择索引大于1(从0开始)
            $("div:gt(1)").css("color", "red");
            // 选择索引小于1(从0开始)
            $("div:lt(1)").css("color", "blue");
            //  选择class不为one的li元素
            $("li:not(.one)")
            // 选择所有的标题元素
            $(":header")
            // 选择获取焦点的元素
            $(":focus")
            // 可见性过滤选择器
            // 选择隐藏的p标签，使其显示
            $("p:hidden").show();
            // 选择显示的p标签，使其隐藏
            $("p:visible").hide();
            // 表单对象过滤选择器
            // 内容过滤选择器、子元素过滤选择器
        })
    </script>
</body>
</html>
```
#### 事件
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        #app{
            width: 300px;
            height: 300px;
            background-color: red;
        }
    </style>
    <script src="../js/jquery/jquery-1.11.1.js"></script>
    <script>
        $(function () {
            // 鼠标事件
            $("div[id='app']").mouseover(function () {
                $("div[id='app']").css('backgroundColor',"blue");
            });
            $("div[id='app']").mouseout(function () {
                $("div[id='app']").css('backgroundColor',"red");
            })
        })
        $(document).keyup(function (event) {
//            console.log(event.keyCode);
            if(event.keyCode==13){
                console.log("提交成功!");
            }
        })
        
    </script>
</head>
<body>
<div id="app"></div>
</body>
</html>
```
#### JQuery动画
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        #app {
            width: 300px;
            height: 300px;
            background-color: red;
        }
    </style>
    <script src="../js/jquery/jquery-1.11.1.js"></script>
    <script>
        $(function () {
            $("div[id='app']").hover(
                    function () {
                        $("div[id='app']").css("backgroundColor", "green");
                    },
                    function () {
                        $("div[id='app']").css("backgroundColor", "red");
                    }
            );
        });
    </script>
</head>
<body>
<button>点击</button>
<br><br><br>
<div id="app"></div>
<script>
    var i=0;
    $("button:first").click(function () {
        i++;
        if(i%2==1){
//            $("div#app").hide("show");
            // 1000毫秒
//            $("div#app").hide(1000);
//            $("div#app").fadeOut("show");
            $("div#app").slideUp("show");
        }else{
//            $("div#app").show("show");
//            $("div#app").fadeIn("show");
            $("div#app").slideDown("show");
        }
        // 淡入、淡出效果
//        fadeIn、fadeOut
        // 改变元素的高度
//        slideUp、slideDown
    })
</script>
</body>
</html>
```
#### 样式及内容操作
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <script src="../js/jquery/jquery-1.11.1.js"></script>
</head>
<body>
<button id="cli">点击</button>
<input type="text" placeholder="随便输入">
<div id="app"></div>
<script>
$("button[id='cli']").click(function () {
//    $("div[id='app']").css("backgroundColor","red")
    // 同时设置多个样式
    $("div[id='app']").css({
        "width":"300px",
        "height":"300px",
        "backgroundColor":"red"
    })
    // 追加样式
//    addClass("classname1 classname2")
    // 移除样式
//    removeClass("style1 style2")
})
    $("input[type='text']").bind({
        "blur":function () {
            // 获取值
//            console.log("失去焦点了",$("input[type='text']").val());
            // 设置值
            // val()为空时，获取值；val("xxx")时，设置值
            $("input[type='text']").val("aaa")
        },
        "focus":function () {
            // 获取值
            console.log("获取焦点了",$("input[type='text']").val());
        }
    })
</script>
</body>
</html>
```
#### 节点操作
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <script src="../js/jquery/jquery-1.11.1.js"></script>
    <style>
        li{
            color: red;
        }
    </style>
</head>
<body>
<div id="app"></div>
<script>
    // 前者添加到后者
//    $("<li>啊哈哈哈</li>").appendTo("#app");
    // 向前者中添加后者
//    $("#app").append("<li>啊哈哈哈</li>");
    // 将后者添加到前者之前
//    $("#app").prepend("<li>啊哈哈哈</li>");
    // 将前者添加到后者之前
//    $("<li>啊哈哈哈</li>").prependTo("#app");
    // 前者后面添加后者，同辈节点
//    $("#app").after("<li>啊哈哈哈</li>");
    // 前者添加后者后面，同辈节点
//        $("<li>啊哈哈哈</li>").insertAfter("#app");
    // 前者前面添加后者，同辈节点
//        $("#app").before("<li>啊哈哈哈</li>");
    // 后者前面添加前者，同辈节点
//        $("<li>啊哈哈哈</li>").insertBefore("#app");
    // replace()和replaceAll()用于替换某个节点
    // 新节点替换旧节点
//    $("旧节点").replaceWith("新节点");
//    $("新节点").replaceAll("旧节点");
//    clone()用于复制某个节点
    // true 表示需要copy事件处理
//    $("span").clone(true).appendTo("div");
    // 删除节点
//    remove() 删除某个节点，自身删除自身
//    detach() 删除整个节点，保留元素事件、附加数据
//    empty() 清空节点内容，清空子节点
</script>
</body>
</html>
```
#### 属性操作&节点遍历
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <script src="../js/jquery/jquery-1.11.1.js"></script>
</head>
<body>
<div id="app" name="哈哈">
    <span id="test">
        <span>aaa</span>
    </span>
</div>
<input type="radio" name="gender">男
<script>
$(function () {
    // 获取name属性的值
    console.log($("#app").attr("name"));
    // 设置值
    $("#app").attr({style:"width: 300px;height:300px;background-color:red"})
    $("img").attr({width:'300px',height:"300px"})
    // prop 和 attr的区别
    // 第一个input标签
    // 显示 undefined
//    console.log($("input[type='radio']").attr("checked"));
    // 显示 false
//    console.log($("input[type='radio']").prop("checked"));
//    $("input[type='radio']").attr("checked","checked");
//    $("input[type='radio']").prop("checked","true");
    // 获取父级元素
//    console.log($("span[id='test']").parent());
    // 获取所有父级及祖级元素
//    console.log($("span[id='test']").parents());
    // 获取元素的所有子元素，但不包含孙子元素（子元素的子元素）
//    console.log($("#app").children());
    // 用于获取该标签之后的元素
//    next()
    // 用于获取该标签之前的元素
//    prev()
    // 用于获取该标签之前、与后的所有同辈元素元素
//    sublings()
//    css()
    // 设置或返回元素的宽度
//    weight
    // 设置或返回元素的高度
//    height
    $(function () {
//        $("div").each(function (i, domEle) {
//            // 索引
////            console.log(i);
//            console.log($(domEle).text());
//        })
    })
    $.each(("div"),function (i, domEle) {
        console.log($(domEle).text());
    })
})
</script>
</body>
</html>
```
