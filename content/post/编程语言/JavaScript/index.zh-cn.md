---
title: JavaScript
description: JavaScript
date: 2023-04-19
slug: JavaScript
image: 202412212151372.png
categories:
    - JavaScript
---

## JavaScript
#### 弹窗
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
    <button onclick="myAlert()" class="large-btn btn">click</button>
    <script>
        // 警告框
        function myAlert(){
            alert("警告");
        }
        // 确认框
        function myConfirm(){
            var flag=confirm("是否确认？");
            console.log("您选择是:",flag===true?'确认':'取消');
        }
        // 提示框
        function myPrompt(){
            prompt("标题","默认的内容是XXX(不写则为空)");
        }
    </script>
</body>
</html>
```
#### 事件
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>练习</title>
    <script src="/js/jquery-3.0.0.min.js"></script>
    <script src="/js/vue2.7.0.min.js"></script>
    <style>
        .my-div{
            width:100px;
            height: 100px;
            background-color: red;
            border: 1px solid black;
        }
    </style>
</head>
<body>
    <div class="my-div" onclick="myClick()"></div>
    <input type="text" onfocus="myFocus()" placeholder="获取焦点"><br>
    <input type="text" onblur="myBlur()" placeholder="失去焦点"><br>
    <div class="my-div" onmouseover="myMouseover()"></div>
    <input type="text" onkeyup="myKeyup()" placeholder="键盘弹起"><br>
    <script>
        // 鼠标点击
        function myClick(){
            console.log("鼠标点击");
        }
        // 获取焦点
        function myFocus(){
            console.log("获取焦点");
        }
        // 失去焦点
        function myBlur(){
            console.log("失去焦点");
        }
        // 鼠标移入
        function myMouseover(){
            console.log("鼠标移入");
        }
        // 键盘弹起
        function myKeyup(){
            console.log("键盘弹起");
        }
    </script>
</body>
</html>
```
#### 时钟
> `setInterval` & `setTimeout`
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
    <div id="app"></div>
    
    <script>
        //        var 日期对象=new Date(参数)
        //        参数格式 MM DD,YYYY,hh:mm:ss
        window.onload = function () {
            this.showTime();
        };
        function showTime() {
            // 每隔1秒执行一次
            setInterval(function () {
                time = this.getTime();
                document.getElementById('app').innerText = time
            }, 1000);
            // 定时1秒后执行
            // setTimeout(function () {
            //     console.log('aaa');
            // })
        }
        function getTime() {
            var today = new Date();
            console.log(today.getDay());
            console.log(today.getMonth() + 1);
            console.log(today.getFullYear() + '年:' + (today.getMonth() + 1) + '月' + today.getDate() + '日' + today.getHours() + ':' + today.getMinutes() + ':' + today.getSeconds());
            return today.getFullYear() + '年:' + (today.getMonth() + 1) + '月' + today.getDate() + '日' + today.getHours() + ':' + today.getMinutes() + ':' + today.getSeconds();
        }
    </script>
</body>
</html>
```
#### 显示与隐藏
> `display 不保留隐藏的属性`
>
> `visibility 只是隐藏，原本的标签还在`
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>练习</title>
    <script src="/js/jquery-3.0.0.min.js"></script>
    <script src="/js/vue2.7.0.min.js"></script>
    <style>
        div {
            display: none;
        }
    </style>
</head>
<body>
    <a href="#" onclick="dpcq()">斗破苍穹</a>&nbsp;<a href="#" onclick="wdqk()">武动乾坤</a><br>
    <button>点击1</button>
    <br>
    <button onclick="showButton1()">点击2</button>
    <script>
        //        document.URL 返回当前文档URL
        //        document.referrer 载入本页面文档的地址
        // 由于代码写在body上方，此时还未加载a标签，所以应该将script写在body下方
        var num = document.getElementsByTagName('a');
        for (var i = 0; i < num.length; i++) {
            console.log(num[i]);
        }
        console.log('aaa');
        // 显示斗破苍穹
        function dpcq() {
            document.getElementsByTagName('div')[0].style.display = 'block';
            document.getElementsByTagName('div')[1].style.display = 'none';
        }
        // 显示武动乾坤
        function wdqk() {
            document.getElementsByTagName('div')[1].style.display = 'block';
            document.getElementsByTagName('div')[0].style.display = 'none';
        }
        function showButton1() {
            var button1 = document.getElementsByTagName('button')[0];
            button1.style.visibility = 'hidden'
            button1.style.display = 'none'
            // display 和 visibility 的区别
            // display 不保留隐藏的属性
            // visibility 只是隐藏，原本的标签还在
        }
    </script>
</body>
</html>
```
#### 全选
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script>
        // document获取对象的其他方法
        //        querySelector() 返回页面中第一个匹配规则的元素
        //        querySelectorAll() 返回页面中匹配规则的全部元素
        window.onload = function () {
            var list = document.querySelectorAll(".one")
            for (let i = 0; i < list.length; i++) {
                list[i].onclick = function () {
                    var flag = true;
                    for (let j = 0; j < list.length; j++) {
                        // 等同于
                        // if(lest[j].checked==false)
                        if (!list[j].checked) {
                            flag = false;
                        }
                        document.querySelector("input").checked = flag;
                    }
                }
            }
        }
        function chooseAll() {
            // 打印全选的选中状态
            // console.log(document.querySelector("input").checked);
            var checkList = document.querySelectorAll(".one")
            for (let i = 0; i < checkList.length; i++) {
                checkList[i].checked = document.querySelector("input").checked;
            }
        }
    </script>
