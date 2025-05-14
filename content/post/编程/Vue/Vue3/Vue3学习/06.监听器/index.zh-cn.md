---
title: 06.监听器
description: 06.监听器
date: 2023-07-13
slug: 06.监听器
image: 202412211946837.png
categories:
    - Vue
---

# 监听器
>watch第一个参数监听源
>
>watch第二个参数回调函数cb（newVal,oldVal）
>
>watch第三个参数一个options配置项是一个对象(`immediate`是否立即调用一次、deep:true 是否开启`深度监听`)
>
>[原博客](https://xiaoman.blog.csdn.net/article/details/122797990)
```vue
<template>
  <a-button danger @click="change">测试</a-button>
</template>
<script setup lang="ts">
import {reactive, ref, computed, watch} from 'vue';
import {message} from "ant-design-vue";
let keywords = ref<number>(0)
watch(keywords, (newVal, oldVal) => {
  message.success('新的值:' + newVal+'   旧的值:'+oldVal);
}, {
  deep: true
})
let change = () => {
  keywords.value += 1;
}
</script>
```
