---
title: Flowable
description: Flowable
date: 2024-02-15
slug: Git
image: 202412212118817.png
categories:
    - Flowable
---

# Flowable
| 一般数据对应表   |                          |
| ---------------- | ------------------------ |
| act_ge_bytearray | 通用的流程定义和流程资源 |
| act_ge_property  | 系统相关属性             |

| 流程历史记录表      |                              |
| ------------------- | ---------------------------- |
| act_hi_actinst      | 历史的流程实例               |
| act_hi_attachment   | 历史的流程附件               |
| act_hi_comment      | 历史的说明性信息             |
| act_hi_detail       | 历史的流程运行中的细节信息   |
| act_hi_entitylink   | 任务参与者数据表             |
| act_hi_identitylink | 历史的流程运行过程中用户关系 |
| act_hi_procinst     | 历史的流程实例               |
| act_hi_taskinst     | 历史的任务实例               |
| act_hi_tsk_log      | 每一次执行可能会带上的数据   |
| act_hi_varinst      | 历史的流程运行中的变量信息   |

| 用户用户组表        |                    |
| ------------------- | ------------------ |
| act_id_bytearray    | 二进制数据表       |
| act_id_group        | 用户组信息表       |
| act_id_info         | 用户信息详情表     |
| act_id_membership   | 人与组关系表       |
| act_id_priv         | 权限表             |
| act_id_priv_mapping | 用户或组权限关系表 |
| act_id_property     | 属性表             |
| act_id_token        | 系统登录日志表     |
| act_id_user         | 用户表             |

| 流程定义表        |                  |
| ----------------- | ---------------- |
| act_re_deployment | 部署单元信息     |
| act_re_model      | 模型信息         |
| act_re_procdef    | 已部署的流程定义 |

| 运行实例表            |                                            |
| --------------------- | ------------------------------------------ |
| act_ru_actinst        | 流程实例每一个活动对应的状态表             |
| act_ru_deadletter_job | 作业失败表，失败次数>重试次数              |
| act_ru_entitylink     | 实例的父子关系表                           |
| act_ru_event_subscr   | 运行时事件                                 |
| act_ru_execution      | 运行时流程执行实例                         |
| act_ru_external_job   | 使用作业表来实现异步逻辑、计时器或历史处理 |
| act_ru_history_job    | 历史作业表                                 |
| act_ru_identitylink   | 用户或组的数据及其与流程实例相关的角色表   |
| act_ru_job            | 运行时作业表                               |
| act_ru_suspended_job  | 运行时挂起的定时作业表                     |
| act_ru_task           | 正在运行的实例的每个未完成用户任务的条目   |
| act_ru_timer_job      | 运行时定时器表                             |
| act_ru_variable       | 实例相关的变量                             |

| 其他表           |              |
| ---------------- | ------------ |
| act_evt_log      | 事件日志表   |
| act_procdef_info | 流程定义信息 |

![image-20231204075630119](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202312040756384_repeat_1701647799480__678008.png)
![image-20231204075743701](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202312040757810_repeat_1701647863829__348359.png)
![image-20231204080127639](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202312040801820_repeat_1701648087833__670878.png)
![image-20231204080144734](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202312040801844_repeat_1701648104859__083755.png)