</head>
<body>
    <input type="checkbox" value="all" onclick="chooseAll()">全选<br>
    <input type="checkbox" class="one" value="eat">吃饭<br>
    <input type="checkbox" class="one" value="game">游戏<br>
    <input type="checkbox" class="one" value="learn">学习<br>
</body>
</html>
```
#### CSS交互
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        #app {
            width: 50px;
            height: 50px;
            background-color: red;
        }
        .one {
            width: 50px;
            height: 50px;
            background-color: red;
        }
        .two {
            width: 50px;
            height: 50px;
            background-color: green;
        }
    </style>
    <script>
        window.onload = function () {
            // 方法一：
            // var allDiv = document.querySelector("#app");
            // allDiv.onmouseover = function () {
            //     allDiv.style.backgroundColor = 'green';
            // };
            // allDiv.onmouseout = function () {
            //     allDiv.style.backgroundColor = 'red';
            // };
            // 方法二：
            // var allDiv = document.querySelector("#app");
            // allDiv.onmouseover = function () {
            //     allDiv.id = '';
            //     // 去掉div id则可以显示 two的样式
            //     // 因为id的优先级大于class
            //     allDiv.className = 'two';
            // };
            // allDiv.onmouseout = function () {
            //     allDiv.className = 'one';
            // };
        };
        function test() {
            // background-color 改为 backgroundColor，驼峰命名
            document.getElementById("app").style.backgroundColor = 'grey';
        }
    </script>
</head>
<body>
    <button onclick="test()">点击</button>
    <br><br>
    <div id="app" class="two"></div>
</body>
</html>
```
#### 获取文本框的值
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
    <input type="text">
    <button onclick="getInputData()">获取</button>
    <script>
            function getInputData(){
                console.log("值:",document.getElementsByTagName("input")[0].value);
            }
    </script>
</body>
</html>
```
#### 省市级联
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script>
        window.onload=function () {
            var pro=document.getElementById("province");
            var city=document.getElementById("city");
            pro.onchange=function () {
                city.innerHTML="";
                var arr=new Array(2);
                arr[0]=['武汉','荆州',"aaa"];
                arr[1]=['长沙','啊啊'];
                var index=pro.selectedIndex-1;
                for(let i=0;i<arr.length;i++){
                    city.add(new Option(arr[index][i],arr[index][i]),null)
                }
            }
        }
    </script>
</head>
<body>
<select id="province">
    <option value="0" hidden>--请选择--</option>
    <option value="1">湖北</option>
    <option value="2">湖南</option>
</select>
<select id="city"></select>
</body>
</html>
```
#### 正则表达式校验
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
    <input type="text" id="username">
    <span id="tip" style="color: red"></span>
    <script>
        // 正则表达式语法
        // 普通方式
        // var reg=/表达式/附加参数
        // 如：var reg=/white/g;
        // 构造函数
        // var reg=new RegExp("表达式","附加参数");
        // 简单模式
        // var reg=/china8/;
        // 复合模式
        // var reg=/^\w+$/;
        // RegExp 对象的方法
        // exec 检索字符中是正则表达式的匹配，返回找到的值
        // test 检索字符中指定的值，返回true or false
        // RegExp 对象的属性
        // global RegExp对象是否具有标志g，全文查找
        // ignoreCase RegExp对象是否具有标志i，忽略大小写
        // multiline RegExp对象是否具有标志m，多行查找
        window.onload = function () {
            var user = document.getElementById("username");
            user.onblur = function () {
                var reg = /^\d{3}\S{1}@\S{2}.com$/
                if (!reg.test(user.value)) {
                    document.getElementById("tip").innerText = '格式输入错误'
                } else {
                    document.getElementById("tip").innerText = ''
                }
            }
        }
    </script>
```
