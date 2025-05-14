---
title: React 权限控制
description: React 权限控制
date: 2025-04-21
slug: React 权限控制
image: 202412212156937.png
categories:
  - React
---

# React 权限控制

> auth.ts

```ts
export enum RoleEnum {
  ADMIN = 'admin',
  DEVLOPER = 'devloper',
  USER = 'user',
}

type Role = keyof typeof ROLES
type Permission = (typeof ROLES)[Role][number]

const ROLES = {
  [RoleEnum.ADMIN]: [
    'view:comment',
    'create:comment',
    'update:comment',
    'delete:comment',
  ],
  [RoleEnum.DEVLOPER]: ['view:comment', 'create:comment', 'update:comment'],
  [RoleEnum.USER]: ['view:comment'],
} as const

export function hasPermission(
  user: { id: number; role: Role },
  permission: Permission
) {
  return (ROLES[user.role] as readonly Permission[]).includes(permission)
}
```

> `src/pages/user/[id]/index.tsx`

```tsx
import useSWR from 'swr'
import { useEffect, useState } from 'react'
import { useParams } from 'react-router-dom'
import { commentService } from '../../../services'
import { hasPermission, RoleEnum } from '../../../utils/auth'

function CommentId() {
  // 获取当前路径参数
  const { id } = useParams()
  const [currentUser, setCurrentUser] = useState({
    id: 0,
    role: RoleEnum.USER,
  })
  const { data, error, isLoading } = useSWR(
    '/comment',
    commentService.getComment
  )
  const commentData = data?.data
  // 初始化用户
  useEffect(() => {
    switch (id) {
      case '1':
        setCurrentUser({
          id: Number(id),
          role: RoleEnum.ADMIN,
        })
        break
      case '2':
        setCurrentUser({
          id: Number(id),
          role: RoleEnum.DEVLOPER,
        })
        break
      default:
        setCurrentUser({
          id: Number(id),
          role: RoleEnum.USER,
        })
    }
  }, [id])
  if (error) return <div>failed to load</div>
  if (isLoading) return <div>loading...</div>
  return (
    <div>
      <div className="relative overflow-x-auto shadow-md sm:rounded-lg">
        <table className="w-full text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400">
          <thead className="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
            <tr>
              <th scope="col" className="px-6 py-3">
                id
              </th>
              <th scope="col" className="px-6 py-3">
                用户id
              </th>
              <th scope="col" className="px-6 py-3">
                内容
              </th>
              <th scope="col" className="px-6 py-3">
                操作
              </th>
            </tr>
          </thead>
          <tbody>
            {commentData?.map((comment) => {
              return (
                <tr
                  key={comment.id}
                  className="odd:bg-white odd:dark:bg-gray-900 even:bg-gray-50 even:dark:bg-gray-800 border-b dark:border-gray-700 border-gray-200">
                  <th
                    scope="row"
                    className="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white">
                    {comment.id}
                  </th>
                  <td className="px-6 py-4">{comment.userId}</td>
                  <td className="px-6 py-4">{comment.comment}</td>
                  <td className="px-6 py-4">
                    {/* 查看权限 */}
                    {hasPermission(currentUser, 'view:comment') && (
                      <button className="bg-yellow-500 hover:bg-yellow-700 text-white font-bold py-2 px-4 rounded mr-2">
                        查看
                      </button>
                    )}
                    {/* 增加权限 */}
                    {hasPermission(currentUser, 'create:comment') && (
                      <button className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mr-2">
                        增加
                      </button>
                    )}
                    {/* 编辑权限 */}
                    {hasPermission(currentUser, 'update:comment') && (
                      <button className="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded mr-2">
                        编辑
                      </button>
                    )}
                    {/* 删除权限 */}
                    {hasPermission(currentUser, 'delete:comment') && (
                      <button className="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded mr-2">
                        删除
                      </button>
                    )}
                  </td>
                </tr>
              )
            })}
          </tbody>
        </table>
      </div>
    </div>
  )
}

export default CommentId
```

效果如下

> 当用户 id 为 1(管理员)时

![202505122246306.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122246306.png)

> 当用户 id 为 2(开发者)时

![202505122247253.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122247253.png)

> 当用户 id 为其他时(普通用户)

![202505122247598.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122247598.png)
以上就实现了建议版的权限控制，但是这个权限控制仍然具有局限性
如果我们希望修改某个用户的权限，只能修改对应角色的权限，但是这样下来就修改了批量的用户
或者为这个用户新增一个单独的角色，但是怎么想也还是不够优雅

> 举个例子，我希望当用户可以修改和删除自己的评论，该如何实现

```ts
export enum RoleEnum {
  ADMIN = 'admin',
  DEVLOPER = 'devloper',
  USER = 'user',
}

type Role = keyof typeof ROLES
type Permission = (typeof ROLES)[Role][number]

const ROLES = {
  [RoleEnum.ADMIN]: [
    'view:comment',
    'create:comment',
    'update:comment',
    'delete:comment',
  ],
  [RoleEnum.DEVLOPER]: [
    'view:comment',
    'create:comment',
    'update:comment',
    // 新增删除自己的评论
    'delete:ownerComment',
  ],
  [RoleEnum.USER]: [
    'view:comment',
    // 新增修改删除自己的评论
    'update:ownerComment',
    'delete:ownerComment',
  ],
} as const

export function hasPermission(
  user: { id: number; role: Role },
  permission: Permission
) {
  return (ROLES[user.role] as readonly Permission[]).includes(permission)
}
```

```tsx
{
  /* 编辑权限 */
}
{
  /* 是否有编辑权限 */
}
{
  ;(hasPermission(currentUser, 'update:comment') ||
    // 是否是作者而且具有[update:ownerComment]的权限
    (hasPermission(currentUser, 'update:ownerComment') &&
      currentUser.id === comment.userId)) && (
    <button className="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded mr-2">
      编辑
    </button>
  )
}
{
  /* 删除权限 */
}
{
  ;(hasPermission(currentUser, 'delete:comment') ||
    // 是否是作者而且具有[delete:ownerComment]的权限
    (hasPermission(currentUser, 'delete:ownerComment') &&
      currentUser.id === comment.userId)) && (
    <button className="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded mr-2">
      删除
    </button>
  )
}
```

实现效果如下
管理员具有全部权限
![202505122247080.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122247080.png)
开发者的情况下，显示自己内容的删除按钮
![202505122248006.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122248006.png)

普通用户的情况下，显示自己内容的编辑和删除按钮
![202505122248136.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo04/202505122248136.png)
但是这样判断权限还是很复杂，判断语句写了一层又一层，如果说我们需要更加精细的权限控制呢，比如只允许删除特定状态的评论(审核中、已发布)
