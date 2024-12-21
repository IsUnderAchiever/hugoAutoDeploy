---
title: 12.插槽slot
description: 12.插槽slot
date: 2024-06-01
slug: 12.插槽slot
image: 202412211946837.png
categories:
    - Vue
---

插槽就是子组件中的提供给父组件使用的一个`占位符`，用<slot></slot> 表示，父组件可以在这个占位符中填充任何模板代码，如 HTML、组件等，填充的内容会替换子组件的<slot></slot>标签。
## 匿名插槽
1.在子组件放置一个插槽
```vue
<template>
  <div>
    <slot></slot>
  </div>
</template>
<script setup lang="ts">
import {reactive, ref} from 'vue';
</script>
<style scoped>
</style>
```
2.父组件使用插槽
在父组件给这个插槽填充内容
```vue
<template>
  <Children>
    <template v-slot>
      <div>123</div>
    </template>
  </Children>
</template>
<script setup lang="ts">
import {markRaw, reactive, ref, shallowRef} from "vue";
import Children from "@/view/Children.vue";
</script>
<style scoped>
</style>
```
## 具名插槽
具名插槽其实就是给插槽取个名字。一个子组件可以放多个插槽，而且可以放在不同的地方，而父组件填充内容时，可以根据这个名字把内容填充到对应插槽中
```vue
<template>
  <div>
    <slot name="header"></slot>
    <slot></slot>
    <slot name="foot"></slot>
  </div>
</template>
```
```vue
<template>
  <Children>
    <template v-slot:header>
      <div>this is header</div>
    </template>
    <template v-slot>
      <div>this is slot</div>
    </template>
    <template v-slot:foot>
      <div>this is foot</div>
    </template>
  </Children>
</template>
<script setup lang="ts">
import {markRaw, reactive, ref, shallowRef} from "vue";
import Children from "@/view/Children.vue";
</script>
<style scoped>
</style>
```
> 插槽简写
```vue
<template>
  <Children>
    <template #header>
      <div>this is header</div>
    </template>
    <template #default>
      <div>this is slot</div>
    </template>
    <template #foot>
      <div>this is foot</div>
    </template>
  </Children>
</template>
<script setup lang="ts">
import {markRaw, reactive, ref, shallowRef} from "vue";
import Children from "@/view/Children.vue";
</script>
<style scoped>
</style>
```
## 作用域插槽
>在子组件动态绑定参数 派发给父组件的slot去使用
```VUE
<template>
  <div>
    <slot name="header"></slot>
    <div>
      <div v-for="item in 100">
        <slot :data="item"></slot>
      </div>
    </div>
    <slot name="footer"></slot>
  </div>
</template>
```
>通过结构方式取值
```VUE
<template>
  <Children>
    <template #header>
      <div>1</div>
    </template>
    <template #default="{ data }">
      <div>{{ data }}</div>
    </template>
    <template #footer>
      <div>3</div>
    </template>
  </Children>
</template>
<script setup lang="ts">
import {markRaw, reactive, ref, shallowRef} from "vue";
import Children from "@/view/Children.vue";
</script>
<style scoped>
</style>
```
## 动态插槽
插槽可以是一个变量名
```VUE
        <Dialog>
            <template #[name]>
                <div>
                    23
                </div>
            </template>
        </Dialog>
const name = ref('header')
```
