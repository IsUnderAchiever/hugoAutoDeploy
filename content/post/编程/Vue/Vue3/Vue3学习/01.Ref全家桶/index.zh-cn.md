---
title: 01.Ref全家桶
description: 01.Ref全家桶
date: 2023-07-13
slug: 01.Ref全家桶
image: 202412211946837.png
categories:
    - Vue
---

# 01.Ref全家桶
## Ref
```vue
<template>
  <p>message1:{{ message1 }}</p>
  <a-button @click="refTest">Ref测试</a-button>
</template>
<script lang="ts">
import {customRef, defineComponent, isRef, reactive, ref, shallowRef, triggerRef} from 'vue';
import {notification} from "ant-design-vue";
export default defineComponent({
  name: '',
  setup() {
    let message1 = ref<string>("信息")
    /**
     * 测试ref
     * 接受一个内部值并返回一个响应式且可变的 ref 对象
     */
    let refTest = () => {
      message1.value += "="
      notification['success']({
        message: '测试ref',
        description:
            '测试成功',
      });
    }
    return {
      message1,
      refTest,
    }
  }
})
</script>
```
## isRef
```vue
<template>
  <p>message1:{{ message1 }}</p>
  <p>message2:{{ message2 }}</p>
  <a-button @click="isRefTest">isRef测试</a-button>
</template>
<script lang="ts">
import {customRef, defineComponent, isRef, onMounted, reactive, ref, shallowRef, triggerRef} from 'vue';
import {notification} from "ant-design-vue";
export default defineComponent({
  name: '',
  setup() {
    let message1 = ref<string>("信息")
    let message2 = "信息"
    /**
     * 测试isRef
     * 判断是不是一个ref对象
     */
    let isRefTest = () => {
      notification['success']({
        message: '测试isRef',
        description:
            'message1' + (isRef(message1) ? '是' : '不是') + 'ref//' + 'message2' + (isRef(message2) ? '是' : '不是') + 'ref',
      });
    }
    
    return {
      message1,
      message2,
      isRefTest
    }
  }
})
</script>
<style scoped>
</style>
```
## shallowRef
```vue
<template>
  <p>message3:{{ message3 }}</p>
  <a-button @click="shallowRefTest(true)">shallowRef测试</a-button>
</template>
<script lang="ts">
import {customRef, defineComponent, isRef, onMounted, reactive, ref, shallowRef, triggerRef} from 'vue';
import {notification} from "ant-design-vue";
export default defineComponent({
  name: '',
  setup() {
    let message3 = shallowRef({
      name: '马小跳'
    })
    /**
     * 测试shallowRef
     */
    let shallowRefTest = (flag: boolean) => {
      // 有以下两种方法
      // 只有直接直接针对value来进行修改才会触发响应式
      if (!flag) {
        // 不会变成响应式的情况
        message3.value.name = 'aaa';
        notification['error']({
          message: '测试shallowRef',
          description: '没有响应式变化'
        });
      } else {
        // 会变成响应式的情况
        message3.value = {name: 'aaa'};
        notification['success']({
          message: '测试shallowRef',
          description: '发生响应式变化'
        });
      }
    }
    return {
      message3,
      shallowRefTest
    }
  }
})
</script>
<style scoped>
</style>
```
## triggerRef
```vue
<template>
  <p>message3:{{ message3 }}</p>
  <a-button @click="triggerRefTest">triggerRef测试</a-button>
</template>
<script lang="ts">
import {customRef, defineComponent, isRef, onMounted, reactive, ref, shallowRef, triggerRef} from 'vue';
import {notification} from "ant-design-vue";
export default defineComponent({
  name: '',
  setup() {
    let message3 = shallowRef({
      name: '马小跳'
    })
    /**
     * 强制更新页面DOM
     */
    let triggerRefTest = () => {
      message3.value.name = 'bbb';
      triggerRef(message3);
      notification['success']({
        message: '测试triggerRef',
        description: '已经强制更新页面DOM'
      });
    }
  
    return {
      message3,
      triggerRefTest
    }
  }
})
</script>
<style scoped>
</style>
```
## customRef
```vue
<template>
  <p>message4:{{ message4 }}</p>
  <a-button @click="customRefTest">customRef测试</a-button>
</template>
<script lang="ts">
import {customRef, defineComponent, isRef, onMounted, reactive, ref, shallowRef, triggerRef} from 'vue';
import {notification} from "ant-design-vue";
export default defineComponent({
  name: '',
  setup() {
    let message4=myCustomRef('哈哈哈');
    /**
     * 自定义ref
     * customRef 是个工厂函数要求我们返回一个对象 并且实现 get 和 set
     * 适合去做防抖之类的
     */
    function myCustomRef(value) {
      let timer: any;
      return customRef((track, trigger) => {
        return {
          get() {
            track();
            return value;
          },
          set(newVal) {
            // 制作防抖
            clearTimeout(timer)
            timer = setTimeout(() => {
              console.log('触发了set')
              value = newVal
              timer = null;
              notification['success']({
                message: '测试customRef',
                description: '已经调用customRefTest方法'
              });
              trigger();
            }, 500)
          }
        }
      })
    }
    let customRefTest=()=>{
      message4.value+='=='
    }
    return {
      message4,
      customRefTest
    }
  }
})
</script>
<style scoped>
</style>
```
## 标签中的ref属性
```vue
<template>
  <div ref="dom">aaa</div>
  <a-button @click="buttonTest">ref属性测试</a-button>
</template>
<script lang="ts">
import {customRef, defineComponent, isRef, onMounted, reactive, ref, shallowRef, triggerRef} from 'vue';
import {notification} from "ant-design-vue";
export default defineComponent({
  name: '',
  setup() {
    const dom=ref<HTMLDivElement>()
    /**
     * 获取到元素节点
     * 要求:ref的值相同
     * 如:ref="dom"、const dom=ref<>();
     */
    let buttonTest=()=>{
      console.log("dom:",dom.value);
    }
    return {
      // 需要return出来才能获取到节点
      dom,
      buttonTest
    }
  }
})
</script>
<style scoped>
</style>
```
