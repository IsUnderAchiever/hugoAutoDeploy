---
title: 15.keepalive缓存组件
description: 15.keepalive缓存组件
date: 2024-06-01
slug: 15.keepalive缓存组件
image: 202412211946837.png
categories:
    - Vue
---

# **内置组件keep-alive**
有时候我们不希望组件被重新渲染影响使用体验；或者处于性能考虑，避免多次重复渲染降低性能。而是希望组件可以缓存下来,维持当前的状态。这时候就需要用到`keep-alive`组件。
开启keep-alive 生命周期的变化
- 初次进入时： onMounted> onActivated
- 退出后触发 `deactivated`
- 再次进入只会触发 onActivated
- 事件挂载的方法等，只执行一次的放在 onMounted中；组件每次进去执行的方法放在 onActivated中
```
<!-- 基本 -->
<keep-alive>
  <component :is="view"></component>
</keep-alive>
<!-- 多个条件判断的子组件 -->
<keep-alive>
  <comp-a v-if="a > 1"></comp-a>
  <comp-b v-else></comp-b>
</keep-alive>
<!-- 和 `<transition>` 一起使用 -->
<transition>
  <keep-alive>
    <component :is="view"></component>
  </keep-alive>
</transition>
```
 **`include` 和 `exclude`**
```
 <keep-alive :include="" :exclude="" :max=""></keep-alive>
```
include 和 exclude 允许组件有条件地缓存。二者都可以用逗号分隔字符串、正则表达式或一个数组来表示：
**`max`**
```
<keep-alive :max="10">
  <component :is="view"></component>
</keep-alive>
```
# keep-alive 源码讲解
源码目录runtime-core/src/components/KeepAlive.ts
### props
```
 props: {
    include: [String, RegExp, Array], // 配置了该属性，那么只有名称匹配的组件会被缓存
    exclude: [String, RegExp, Array], // 配置了该属性，那么任何名称匹配的组件都不会被缓存
    max: [String, Number]// 最多可以缓存多少组件实例
  },
  setup(props: KeepAliveProps, { slots }: SetupContext) {
    const instance = getCurrentInstance()!
    const sharedContext = instance.ctx as KeepAliveContext
  })
      const { include, exclude, max } = props
        //如果include 子组件名称 不包含， 和  exclude包含的名字 就不该缓存 直接返回
      if (
        (include && (!name || !matches(include, name))) ||
        (exclude && name && matches(exclude, name))
      ) {
        current = vnode
        return rawVNode
      }
```
 
```
    watch(
      () => [props.include, props.exclude],
      ([include, exclude]) => {
        //props 是响应式的 所以这个操作需要在做一遍
        include && pruneCache(name => matches(include, name))
        exclude && pruneCache(name => !matches(exclude, name))
      },
      // prune post-render after `current` has been updated
      { flush: 'post', deep: true }
    )
```
### 缓存设计
```
    //KeepAlive 组件的缓存管理
    const cache: Cache = new Map()
    const keys: Keys = new Set()
    let pendingCacheKey: CacheKey | null = null
    //缓存函数
    const cacheSubtree = () => {
      // fix #1621, the pendingCacheKey could be 0
      if (pendingCacheKey != null) {
        cache.set(pendingCacheKey, getInnerChild(instance.subTree))
      }
    }
    //KeepLive 组件中对缓存的管理时，首先会在组件的 onMounted 或 onUpdated 生命周期钩子函数中设置缓存，如下代码所示：
    onMounted(cacheSubtree)
    onUpdated(cacheSubtree)
   //Vnode 的key 作为缓存的key
      pendingCacheKey = key
      //KeepLive组件返回的函数中根据 vnode 对象的 key 去缓存中查找是否有缓存的组件，
       //如果缓存存在，则继承组件实例，并将用于描述组件的 vnode 对象标记为 COMPONENT_KEPT_ALIVE，
       //这样渲染器就不会重新创建新的组件实例；如果缓存不存在，则将 vnode 对象的key 添加到 keys 集合中
      if (cachedVNode) {
        // 如果缓存中存在缓存的组件
        vnode.el = cachedVNode.el
        vnode.component = cachedVNode.component
        //无须创建组件实例，继承缓存的组件即可
        if (vnode.transition) {
          // 如果组件上有动画，处理动画
          setTransitionHooks(vnode, vnode.transition!)
        }
        // 标记Vnode 不会重新渲染
        vnode.shapeFlag |= ShapeFlags.COMPONENT_KEPT_ALIVE
        // make this key the freshest
        keys.delete(key)
        keys.add(key)
      } else {
        //将 vnode 的key 添加到 keys 集合中，keys 集合用户维护缓存组件的 key
        keys.add(key)
        // prune oldest entry
        if (max && keys.size > parseInt(max as string, 10)) {
          pruneCacheEntry(keys.values().next().value)
        }
      }
```
### 生命周期
```
   //隐藏容器
    const storageContainer = createElement('div')
    //在实例上注册两个钩子函数 activate，  deactivate
    sharedContext.activate = (vnode, container, anchor, isSVG, optimized) => {
      const instance = vnode.component!
      move(vnode, container, anchor, MoveType.ENTER, parentSuspense)
      // props 可能会发生变化，因此需要执行 patch 过程
      patch(
        instance.vnode,
        vnode,
        container,
        anchor,
        instance,
        parentSuspense,
        isSVG,
        vnode.slotScopeIds,
        optimized
      )
      queuePostRenderEffect(() => {
        instance.isDeactivated = false
        if (instance.a) {
          invokeArrayFns(instance.a)
        }
        const vnodeHook = vnode.props && vnode.props.onVnodeMounted
        if (vnodeHook) {
          invokeVNodeHook(vnodeHook, instance.parent, vnode)
        }
      }, parentSuspense)
      if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
        // Update components tree
        devtoolsComponentAdded(instance)
      }
    }
    sharedContext.deactivate = (vnode: VNode) => {
      const instance = vnode.component!
      //将组件移动到隐藏容器中
      //在 “卸载” 组件时，并不是真正的卸载，而是调用 move 方法，将组件搬运到一个隐藏的容器中
      move(vnode, storageContainer, null, MoveType.LEAVE, parentSuspense)
      queuePostRenderEffect(() => {
        if (instance.da) {
          invokeArrayFns(instance.da)
        }
        const vnodeHook = vnode.props && vnode.props.onVnodeUnmounted
        if (vnodeHook) {
          invokeVNodeHook(vnodeHook, instance.parent, vnode)
        }
        instance.isDeactivated = true
      }, parentSuspense)
      if (__DEV__ || __FEATURE_PROD_DEVTOOLS__) {
        // Update components tree
        devtoolsComponentAdded(instance)
      }
    }
```
 
