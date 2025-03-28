---
title: 使用 css 实现环状进度
date: 2020-08-22T09:44:03+08:00
draft: false
tags:
  - 前端
ShowToc: true
TocOpen: true
ShowWordCount: true
summary: "使用纯 css 实现一个圆环的进度效果，主要使用了`background-image`渐变属性和动画状态的`伪元素遮盖`来实现"
cover:
  image: "/images/使用css实现环状进度/cover.jpg"
  alt: "cover image"
  caption: "封面图来自 Unsplash | 作者 Pankaj Patel"
  relative: false
---

### html

```html
<div class="round"></div>
```

### css

使用`background-image`渐变属性和动画状态的`伪元素遮盖`来实现：

```css
.round {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background-color: #eee;
  background-image: linear-gradient(to right, transparent 50%, blue 0);
  position: relative;
}
.round::before {
  width: 50%;
  height: 100%;
  content: "";
  position: absolute;
  top: 0;
  left: 50%;
  border-radius: 0 100% 100% 0 / 50%;
  background-color: inherit;
  transform-origin: left;
  animation: spin 5s linear infinite, dye 10s step-end infinite;
}
@keyframes spin {
  to {
    transform: rotate(0.5turn);
  }
}
@keyframes dye {
  50% {
    background: blue;
  }
}s
```

### 效果

![demo.gif](/images/使用css实现环状进度/demo.gif)
