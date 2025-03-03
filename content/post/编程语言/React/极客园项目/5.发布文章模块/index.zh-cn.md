---
title: 5. 发布文章模块
description: 5. 发布文章模块
date: 2025-03-03
slug: 5. 发布文章模块
image: 202412212156937.png
categories:
    - React
---

> 本文转载自[B站 柴柴前端教书匠](https://www.bilibili.com/video/BV1Z44y1K7Fj/?spm_id_from=333.1387.0.0)

# 5. 发布文章模块

## 1. 基本结构搭建
`本节目标:`  能够搭建发布文章页面的基本结构



**实现步骤**

1. 使用Card、Form组件搭建基本页面结构
2. 创建样式文件，对样式做出调整



**代码实现**

`pages/Publish/index.js`

```jsx
import {
  Card,
  Breadcrumb,
  Form,
  Button,
  Radio,
  Input,
  Upload,
  Space,
  Select
} from 'antd'
import { PlusOutlined } from '@ant-design/icons'
import { Link } from 'react-router-dom'
import './index.scss'

const { Option } = Select

const Publish = () => {
  return (
    <div className="publish">
      <Card
        title={
          <Breadcrumb separator=">">
            <Breadcrumb.Item>
              <Link to="/home">首页</Link>
            </Breadcrumb.Item>
            <Breadcrumb.Item>发布文章</Breadcrumb.Item>
          </Breadcrumb>
        }
      >
        <Form
          labelCol={{ span: 4 }}
          wrapperCol={{ span: 16 }}
          initialValues={{ type: 1 }}
        >
          <Form.Item
            label="标题"
            name="title"
            rules={[{ required: true, message: '请输入文章标题' }]}
          >
            <Input placeholder="请输入文章标题" style={{ width: 400 }} />
          </Form.Item>
          <Form.Item
            label="频道"
            name="channel_id"
            rules={[{ required: true, message: '请选择文章频道' }]}
          >
            <Select placeholder="请选择文章频道" style={{ width: 400 }}>
              <Option value={0}>推荐</Option>
            </Select>
          </Form.Item>

          <Form.Item label="封面">
            <Form.Item name="type">
              <Radio.Group>
                <Radio value={1}>单图</Radio>
                <Radio value={3}>三图</Radio>
                <Radio value={0}>无图</Radio>
              </Radio.Group>
            </Form.Item>
            <Upload
              name="image"
              listType="picture-card"
              className="avatar-uploader"
              showUploadList
            >
              <div style={{ marginTop: 8 }}>
                <PlusOutlined />
              </div>
            </Upload>
          </Form.Item>
          <Form.Item
            label="内容"
            name="content"
            rules={[{ required: true, message: '请输入文章内容' }]}
          ></Form.Item>

          <Form.Item wrapperCol={{ offset: 4 }}>
            <Space>
              <Button size="large" type="primary" htmlType="submit">
                发布文章
              </Button>
            </Space>
          </Form.Item>
        </Form>
      </Card>
    </div>
  )
}

export default Publish
```



`pages/Publish/index.scss`

```css
.publish {
  position: relative;
}

.ant-upload-list {
  .ant-upload-list-picture-card-container,
  .ant-upload-select {
    width: 146px;
    height: 146px;
  }
}
```



## 2. 富文本编辑器
`本节目标:`  能够安装并初始化富文本编辑器



**实现步骤**

1. 安装富文本编辑器：`yarn add react-quill@2.0.0-beta.2`  [react-quill需要安装beta版本适配react18 否则无法输入中文]
2. 导入富文本编辑器组件以及样式文件
3. 渲染富文本编辑器组件
4. 通过 Form 组件的 `initialValues` 为富文本编辑器设置初始值，否则会报错
5. 调整富文本编辑器的样式



**代码实现**

`pages/Publish/index.js`

```jsx
import ReactQuill from 'react-quill'
import 'react-quill/dist/quill.snow.css'

const Publish = () => {
  return (
    // ...
    <Form
      labelCol={{ span: 4 }}
      wrapperCol={{ span: 16 }}
      // 注意：此处需要为富文本编辑表示的 content 文章内容设置默认值
      initialValues={{ content: '' }}
    >
      <Form.Item
        label="内容"
        name="content"
        rules={[{ required: true, message: '请输入文章内容' }]}
      >
        <ReactQuill
          className="publish-quill"
          theme="snow"
          placeholder="请输入文章内容"
        />
      </Form.Item>
    </Form>
  )
}
```



`pages/Publish/index.scss`

```css
.publish-quill {
  .ql-editor {
    min-height: 300px;
  }
}
```



## 3. 频道数据获取
`本节目标:` 实现频道数据的获取和渲染



**实现步骤**

1. 使用useState初始化数据和修改数据的方法
2. 在useEffect中调用接口并保存数据
3. 使用数据渲染对应模块



**代码实现**

```jsx
// 频道列表
const [channels, setChannels] = useState([])
    useEffect(() => {
    async function fetchChannels() {
      const res = await http.get('/channels')
      setChannels(res.data.channels)
    }
    fetchChannels()
}, [])

// 模板渲染
return (
 <Form.Item
    label="频道"
    name="channel_id"
    rules={[{ required: true, message: '请选择文章频道' }]}
  >
    <Select placeholder="请选择文章频道" style={{ width: 200 }}>
      {channels.map(item => (
        <Option key={item.id} value={item.id}>
          {item.name}
        </Option>
      ))}
    </Select>
  </Form.Item>
)
```



## 4. 上传封面实现
`本节目标:` 能够实现上传图片



**实现步骤**

1. 为 Upload 组件添加 action 属性，指定封面图片上传接口地址
2. 创建状态 fileList 存储已上传封面图片地址，并设置为 Upload 组件的 fileList 属性值
3. 为 Upload 添加 onChange 属性，监听封面图片上传、删除等操作
4. 在 change 事件中拿到当前图片数据，并存储到状态 fileList 中



**代码实现**

```jsx
import { useState } from 'react'

const Publish = () => {
  const [fileList, setFileList] = useState([])
  // 上传成功回调
  const onUploadChange = info => {
    const fileList = info.fileList.map(file => {
      if (file.response) {
        return {
          url: file.response.data.url
        }
      }
      return file
    })
    setFileList(fileList)
  }

  return (
    <Upload
      name="image"
      listType="picture-card"
      className="avatar-uploader"
      showUploadList
      action="http://geek.itheima.net/v1_0/upload"
      fileList={fileList}
      onChange={onUploadChange}
    >
      <div style={{ marginTop: 8 }}>
        <PlusOutlined />
      </div>
    </Upload>
  )
}
```



## 5.切换图片Type
`本节目标:` 实现点击切换图片类型



**实现步骤**

1. 创建状态 maxCount
2. 给 Radio 添加 onChange 监听单图、三图、无图的切换事件
3. 在切换事件中修改 maxCount 值
4. 只在 maxCount 不为零时展示 Upload 组件



**代码实现**

`pages/Publish/index.js`

```jsx
const Publish = () => {
  const [imgCount, setImgCount] = useState(1)

  const changeType = e => {
    const count = e.target.value
    setImgCount(count)
  }

  return (
    // ...
    <Form.Item label="封面">
      <Form.Item name="type">
        <Radio.Group onChange={changeType}>
          <Radio value={1}>单图</Radio>
          <Radio value={3}>三图</Radio>
          <Radio value={0}>无图</Radio>
        </Radio.Group>
      </Form.Item>
      {maxCount > 0 && (
        <Upload
          name="image"
          listType="picture-card"
          className="avatar-uploader"
          showUploadList
          action="http://geek.itheima.net/v1_0/upload"
        >
          <div style={{ marginTop: 8 }}>
            <PlusOutlined />
          </div>
        </Upload>
      )}
    </Form.Item>
  )
}
```



## 6. 控制最大上传数量
`本节目标:` 控制Upload组件的最大上传数量和是否支持多张图片



**实现步骤**

1. 修改 Upload 组件的 `maxCount（最大数量）`属性控制最大上传数量
2. 控制`multiple （支持多图选择）属性` 控制是否支持选择多张图片



**代码实现**

`pages/Publish/index.js`

```jsx
const Publish = () => {
  return (
    // ...
    <Form.Item label="封面">
      <Form.Item name="type">
        <Radio.Group onChange={changeType}>
          <Radio value={1}>单图</Radio>
          <Radio value={3}>三图</Radio>
          <Radio value={0}>无图</Radio>
        </Radio.Group>
      </Form.Item>
      {maxCount > 0 && (
        <Upload
          name="image"
          listType="picture-card"
          className="avatar-uploader"
          showUploadList
          action="http://geek.itheima.net/v1_0/upload"
          maxCount={ maxCount }
          multiple={ maxCount > 1 }
        >
          <div style={{ marginTop: 8 }}>
            <PlusOutlined />
          </div>
        </Upload>
      )}
    </Form.Item>
  )
}
```



## 7. 暂存图片列表实现
`本节目标:` 能够实现暂存已经上传的图片列表，能够在切换图片类型的时候完成切换



**问题描述**

如果当前为三图模式，已经完成了上传，选择单图只显示一张，再切换到三图继续显示三张，该如何实现？



**实现思路**

在上传完毕之后通过ref存储所有图片，需要几张就显示几张，其实也就是把ref当仓库，用多少拿多少



**实现步骤 （特别注意useState异步更新的坑）**

1. 通过useRef创建一个暂存仓库，在上传完毕图片的时候把图片列表存入
2. 如果是单图模式，就从仓库里取第一张图，以**数组的形式**存入fileList
3. 如果是三图模式，就把仓库里所有的图片，以**数组的形式**存入fileList



**代码实现**

```javascript
const Publish = () => {
  // 1. 声明一个暂存仓库
  const fileListRef = useRef([])
  
  // 2. 上传图片时，将所有图片存储到 ref 中
  const onUploadChange = info => {
    // ...
    fileListRef.current = imgUrls
  }
  
  // 3. 切换图片类型
  const changeType = e => {
    // 使用原始数据作为判断条件
    const count = e.target.value
    setMaxCount(count)

    if (count === 1) {
      // 单图，只展示第一张
      const firstImg = fileListRef.current[0]
      setFileList(!firstImg ? [] : [firstImg])
    } else if (count === 3) {
      // 三图，展示所有图片
      setFileList(fileListRef.current)
    }
  }

}
```

## 8. 发布文章实现
`本节目标:` 能够在表单提交时组装表单数据并调用接口发布文章



**实现步骤**

1.  给 Form 表单添加 `onFinish` 用来获取表单提交数据 
2.  在事件处理程序中，拿到表单数据按照接口需要格式化数据 
3.  调用接口实现文章发布，其中的接口数据格式为: 

```javascript
{
   channel_id: 1
   content: "<p>测试</p>"
   cover: {
      type: 1, 
      images: ["http://geek.itheima.net/uploads/1647066600515.png"]
   },
   type: 1
   title: "测试文章"
}
```

 

**代码实现**

```jsx
const Publish = () => {
    const onFinish = async (values) => {
    // 数据的二次处理 重点是处理cover字段
    const { channel_id, content, title, type } = values
    const params = {
      channel_id,
      content,
      title,
      type,
      cover: {
        type: type,
        images: fileList.map(item => item.response.data.url)
      }
    }
    await http.post('/mp/articles?draft=false', params)
  }
}
```



## 9. 编辑文章-文案适配
`本节目标:` 能够在编辑文章时展示数据



**实现步骤**

1. 通过路由参数拿到文章id
2. 根据文章 id 是否存在判断是否为编辑状态
3. 如果是编辑状态，展示编辑时的文案信息



**代码实现**

```jsx
import { useSearchParams } from 'react-router-dom'

const Publish = () => {
  const [params] = useSearchParams()
  const articleId = params.get('id')

  return (
    <Card
      title={
        <Breadcrumb separator=">">
          <Breadcrumb.Item>
            <Link to="/home">首页</Link>
          </Breadcrumb.Item>
          <Breadcrumb.Item>
            {articleId ? '修改文章' : '发布文章'}
          </Breadcrumb.Item>
        </Breadcrumb>
      }
    >
      // ...
      <Button size="large" type="primary" htmlType="submit">
        {articleId ? '修改文章' : '发布文章'}
      </Button>
  )
}
```



## 10.编辑文章-数据获取
`本节目标:` 使用id获取文章详情



> 判断文章 id 是否存在，如果存在就根据 id 获取文章详情数据
>

```javascript
useEffect(() => {
    async function getArticle () {
      const res = await http.get(`/mp/articles/${articleId}`)
    }
    if (articleId) {
      // 拉取数据回显
      getArticle()
    }
}, [articleId])
```



## 11. 编辑文章-回显Form
`本节目标:` 完成Form组件的回填操作

> 调用Form组件的实例对象方法 `setFieldsValue`
>

```javascript
useEffect(() => {
    async function getArticle () {
      const res = await http.get(`/mp/articles/${articleId}`)
      const { cover, ...formValue } = res.data
      // 动态设置表单数据
      form.setFieldsValue({ ...formValue, type: cover.type })
    }
    if (articleId) {
      // 拉取数据回显
      getArticle()
    }
}, [articleId])
```



## 12. 编辑文章-回显Upload相关


> 1.Upload回显列表 fileList   2. 暂存列表 cacheImgList   3. 图片数量 imgCount
>
> 核心要点：fileList和暂存列表要求格式统一
>



表单的赋值回显需要调用`setFieldsValue`方法，其中图片上传upload组件的回显依赖的数据格式如下：

```javascript
[
  { url: 'http://geek.itheima.net/uploads/1647066120170.png' }  
  ...
]
```



**代码实现**

```jsx
useEffect(() => {
    async function getArticle () {
      const res = await http.get(`/mp/articles/${articleId}`)
      const { cover, ...formValue } = res.data
      // 动态设置表单数据
      form.setFieldsValue({ ...formValue, type: cover.type })
      // 格式化封面图片数据
      const imageList = cover.images.map(url => ({ url }))
      setFileList(imageList)
      setMaxCount(cover.type)
      fileListRef.current = imageList
    }
    if (articleId) {
      // 拉取数据回显
      getArticle()
    }
}, [articleId])
```



## 11. 编辑保存
`本节目标:` 能够在编辑文章时对文章进行修改

**代码实现**

```javascript
// 提交表单
const onFinish = async (values) => {
    const { type, ...rest } = values
    const data = {
      ...rest,
      // 注意：接口会按照上传图片数量来决定单图 或 三图
      cover: {
        type,
        images: fileList.map(item => item.url)
      }
    }
    if(articleId){
      // 编辑
      await http.put(`/mp/articles/${data.id}?draft=false`,data)
    }else{
      // 新增
      await http.post('/mp/articles?draft=false', data)
    }
}
```



# 




> 更新: 2022-07-08 12:19:01  
> 原文: <https://www.yuque.com/fechaichai/tzzlh1/oh9sx3>