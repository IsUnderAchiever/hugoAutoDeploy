---
title: 18.依赖注入Provide、Inject
description: 18.依赖注入Provide、Inject
date: 2023-07-13
slug: 18.依赖注入Provide、Inject
image: 202412211946837.png
categories:
    - Vue
---

# 依赖注入Provide、Inject
> Home.vue
```vue
<template>
  <div v-for="item in colorList">
    <input type="radio" name="color" v-model="colorVal" :value="item.value">{{item.showName}}
  </div>
  <A/>
  <B/>
</template>
<script setup lang='ts'>
import {provide, reactive, ref} from 'vue'
import A from './A.vue'
import B from './B.vue'
type Color={
  value:string,
  showName:string
}
let colorList=reactive<Array<Color>>([{
  value:'yellow',
  showName:'黄色'
},{
  value:'red',
  showName:'红色'
},{
  value:'purple',
  showName:'紫色'
}])
let colorVal=ref<string>('')
provide('color',colorVal);
</script>
```
> A.vue
>
> `在css中也可以使用v-bind来绑定参数`
```vue
<template>
  <div>这是A组件</div>
  <div id="attA"></div>
</template>
<script setup lang="ts">
import {inject, reactive, ref} from 'vue';
import type {Ref} from "vue";
let color=inject<Ref<string>>('color');
</script>
<style scoped>
#attA{
  width: 30px;
  height: 30px;
  background: v-bind(color);
}
</style>
```
> B.vue
```vue
<template>
  <div>这是B组件</div>
  <div id="attB"></div>
</template>
<script setup lang="ts">
import {inject, reactive, ref} from 'vue';
import type {Ref} from "vue";
let color=inject<Ref<string>>('color');
</script>
<style scoped>
#attB{
  width: 30px;
  height: 30px;
  background: v-bind(color);
}
</style>
```
> 注意，此时的参数可以在子组件中修改
>
> 此时会导致父组件以及其他子组件的该参数都会发生改变
```vue
<template>
  <div>这是A组件</div>
  <button @click="changeColor">修改颜色</button>
  <div id="attA"></div>
</template>
<script setup lang="ts">
import {inject, reactive, ref} from 'vue';
import type {Ref} from "vue";
let color=inject<Ref<string>>('color');
let changeColor=()=>{
  color.value='black'
}
</script>
<style scoped>
#attA{
  width: 30px;
  height: 30px;
  background: v-bind(color);
}
</style>
```
> 若不希望子组件进行修改，可设置为`readonly`
```javascript
let colorVal=ref<string>('')
provide('color',readonly(colorVal));
```
> 子组件无法使用xxx.?value来进行修改，可采用以下两种方法
**1.默认值**
```vue
let color=inject<Ref<string>>('color',ref('red'));
```
**2.非空断言**
```
color!.value='xxx'
```
