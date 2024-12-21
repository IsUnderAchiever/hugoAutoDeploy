---
title: 媒介查询
description: 媒介查询
date: 2024-02-15
slug: 媒介查询
image: 202412211946837.png
categories:
    - Vue
---

```vue
<template>
  <div class="header-div"></div>
</template>
<script setup lang="ts">
</script>
<style lang="scss" scoped>
$breakPoints: (
    'phone':(
        320px,
        480px
    ),
    'pad':(
        481px,
        768px
    ),
    'notebook':(
        769px,
        1024px
    ),
    'desktop':(
        1025px,
        1200px
    ),
    'tv':(
        1201px
    ),
);
// 混合
@mixin respond-to($breakName) {
  $bp: map-get($breakPoints, $breakName);
  @if type-of($bp)== 'list' {
    $min: nth($bp, 1);
    $max: nth($bp, 2);
    @media (min-width: $min) and (max-width: $max) {
      @content;
    }
  }
  @else {
    @media (min-width: $bp) {
      @content;
    }
  }
}
.header-div {
  width: 100%;
  @include respond-to('phone') {
    background-color: red;
    height: 40px;
  }
  @include respond-to('pad') {
    height: 60px;
  }
  @include respond-to('notebook') {
    background-color: green;
    height: 80px;
  }
}
</style>
```
