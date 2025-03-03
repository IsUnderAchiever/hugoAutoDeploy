---
title: 4. 文章管理模块
description: 4. 文章管理模块
date: 2025-03-03
slug: 4. 文章管理模块
image: 202412212156937.png
categories:
    - React
---

> 本文转载自[B站 柴柴前端教书匠](https://www.bilibili.com/video/BV1Z44y1K7Fj/?spm_id_from=333.1387.0.0)

# 4. 文章管理模块

## 1. 筛选区结构
`本节目标:` 能够使用antd组件库搭建筛选区域结构



![202503032212605.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202503032212605.png)



> 重点关注
>
> 1.  如何让RangePicker日期范围选择框选择中文 
> 2.  Select组件配合Form.Item使用时，如何配置默认选中项  
`<Form initialValues={{ status: null }} >` 
>



**代码实现**

```jsx
import { Link } from 'react-router-dom'
import { Card, Breadcrumb, Form, Button, Radio, DatePicker, Select } from 'antd'
import 'moment/locale/zh-cn'
import locale from 'antd/es/date-picker/locale/zh_CN'
import './index.scss'

const { Option } = Select
const { RangePicker } = DatePicker

const Article = () => {
  return (
    <div>
      <Card
        title={
          <Breadcrumb separator=">">
            <Breadcrumb.Item>
              <Link to="/home">首页</Link>
            </Breadcrumb.Item>
            <Breadcrumb.Item>内容管理</Breadcrumb.Item>
          </Breadcrumb>
        }
        style={{ marginBottom: 20 }}
      >
        <Form initialValues={{ status: null }}>
          <Form.Item label="状态" name="status">
            <Radio.Group>
              <Radio value={null}>全部</Radio>
              <Radio value={0}>草稿</Radio>
              <Radio value={1}>待审核</Radio>
              <Radio value={2}>审核通过</Radio>
              <Radio value={3}>审核失败</Radio>
            </Radio.Group>
          </Form.Item>

          <Form.Item label="频道" name="channel_id">
            <Select
              placeholder="请选择文章频道"
              defaultValue="lucy"
              style={{ width: 120 }}
            >
              <Option value="jack">Jack</Option>
              <Option value="lucy">Lucy</Option>
            </Select>
          </Form.Item>

          <Form.Item label="日期" name="date">
            {/* 传入locale属性 控制中文显示*/}
            <RangePicker locale={locale}></RangePicker>
          </Form.Item>

          <Form.Item>
            <Button type="primary" htmlType="submit" style={{ marginLeft: 80 }}>
              筛选
            </Button>
          </Form.Item>
        </Form>
      </Card>
    </div>
  )
}

export default Article
```



## 2. 表格区域结构
`本节目标:` 能够基于Table组件搭建表格结构



> 重点关注
>
> 1. 通过哪个属性指定Table组件的列信息
> 2. 通过哪个属性指定Table数据
> 3. 通过哪个属性指定Table列表用到的key属性
>



**代码实现**

```jsx
import { Link } from 'react-router-dom'
import { Table, Tag, Space } from 'antd'
import { EditOutlined, DeleteOutlined } from '@ant-design/icons'

import img404 from '@/assets/error.png'

const Article = () => {
  const columns = [
    {
      title: '封面',
      dataIndex: 'cover',
      width:120,
      render: cover => {
        return <img src={cover || img404} width={80} height={60} alt="" />
      }
    },
    {
      title: '标题',
      dataIndex: 'title',
      width: 220
    },
    {
      title: '状态',
      dataIndex: 'status',
      render: data => <Tag color="green">审核通过</Tag>
    },
    {
      title: '发布时间',
      dataIndex: 'pubdate'
    },
    {
      title: '阅读数',
      dataIndex: 'read_count'
    },
    {
      title: '评论数',
      dataIndex: 'comment_count'
    },
    {
      title: '点赞数',
      dataIndex: 'like_count'
    },
    {
      title: '操作',
      render: data => {
        return (
          <Space size="middle">
            <Button type="primary" shape="circle" icon={<EditOutlined />} />
            <Button
              type="primary"
              danger
              shape="circle"
              icon={<DeleteOutlined />}
            />
          </Space>
        )
      }
    }
  ]

  const data = [
      {
          id: '8218',
          comment_count: 0,
          cover: {
            images:['http://geek.itheima.net/resources/images/15.jpg'],
          },
          like_count: 0,
          pubdate: '2019-03-11 09:00:00',
          read_count: 2,
          status: 2,
          title: 'wkwebview离线化加载h5资源解决方案' 
      }
  ]
  
  return (
    <div>
      <Card title={`根据筛选条件共查询到 count 条结果：`}>
        <Table rowKey="id" columns={columns} dataSource={data} />
      </Card>
    </div>
  )
}
```



## 3. 渲染频道数据
`本节目标:`  使用接口数据渲染频道列表



**实现步骤**

1. 使用axios获取数据
2. 将使用频道数据列表改写下拉框组件



**代码实现**

`pages/Article/index.js`

```jsx
// 获取频道列表
const [channels, setChannels] = useState([])
useEffect(() => {
    async function fetchChannels() {
      const res = await http.get('/channels')
      setChannels(res.data.channels)
    }
    fetchChannels()
}, [])

// 渲染模板
return (
<Form.Item label="频道" name="channel_id" >
    <Select placeholder="请选择文章频道" style={{ width: 200 }} >
      {channels.map(item => (
        <Option key={item.id} value={item.id}>
          {item.name}
        </Option>
      ))}
    </Select>
</Form.Item>
)
```



## 4. 渲染表格数据
`本节目标:`  使用接口数据渲染表格数据



**实现步骤**

1. 声明列表相关数据管理
2. 声明参数相关数据管理
3. 调用接口获取数据
4. 使用接口数据渲染模板



**代码实现**

```jsx
// 文章列表数据管理
const [article, setArticleList] = useState({
    list: [],
    count: 0
})

// 参数管理
const [params, setParams] = useState({
    page: 1,
    per_page: 10
})

// 发送接口请求
useEffect(() => {
    async function fetchArticleList() {
      const res = await http.get('/mp/articles', { params })
      const { results, total_count } = res.data
      setArticleList({
        list: results,
        count: total_count
      })
    }
    fetchArticleList()
}, [params])

// 模板渲染
return (
 <Card title={`根据筛选条件共查询到 ${article.count} 条结果：`}>
    <Table
      dataSource={article.list}
      columns={columns}
    />
 </Card>
)
```



## 5. 筛选功能实现
`本节目标:` 能够根据筛选条件筛选表格数据



**实现步骤**

1. 为表单添加`onFinish`属性监听表单提交事件，获取参数
2. 根据接口字段格式要求格式化参数格式
3. 修改`params` 触发接口的重新发送



**代码实现**

```jsx
// 筛选功能
const onSearch = values => {
    const { status, channel_id, date } = values
    // 格式化表单数据
    const _params = {}
    // 格式化status
    _params.status = status
    if (channel_id) {
      _params.channel_id = channel_id
    }
    if (date) {
      _params.begin_pubdate = date[0].format('YYYY-MM-DD')
      _params.end_pubdate = date[1].format('YYYY-MM-DD')
    }
    // 修改params参数 触发接口再次发起
    setParams({
       ...params,
       ..._params
    })
}
// Form绑定事件
return (
    <Form onFinish={ onSearch }></Form>
)
```



## 6. 分页功能实现
`本节目标:`  能够实现分页获取文章列表数据



**实现步骤**

1. 为Table组件指定pagination属性来展示分页效果
2. 在分页切换事件中获取到筛选表单中选中的数据
3. 使用当前页数据修改params参数依赖引起接口重新调用获取最新数据



**代码实现**

```jsx
const pageChange = (page) => {
    // 拿到当前页参数 修改params 引起接口更新
    setParams({
      ...params,
      page
    })
}

return (
   <Table
      dataSource={article.list}
      columns={columns}
      pagination={{
        position: ['bottomCenter'],
        current: params.page,
        pageSize: params.per_page,
        onChange: pageChange
      }}
    />
)
```



## 7. 删除功能
`本节目标:`  能够实现点击删除按钮弹框确认



**实现步骤**

1. 给删除文章按钮绑定点击事件
2. 弹出确认窗口，询问用户是否确定删除文章
3. 拿到参数调用删除接口，更新列表



**代码实现**

```jsx
// 删除回调
const delArticle = async (data) => {
    await http.delete(`/mp/articles/${data.id}`)
    // 更新列表
    setParams({
      page: 1,
      per_page: 10
    })
}

const columns = [
  // ...
  {
      title: '操作',
      render: data => {
        return (
          <Space size="middle">
            <Button type="primary" shape="circle" icon={<EditOutlined />} />
            <Popconfirm
              title="确认删除该条文章吗?"
              onConfirm={() => delArticle(data)}
              okText="确认"
              cancelText="取消"
            >
              <Button
                type="primary"
                danger
                shape="circle"
                icon={<DeleteOutlined />}
              />
            </Popconfirm>
          </Space>
        )
      }
]
```



## 8. 编辑文章跳转
`本节目标:`  能够实现编辑文章跳转功能



**代码实现**

```jsx
const columns = [
  // ...
  {
    title: '操作',
    render: data => (
      <Space size="middle">
        <Button
          type="primary"
          shape="circle"
          icon={<EditOutlined />}
          onClick={() => history.push(`/home/publish?id=${data.id}`)}
        />
      </Space>
    )
  }
]
```



# 


> 更新: 2022-07-07 21:30:58  
> 原文: <https://www.yuque.com/fechaichai/tzzlh1/rm4hzp>