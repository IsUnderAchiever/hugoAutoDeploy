---
title: 05.计算属性
description: 05.计算属性
date: 2023-07-16
slug: 05.计算属性
image: 202412211946837.png
categories:
    - Vue
---

# 计算属性
> `computed`
>
> 购物车案例
```vue
<template>
  <a-input v-model:value="searchContent" type="text" style="width: 500px" placeholder="请输入关键词搜索"/>
  <table>
    <tr>
      <th v-for="title in dataList.title">{{ title }}</th>
    </tr>
    <tr v-for="(item,index) in searchDataList">
      <td align="center">{{ item.key }}</td>
      <td align="center">{{ item.name }}</td>
      <td align="center">{{ item.money }}</td>
      <td align="center">
        <a-button type="default" @click="item.num>1?item.num--:item.num">-</a-button>
        {{ item.num }}
        <a-button type="default" @click="item.num<99?item.num++:item.num">+</a-button>
      </td>
      <td align="center">{{ item.money * item.num }}￥</td>
      <td align="center">
        <a-button danger type="default" @click="deleteItem(index)">删除</a-button>
      </td>
    </tr>
    <tr>
      <td colspan="5" align="right">总价:{{ total }}</td>
    </tr>
  </table>
</template>
<script setup lang="ts">
import {reactive, ref, computed} from 'vue';
interface Data {
  key: string,
  name: string,
  money: number,
  num: number,
}
const searchContent = ref<string>('');
const dataList = reactive({
  title: [
    '编号',
    '商品',
    '单价',
    '数量',
    '金额',
    '操作'
  ],
  arr: [
    {
      key: '1',
      name: '昂贵的辣条',
      money: 100,
      num: 1,
    },
    {
      key: '2',
      name: '便宜的面包',
      money: 50,
      num: 1,
    },
    {
      key: '3',
      name: '稀有的牛奶',
      money: 200,
      num: 1,
    },
  ]
});
let data = dataList.arr;
let total = computed(() => {
  return data.reduce((pre: number, now: Data) => {
    return pre + now.num * now.money
  }, 0)
});
let deleteItem = (index: number) => {
  data.splice(index, 1)
}
const searchDataList = computed(() => {
  return data.filter((item: Data) => {
    return item.name.includes(searchContent.value);
  });
});
</script>
<style scoped>
th {
  width: 100px;
}
</style>
```
> `reduce`函数接受两个值，一个是之前的值，一个是改变后的值
>
> 第一次没有之前的值，所以赋值为0
![image-20230716213854442](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307162139704.png)
> 小满zs->购物车案例
```vue
<template>
    <div>
        <input placeholder="请输入名称" v-model="keyWord" type="text">
        <table style="margin-top:10px;" width="500" cellspacing="0" cellpadding="0" border>
            <thead>
                <tr>
                    <th>物品</th>
                    <th>单价</th>
                    <th>数量</th>
                    <th>总价</th>
                    <th>操作</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(item, index) in searchData">
                    <td align="center">{{ item.name }}</td>
                    <td align="center">{{ item.price }}</td>
                    <td align="center">
                        <button @click="item.num > 1 ? item.num-- : null">-</button>
                        <input v-model="item.num" type="number">
                        <button @click="item.num < 99 ? item.num++ : null">+</button>
                    </td>
                    <td align="center">{{ item.price * item.num }}</td>
                    <td align="center">
                        <button @click="del(index)">删除</button>
                    </td>
                </tr>
            </tbody>
            <tfoot>
                <tr>
                    <td colspan="5" align="right">
                        <span>总价：{{ total }}</span>
                    </td>
                </tr>
            </tfoot>
 
        </table>
    </div>
</template>
 
<script setup lang='ts'>
import { reactive, ref,computed } from 'vue'
let keyWord = ref<string>('')
interface Data {
    name: string,
    price: number,
    num: number
}
const data = reactive<Data[]>([
    {
        name: "小满的绿帽子",
        price: 100,
        num: 1,
    },
    {
        name: "小满的红衣服",
        price: 200,
        num: 1,
    },
    {
        name: "小满的黑袜子",
        price: 300,
        num: 1,
    }
])
 
let searchData = computed(()=>{
    return data.filter(item => item.name.includes(keyWord.value))
})
 
let total = computed(() => {
    return data.reduce((prev: number, next: Data) => {
        return prev + next.num * next.price
    }, 0)
})
 
const del = (index: number) => {
    data.splice(index, 1)
}
 
</script>
 
<style scoped lang='less'></style>
```
