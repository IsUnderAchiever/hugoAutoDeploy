---
title: 09.父子组件传参
description: 09.父子组件传参
date: 2023-07-17
slug: 09.父子组件传参
image: 202412211946837.png
categories:
    - Vue
---

# 父子组件传参
## 父传子
> 父组件通过`v-bind`绑定一个数据，然后子组件通过defineProps接受传过来的值
**父组件**
```vue
<template>
  <children title="标题1"/>
</template>
<script setup lang="ts">
import Children from '@/view/Children.vue'
</script>
```
**子组件**
```vue
<template>
这里是children，父组件传递的title为<span>{{title}}</span>
</template>
<script setup lang="ts">
import {reactive, ref} from 'vue';
defineProps({
  title:{
    type:String,
    default:'默认标题'
  }
})
</script>
```
> 如果使用`typescript`，则可以采用如下写法
```vue
<template>
这里是children，父组件传递的title为<span>{{title}}</span>
</template>
<script setup lang="ts">
import {reactive, ref} from 'vue';
const props=defineProps<{
  title:string
}>()
console.log('父组件传递的title为:',props.title);
</script>
```
> 若需要设置默认值
```vue
<template>
这里是children，父组件传递的title为<span>{{title}}</span>
</template>
<script setup lang="ts">
import {reactive, ref} from 'vue';
type Props={
  title:string
  arr:number[]
}
withDefaults(defineProps<Props>(),{
  title:'默认标题',
  arr:()=>[1,2,3,4,5,6]
})
</script>
```
## 子传父
**子组件**
```vue
<template>
  <button @click="send">sendToFather</button>
</template>
<script setup lang="ts">
import {reactive, ref} from 'vue';
const emit=defineEmits(['on-click'])
const send=()=>{
  emit('on-click','a','b','c')
}
</script>
```
**父组件**
```vue
<template>
  <children @on-click="getProps"/>
</template>
<script setup lang="ts">
import Children from '@/view/Children.vue'
let getProps=(...name:string[])=>{
  console.log(name);
}
</script>
```
> ts写法
```vue
<template>
  <button @click="send">sendToFather</button>
</template>
<script setup lang="ts">
import {reactive, ref} from 'vue';
interface Props {
  (e: 'on-click', ...name: string[]): void
}
const emit = defineEmits<Props>()
const send = () => {
  emit('on-click', 'a', 'b', 'c')
}
</script>
```
**给父组件传递方法**
**子组件**
```vue
<template>
  这是子组件
</template>
<script setup lang="ts">
import {reactive, ref} from 'vue';
let time=ref<string>('2023/7/17 22:00')
const open=()=>{
  console.log('这是open方法,当前时间为:',time.value)
}
defineExpose({
  name:'马小跳',
  open
})
</script>
```
**父组件**
```vue
<template>
  <children ref="childProps"/>
  <button @click="test">测试</button>
</template>
<script setup lang="ts">
import Children from '@/view/Children.vue'
import {ref} from "vue";
const childProps=ref(null);
let test=()=>{
  console.log('name:',childProps.value.name);
  childProps.value.open()
}
</script>
```
