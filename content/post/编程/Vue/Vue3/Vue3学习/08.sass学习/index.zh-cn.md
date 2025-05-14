---
title: 08.sass学习
description: 08.sass学习
date: 2023-07-17
slug: 08.sass学习
image: 202412211946837.png
categories:
    - Vue
---

# sass学习
> [sass官方文档](https://www.sass.hk/docs/)
>
> [less官方文档](https://less.bootcss.com/#%E6%A6%82%E8%A7%88)
>
> [原博客](https://xiaoman.blog.csdn.net/article/details/122832888)
bem架构
`BEM`实际上是`block`、`element`、`modifier`的缩写，分别为块层、元素层、修饰符层
BEM 命名约定的模式是：
```css
.block {}
 
.block__element {}
 
.block--modifier {}
```
### 嵌套规则
> Sass 允许将一套 CSS 样式嵌套进另一套样式中，内层的样式将它外层的选择器作为父选择器
```css
#main p {
  color: #00ff00;
  width: 97%;
  .redbox {
    background-color: #ff0000;
    color: #000000;
  }
}
```
### 父选择器 `&` 
```css
a {
  font-weight: bold;
  text-decoration: none;
  &:hover { text-decoration: underline; }
  body.firefox & { font-weight: normal; }
}
```
### 变量 `$`
```css
$width: 5em;
#main {
  width: $width;
}
```
### 插值语句 `#{}`
```css
$name: foo;
$attr: border;
p.#{$name} {
  #{$attr}-color: blue;
}
```
### @at-root
```css
@media print {
  .page {
    width: 8in;
    @at-root (without: media) {
      color: red;
    }
  }
}
```
> 编译后
```css
@media print {
  .page {
    width: 8in;
  }
}
.page {
  color: red;
}
```
### 定义混合指令 `@mixin`
>混合指令的用法是在 `@mixin` 后添加名称与样式，比如名为 `large-text` 的混合通过下面的代码定义：
```css
@mixin large-text {
  font: {
    family: Arial;
    size: 20px;
    weight: bold;
  }
  color: #ff0000;
}
```
### 引用混合样式 `@include`
```css
@mixin large-text {
  font: {
    family: Arial;
    size: 20px;
    weight: bold;
  }
  color: #ff0000;
}
.page-title {
  @include large-text;
  padding: 4px;
  margin-top: 10px;
}
```
> 编译后
```css
.page-title {
    font-family: Arial;
    font-size: 20px;
    font-weight: bold;
    color: #ff0000;
    padding: 4px;
    margin-top: 10px;
}
```
### 练习:实现layout布局
```css
$block-sel: "-" !default;
$element-sel: "__" !default;
$modifier-sel: "--" !default;
$namespace:'xm' !default;
@mixin bfc {
    height: 100%;
    overflow: hidden;
}
 
//混入
@mixin b($block) {
   $B: $namespace + $block-sel + $block; //变量
   .#{$B}{ //插值语法#{}
     @content; //内容替换
   }
}
 
@mixin flex {
    display: flex;
}
 
@mixin e($element) {
    $selector:&;
    @at-root {
        #{$selector + $element-sel + $element} {
            @content;
        }
    }
}
 
@mixin m($modifier) {
    $selector:&;
    @at-root {
        #{$selector + $modifier-sel + $modifier} {
            @content;
        }
    }
}
```
**全局扩充sass**
```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
 
// https://vitejs.dev/config/
export default defineConfig({
    plugins: [vue()],
    css: {
        preprocessorOptions: {
            scss: {
                additionalData: "@import './src/bem.scss';"
            }
        }
    }
})
```
**Vue 组件用法**
```vue
<template>
    <div class="xm-wraps">
         <div>
            <Menu></Menu>
         </div>
         <div class="xm-wraps__right">
            <Header></Header>
            <Content></Content>
         </div>
    </div>
</template>
 
<script lang="ts" setup>
import { ref, reactive } from "vue"
import Menu from './Menu/index.vue'
import Content from './Content/index.vue'
import Header from './Header/index.vue'
</script>
 
<style lang="scss" scoped>
@include b('wraps'){
    @include bfc;
    @include flex;
    @include e(right){
        flex:1;
        display: flex;
        flex-direction: column;
    }
}
</style>
```
