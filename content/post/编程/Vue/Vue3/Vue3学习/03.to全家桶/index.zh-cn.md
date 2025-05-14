---
title: 03.to全家桶
description: 03.to全家桶
date: 2023-07-13
slug: 03.to全家桶
image: 202412211946837.png
categories:
    - Vue
---

# 03.to全家桶
## toRef
> 修改响应式对象的值
>
> 对于非响应式数据，只会改变数据，视图依然不会发生变化
>
> toRef(A,B)
>
> A=>对象
>
> B=>key值
```vue
<template>
  <a-button danger @click="test1">test</a-button>
</template>
<script setup lang="ts">
import {reactive,toRef, toRefs, toRaw} from 'vue';
let user=reactive({
  name:'马小跳',
  gender:'男',
  like:'篮球'
})
let refName=toRef(user,'name');
let test1=()=>{
  console.log(refName);
  console.log(refName.value);
  // 对值进行修改
  refName.value='洛洛';
}
</script>	
```
![image-20230713225812269](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132258373.png)
**应用场景**
有某函数需要接受指定参数，可使用toRef解构出对象的属性给指定函数
```vue
<template>
  {{user.name}}
  <a-button danger @click="test1">test</a-button>
</template>
<script setup lang="ts">
import {reactive,toRef, toRefs, toRaw} from 'vue';
import {message} from "ant-design-vue";
let user=reactive({
  name:'马小跳',
  gender:'男',
  like:'篮球'
})
let test1=()=>{
  let refName=toRef(user,'name');
  sendMessage(refName.value);
}
let sendMessage=(name:string)=>{
  message.success('你好,'+name);
}
</script>
```
## toRefs
```vue
<template>
  {{user.name}}
  <a-button danger @click="test1">test</a-button>
</template>
<script setup lang="ts">
import {reactive,toRef, toRaw} from 'vue';
import {message} from "ant-design-vue";
let user=reactive({
  name:'马小跳',
  gender:'男',
  like:'篮球'
})
// 类似实现原理
const toRefs=<T extends object>(object:T)=>{
  const map:any={};
  for (let key in object) {
    map[key]=toRef(object,key)
  }
  return map;
}
let test1=()=>{
  let {name,gender}=toRefs(user);
  console.log(name.value);
  console.log(gender.value);
}
</script>
```
> 否则失去响应式
```vue
<template>
  {{ user.name }}
  <a-button danger @click="test1">test</a-button>
</template>
<script setup lang="ts">
import {reactive, toRef, toRefs, toRaw} from 'vue';
let user = reactive({
  name: '马小跳',
  gender: '男',
  like: '篮球'
})
let test1 = () => {
  // let {name,gender}=toRefs(user);
  // 会失去响应式
  let {name, gender} = user;
  console.log(name);
  console.log(gender);
}
</script>
```
## toRaw
> 获取代理前的对象
```vue
<template>
  {{ user.name }}
  <a-button danger @click="test1">test</a-button>
</template>
<script setup lang="ts">
import {reactive, toRef, toRefs, toRaw} from 'vue';
let user = reactive({
  name: '马小跳',
  gender: '男',
  like: '篮球'
})
let test1 = () => {
  console.log(user);
  console.log(toRaw(user));
}
</script>
```
![image-20230713231621346](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307132316449.png)
