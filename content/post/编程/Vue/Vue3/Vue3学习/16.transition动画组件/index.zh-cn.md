---
title: 16.transition动画组件
description: 16.transition动画组件
date: 2024-06-01
slug: 16.transition动画组件
image: 202412211946837.png
categories:
    - Vue
---

# transition动画组件
### 自定义过渡 class 类名
trasnsition props
- `enter-from-class`
- `enter-active-class`
- `enter-to-class`
- `leave-from-class`
- `leave-active-class`
- `leave-to-class`
自定义过度时间 单位毫秒
你也可以分别指定进入和离开的持续时间：
```vue
<transition :duration="1000">...</transition>
<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```
```vue
<template>
  <button @click="flag=!flag">点击{{ flag ? '隐藏' : '显示' }}</button>
  <transition leave-from-class="leaveFrom" leave-active-class="leaveActive" leave-to-class="leaveTo"
              enter-from-class="enterFrom" enter-active-class="enterActive" enter-to-class="enterTo">
    <div id="att" v-if="flag"></div>
  </transition>
</template>
<script setup lang="ts">
import {defineAsyncComponent, reactive, ref} from "vue";
import A from './A.vue'
const flag = ref<boolean>(true)
</script>
<style scoped lang="scss">
#att {
  margin: 10px 10px;
  width: 300px;
  height: 300px;
  background: red;
}
.leaveFrom{
  margin: 10px 10px;
  width: 300px;
  height: 300px;
  background: red;
}
.leaveActive{
  transition: all 1.5s ease;
}
.leaveTo{
  margin: 10px 10px;
  width: 0;
  height: 0;
  background: red;
}
.enterTo{
  margin: 10px 10px;
  width: 300px;
  height: 300px;
  background: red;
}
.enterActive{
  transition: all 1s linear;
}
.enterFrom{
  margin: 10px 10px;
  width: 0;
  height: 0;
  background: red;
}
</style>
```
通过自定义class 结合css动画库animate css
```bash
npm install animate.css
```
[官网](https://animate.style/)
**引入** 
```javascript
import 'animate.css'
```
使用方法
官方文档 [Animate.css | A cross-browser library of CSS animations.](https://animate.style/)
```vue
        <transition
            leave-active-class="animate__animated animate__bounceInLeft"
            enter-active-class="animate__animated animate__bounceInRight"
        >
            <div v-if="flag" class="box"></div>
        </transition>
```
```vue
<template>
  <button @click="flag=!flag">点击{{ flag ? '隐藏' : '显示' }}</button>
  <transition leave-active-class="animate__animated animate__bounceInLeft" enter-active-class="animate__animated animate__bounceInRight">
    <div id="att" v-if="flag"></div>
  </transition>
</template>
<script setup lang="ts">
import {defineAsyncComponent, reactive, ref} from "vue";
import A from './A.vue'
import 'animate.css'
const flag = ref<boolean>(true)
</script>
<style scoped lang="scss">
#att {
  margin: 10px 10px;
  width: 300px;
  height: 300px;
  background: red;
}
.leaveFrom{
  margin: 10px 10px;
  width: 300px;
  height: 300px;
  background: red;
}
.leaveActive{
  transition: all 1.5s ease;
}
.leaveTo{
  margin: 10px 10px;
  width: 0;
  height: 0;
  background: red;
}
.enterTo{
  margin: 10px 10px;
  width: 300px;
  height: 300px;
  background: red;
}
.enterActive{
  transition: all 1s linear;
}
.enterFrom{
  margin: 10px 10px;
  width: 0;
  height: 0;
  background: red;
}
</style>
```
# 3.transition 生命周期8个
```vue
  @before-enter="beforeEnter" //对应enter-from
  @enter="enter"//对应enter-active
  @after-enter="afterEnter"//对应enter-to
  @enter-cancelled="enterCancelled"//显示过度打断
  @before-leave="beforeLeave"//对应leave-from
  @leave="leave"//对应enter-active
  @after-leave="afterLeave"//对应leave-to
  @leave-cancelled="leaveCancelled"//离开过度打断
```
当只用 JavaScript 过渡的时候，在 **`enter` 和 `leave` 钩子中必须使用 `done` 进行回调**
结合gsap 动画库使用 [GreenSock](https://greensock.com/)
```bash
npm install gsap -S
```
**引入**
```javascript
import gsap from 'gsap'
```
```javascript
const beforeEnter = (el: Element) => {
    console.log('进入之前from', el);
}
const Enter = (el: Element,done:Function) => {
    console.log('过度曲线');
    setTimeout(()=>{
       done()
    },3000)
}
const AfterEnter = (el: Element) => {
    console.log('to');
}
```
```vue
<template>
  <button @click="flag=!flag">点击{{ flag ? '隐藏' : '显示' }}</button>
  <transition @beforeEnter="enterFrom" @enter="enterActive">
    <div id="att" v-if="flag"></div>
  </transition>
</template>
<script setup lang="ts">
import {defineAsyncComponent, reactive, ref} from "vue";
import A from './A.vue'
import 'animate.css'
import gsap from 'gsap'
const flag = ref<boolean>(true)
let enterFrom=(el:Element)=>{
  gsap.set(el,{
    width:0,
    height:0
  });
}
let enterActive=(el:Element,done:gsap.Callback)=>{
  gsap.to(el,{
    width:300,
    height:300
  });
}
</script>
<style scoped lang="scss">
#att {
  margin: 10px 10px;
  width: 300px;
  height: 300px;
  background: red;
}
</style>
```
# 4.appear
通过这个属性可以设置初始节点过度 就是页面加载完成就开始动画 对应三个状态
```vue
appear-active-class=""
appear-from-class=""
appear-to-class=""
appear
```
