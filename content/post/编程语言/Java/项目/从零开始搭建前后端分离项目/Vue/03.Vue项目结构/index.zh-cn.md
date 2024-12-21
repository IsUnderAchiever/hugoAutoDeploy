---
title: 从零开始搭建前后端分离项目03
description: 从零开始搭建前后端分离项目03
date: 2022-12-15
slug: 从零开始搭建前后端分离项目03
image: 202412211946837.png
categories:
    - Vue
---

## vue项目布局
[Element-plus](https://element-plus.gitee.io/zh-CN/)
HelloWorld组件删掉，再把HomeView内相关的内容删一删，App.vue内样式全部删掉
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946517.png)
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946393.png)
如果报错，可能是由于权限不够，复制报错信息上网查找解决方案即可
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946292.png)
全局引入element-plus
再次运行项目
![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946008.png)
选择这里的代码，复制到App.vue
![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946935.png)
```vue
<template>
  <div class="common-layout">
    <el-container>
      <el-header>
        Header
      </el-header>
      <el-container>
        <el-aside width="200px">Aside</el-aside>
        <el-main>
          <router-view/>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>
<script lang="ts">
import {defineComponent} from 'vue';
export default defineComponent({
  components: {
  },
});
</script>
<style>
</style>
```
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946870.png)
现在就是页面暂时的结构
由于项目会涉及到图标，继续下载
```bash
npm install @element-plus/icons-vue
```
```js
// main.ts
// 如果您正在使用CDN引入，请删除下面一行。
import * as ElementPlusIconsVue from '@element-plus/icons-vue'
const app = createApp(App)
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component)
}
```
![7](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946200.png)
测试一下图标是否安装好了
```html
<!-- Home -->
<el-row>
      <el-button icon="Search" circle/>
      <el-button type="primary" icon="Edit" circle/>
      <el-button type="success" icon="Check" circle/>
      <el-button type="info" icon="Message" circle/>
      <el-button type="warning" icon="Star" circle/>
      <el-button type="danger" icon="Delete" circle/>
</el-row>
import {
  Check,
  Delete,
  Edit,
  Message,
  Search,
  Star,
} from '@element-plus/icons-vue'
<!-- main.ts -->
import * as ElementPlusIconsVue from '@element-plus/icons-vue'
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
    app.component(key, component)
}
```
![8](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946601.png)
可以看到可以正常使用，控制台也没报错
```vue
<!-- 图标若不显示 -->
<!-- 将标签中的 :icon 改为icon即可 -->
```
新建一个头部组件
component下新建Header.vue
![9](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946541.png)
```vue
<template>
  <div style="height: 50px;line-height: 50px;border-bottom: 1px solid #ccc;display: flex">
    <div style="width: 200px;padding-left:30px;font-weight: bold;color:dodgerblue">管理系统</div>
    <div style="flex: 1"></div>
    <div style="width: 150px;margin-top: 10px;">
      <el-dropdown trigger="click">
        <span class="el-dropdown-link">
          深海火锅店<el-icon class="el-icon--right"><arrow-down/></el-icon>
        </span>
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item icon="UserFilled">
              个人信息
            </el-dropdown-item>
            <el-dropdown-item icon="SwitchButton" @click="logout">
              退出登录
            </el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </div>
</template>
<script lang="ts">
import {ref} from 'vue'
import { h } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import {
  ArrowDown,
  Check,
  CircleCheck,
  CirclePlus,
  CirclePlusFilled,
  Plus,
} from '@element-plus/icons-vue'
export default {
  name: "Header",
  setup() {
    const activeIndex = ref('1')
    const activeIndex2 = ref('1')
    const handleSelect = (key: string, keyPath: string[]) => {
      console.log(key, keyPath)
    }
    const logout=()=>{
      ElMessageBox({
        title: '提示',
        message: h('p', null, [
          h('span', null, '您即将'),
          h('i', { style: 'color: teal' }, '退出登录'),
        ]),
        showCancelButton: true,
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        beforeClose: (action, instance, done) => {
          if (action === 'confirm') {
            instance.confirmButtonLoading = true
            instance.confirmButtonText = 'Loading...'
            setTimeout(() => {
              done()
              setTimeout(() => {
                instance.confirmButtonLoading = false
              }, 300)
            }, 3000)
          } else {
            done()
          }
        },
      }).then((action) => {
        ElMessage({
          type: 'info',
          message: `action: ${action}`,
        })
      })
    }
    return {activeIndex, activeIndex2, handleSelect,logout}
  }
}
</script>
<style scoped>
</style>
```
暂时是定死了名字，用户名字是深海火锅店，以后再更改
此时我们发现文字和顶部之间有一段距离
在asset下新建css文件夹，css下新建global.css文件【右键新建，stylesheet，选择add】
```css
*{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}
```
在main.ts中引入
```js
// import '@/assets/css/global.css'添加这句话--------------
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import * as ElementPlusIconsVue from '@element-plus/icons-vue'
import '@/assets/css/global.css'
// @ts-ignore
import zhCn from 'element-plus/dist/locale/zh-cn.mjs'
const app = createApp(App)
app.use(store)
app.use(router)
app.use(ElementPlus)
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
    app.component(key, component)
}
app.mount('#app')
app.use(ElementPlus, {
    locale: zhCn,
})
```
再新建左侧导航栏Aside.vue
```vue
<template>
  <el-menu
      default-active="2"
      class="el-menu-vertical-demo"
      @open="handleOpen"
      @close="handleClose">
    <el-sub-menu index="1">
      <template #title>
        <el-icon>
          <location/>
        </el-icon>
        <span>学生信息管理</span>
      </template>
      <el-menu-item-group title="Group One">
        <el-menu-item index="1-1">item one</el-menu-item>
        <el-menu-item index="1-2">item two</el-menu-item>
      </el-menu-item-group>
      <el-menu-item-group title="Group Two">
        <el-menu-item index="1-3">item three</el-menu-item>
      </el-menu-item-group>
      <el-sub-menu index="1-4">
        <template #title>item four</template>
        <el-menu-item index="1-4-1">item one</el-menu-item>
      </el-sub-menu>
    </el-sub-menu>
    <el-menu-item index="2">
      <el-icon>
        <icon-menu/>
      </el-icon>
      <span>Navigator Two</span>
    </el-menu-item>
    <el-menu-item index="3" disabled>
      <el-icon>
        <document/>
      </el-icon>
      <span>Navigator Three</span>
    </el-menu-item>
    <el-menu-item index="4">
      <el-icon>
        <setting/>
      </el-icon>
      <span>Navigator Four</span>
    </el-menu-item>
  </el-menu>
</template>
<script lang="ts">
import {
  Document,
  Menu as IconMenu,
  Location,
  Setting,
} from '@element-plus/icons-vue'
export default {
  name: "Aside",
  setup() {
    const handleOpen = (key: string, keyPath: string[]) => {
      console.log(key, keyPath)
    }
    const handleClose = (key: string, keyPath: string[]) => {
      console.log(key, keyPath)
    }
    return {handleOpen, handleClose}
  }
}
</script>
<style scoped>
</style>
```
在App.vue中引入header、aside组件
```vue
<template>
  <el-config-provider :locale="locale">
    <slot name="app">
      <div class="common-layout">
        <el-container>
          <el-header>
            <Header/>
          </el-header>
          <el-container>
            <el-aside width="250px">
              <Aside/>
            </el-aside>
            <el-main>
              <router-view/>
            </el-main>
          </el-container>
        </el-container>
      </div>
    </slot>
  </el-config-provider>
</template>
<script lang="ts">
import {defineComponent} from 'vue';
import Header from "@/components/Header.vue";
import Aside from "@/components/Aside.vue";
import { ElConfigProvider } from 'element-plus'
// 国际化：设置为中文
// begin----
import zhCn from 'element-plus/lib/locale/lang/zh-cn'
export default defineComponent({
  name: 'ZhProvider',
  components: {
    Header,
    Aside,
    [ElConfigProvider.name]: ElConfigProvider
  },
  setup() {
    let locale = zhCn
    return {
      locale
    }
  }
});
// end----
</script>
<style>
</style>
```
最后完成主页面HomeView.vue
```vue
<template>
  <!--功能区域-->
  <div>
    <el-button type="info" icon="Plus" @click="openAdd">新增</el-button>
    <el-button type="info" icon="Upload">导入</el-button>
    <el-button type="info" icon="Download">导出</el-button>
  </div>
  <!--搜索区域-->
  <div style="margin: 10px 0">
    <el-input v-model="search" placeholder="请输入关键字" style="width: 25%" clearable/>
    <el-button type="primary" style="margin-left: 5px" icon="Search">搜索</el-button>
  </div>
  <el-table :data="tableData" style="width: 100%">
    <el-table-column fixed prop="date" label="日期" width="150" sortable/>
    <el-table-column prop="name" label="姓名" width="120"/>
    <el-table-column prop="age" label="年龄" width="120" sortable/>
    <el-table-column prop="gender" label="性别" width="120"/>
    <el-table-column prop="college" label="学院" width="120"/>
    <el-table-column prop="address" label="地址" width="300"/>
    <el-table-column prop="state" label="状态" width="120"/>
    <el-table-column fixed="right" label="操作" width="150">
      <template #default>
        <el-popconfirm type="warning" title="确认删除此选项?">
          <template #reference>
            <el-button link type="warning" size="small">删除</el-button>
          </template>
        </el-popconfirm>
        <el-button link type="primary" plain size="small" @click="handleClick">查看</el-button>
        <el-button link type="info" size="small" @click="editMsg">编辑</el-button>
      </template>
    </el-table-column>
  </el-table>
  <hr class="my-4"/>
  <!--分页区域-->
  <div class="demo-pagination-block" style="margin: 5px">
    <el-pagination
        v-model:current-page="currentPage"
        v-model:page-size="pageSize"
        :page-sizes="[100, 200, 300, 400]"
        :small="small"
        layout="total, sizes, prev, pager, next, jumper"
        :total="400"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"/>
  </div>
</template>
<script lang="ts">
import {defineComponent} from 'vue';
import {h,ref} from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { UploadFilled } from '@element-plus/icons-vue'
export default defineComponent({
  name: 'HomeView',
  components: {},
  setup() {
    /* 查看信息 */
    const handleClick = () => {
      console.log('click')
      ElMessageBox({
        title: '信息',
        message: h('p', null, [
          h('span', null, 'Message can be '),
          h('i', { style: 'color: teal' }, 'VNode'),
        ]),
      })
    }
    /* 编辑信息 */
    const editMsg = () => {
      ElMessageBox.confirm(
          '您正在修改他人的个人信息. 是否继续?',
          '警告',
          {
            confirmButtonText: '确认',
            cancelButtonText: '取消',
            type: 'warning',
          }
      )
          .then(() => {
            ElMessage({
              type: 'success',
              message: '修改成功',
            })
          })
          .catch(() => {
            ElMessage({
              type: 'info',
              message: '取消修改',
            })
          })
    }
    const search = ref("");
    const tableData = [
      {
        date: '2016-05-03',
        name: 'Tom',
        age: '12',
        gender: '女',
        college: '自动化',
        address: 'No. 189, Grove St, Los Angeles',
        state: '24小时阴性',
      },
      {
        date: '2016-05-03',
        name: 'Tom',
        age: '15',
        gender: '男',
        college: '经管',
        address: 'No. 189, Grove St, Los Angeles',
        state: '24小时阴性',
      },
      {
        date: '2016-05-03',
        name: 'Tom',
        age: '12',
        gender: '女',
        college: '计算机',
        address: 'No. 189, Grove St, Los Angeles',
        state: '24小时阴性',
      },
      {
        date: '2016-05-03',
        name: 'Tom',
        age: '16',
        gender: '男',
        college: '软件工程',
        address: 'No. 189, Grove St, Los Angeles',
        state: '24小时阴性',
      },
    ]
    const currentPage = ref(1)
    const pageSize = ref(100)
    const small = ref(false)
    const handleSizeChange = (val: number) => {
      console.log(`${val} items per page`)
    }
    const handleCurrentChange = (val: number) => {
      console.log(`current page: ${val}`)
    }
    /* 消息弹出框 */
    const openAdd = () => {
      ElMessageBox.confirm(
          '您希望添加如下内容?',
          '提示',
          {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            draggable: true,
          }
      )
          .then(() => {
            ElMessage({
              type: 'success',
              message: 'Delete completed',
            })
          })
          .catch(() => {
            ElMessage({
              type: 'info',
              message: 'Delete canceled',
            })
          })
    }
    return {
      handleClick,
      editMsg,
      tableData,
      search,
      currentPage,
      pageSize,
      small,
      handleSizeChange,
      handleCurrentChange,
      openAdd
    }
  }
});
</script>
```
![10](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201947673.png)
![11](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201947968.png)
![12](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201947582.png)
![13](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201947800.png)
链接：https://pan.baidu.com/s/11qXNPDQ_87TKmvF8R_RlrQ?pwd=m9fn 
提取码：m9fn 
--来自百度网盘超级会员V3的分享
