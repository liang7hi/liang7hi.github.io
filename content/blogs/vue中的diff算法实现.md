---
title: Vue 中的 diff 算法实现
date: 2020-11-05T18:36:39+08:00
draft: false
tags:
  - 前端
ShowToc: true
TocOpen: true
ShowWordCount: true
summary: "介绍 React 和 Vue 中虚拟 DOM 替代直接操作 DOM 的原理，详细阐述 diff 算法如何比对新旧虚拟 DOM 实现高效更新"
cover:
  image: "/images/vue中的diff算法实现/cover.jpg"
  alt: "cover image"
  caption: "封面图来自 Unsplash | 作者 Андрей Сизов"
  relative: false
---

# Vue 中的 diff 算法实现

## 摘要

介绍 React 和 Vue 中虚拟 DOM 替代直接操作 DOM 的原理，详细阐述 diff 算法如何比对新旧虚拟 DOM 实现高效更新。

## 一、diff 算法和 virtual dom

![WechatIMG3177.png](/images/vue中的diff算法实现/HrGjhDp8ebSt62R.png)

在现代 `mvvm` 框架 React 和 Vue 中，直接操作 DOM 相对耗费更多性能。因此，React 和 Vue 都采用 `virtual dom` 来替代直接处理 DOM。大致过程为：依据 DOM 结构生成一份对象结构的 `virtual dom`。当数据更新需要反映在 DOM 上时，先将变化体现在 `virtual dom` 上，再通过 `diff` 算法对比数据更新前后的两次 `virtual dom`，找出差异。最后仅将这些改变的部分更新到 DOM 上，无需重新渲染整个页面。

简言之，`diff` 算法的作用就是找出两个 `virtual dom` 树结构的不同之处。

## 二、diff 原理

### （一）patch 函数

首先传入新旧 `vnode` 到 `patch` 函数开始比较：

```javascript
function patch(oldVnode, vnode) {
  if (sameVnode(oldVnode, vnode)) {
    patchVnode(oldVnode, vnode); // 如果两个vnode值得比较，调用patchVnode
  } else {
    const oEl = oldVnode.el; // oldVnode.el为对应的真实dom元素
    let parentEle = api.parentNode(oEl);
    createEle(vnode); // 按照vnode结构创建真实dom插入父元素，直接替换oldVnode
    if (parentEle !== null) {
      api.insertBefore(parentEle, vnode.el, api.nextSibling(oEl));
      api.removeChild(parentEle, oldVnode.el);
      oldVnode = null;
    }
  }
  return vnode;
}
```

### （二）sameVnode 函数

上述代码中，首先使用 `sameVnode` 方法判断两个 `vnode` 是否值得比较，代码如下：

```javascript
function sameVnode(oldVnode, vnode) {
  //  key代表当前节点数据的key属性，大概的意思就是div和span为不同标签。p.class1和p.class2为不同标签
  return vnode.key === oldVnode.key && vnode.sel === oldVnode.sel;
}
```

若两个 `vnode` 通过 `sameVnode` 比较结果为 `true`，则按照后续步骤，继续往下处理，执行 `patchVnode` 方法；若为 `false`，则直接用新元素替换旧元素。

### （三）patchVnode 函数

如果两个 `vnode` 值得比较，调用 `patchVnode`：

```javascript
patchVnode (oldVnode, vnode) {
    const el = vnode.el = oldVnode.el;
    let i, oldCh = oldVnode.children, ch = vnode.children;
    if (oldVnode === vnode) return; // 引用相同，就说明相同，直接return
    if (oldVnode.text!== null && vnode.text!== null && oldVnode.text!== vnode.text) {
        // 如果有文本内容，比较，如果不同直接设置，用新的替换旧的文本
        api.setTextContent(el, vnode.text);
    } else {
        updateEle(el, vnode, oldVnode);
        if (oldCh && ch && oldCh!== ch) { // 如果都有子节点，且不同，开始比较子节点
            updateChildren(el, oldCh, ch);
        } else if (ch) {
            // 如果只有新vnode上有子节点，直接新建
            createEle(vnode);
        } else if (oldCh) {
            // 如果新vnode没子子节，而老vnode有，则删除老节点的子节点
            api.removeChildren(el);
        }
    }
}
```

### （四）updateChildren 函数

如果新旧 `vnode` 都有子节点且不同，开始比较子节点，调用 `updateChildren` 函数开始比较子节点：

```javascript
updateChildren (parentElm, oldCh, newCh) { // parentElm：真是dom节点。oldCh：老vnode的所有子节点。newCh：新vnode的所有子节点。
    // 通过下标，设置新旧vnode的前后两个元素，
    let oldStartIdx = 0, newStartIdx = 0;
    let oldEndIdx = oldCh.length - 1;
    let oldStartVnode = oldCh[0];
    let oldEndVnode = oldCh[oldEndIdx];
    let newEndIdx = newCh.length - 1;
    let newStartVnode = newCh[0];
    let newEndVnode = newCh[newEndIdx];
    // 设置一些比较过程需要的变量
    let oldKeyToIdx;
    let idxInOld;
    let elmToMove;
    let before;
    // 两对节点，分别从头和尾向中间循环靠拢依次比对，如果起始下标大于结尾下标，证明比对结束，作为循环结束条件
    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (oldStartVnode == null) {   //对于vnode.key的比较，会把oldVnode = null
            oldStartVnode = oldCh[++oldStartIdx];
        } else if (oldEndVnode == null) {
            oldEndVnode = oldCh[--oldEndIdx];
        } else if (newStartVnode == null) {
            newStartVnode = newCh[++newStartIdx];
        } else if (newEndVnode == null) {
            newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldStartVnode, newStartVnode)) {
            patchVnode(oldStartVnode, newStartVnode);
            oldStartVnode = oldCh[++oldStartIdx];
            newStartVnode = newCh[++newStartIdx];
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
            patchVnode(oldEndVnode, newEndVnode);
            oldEndVnode = oldCh[--oldEndIdx];
            newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldStartVnode, newEndVnode)) {
            patchVnode(oldStartVnode, newEndVnode);
            api.insertBefore(parentElm, oldStartVnode.el, api.nextSibling(oldEndVnode.el));
            oldStartVnode = oldCh[++oldStartIdx];
            newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldEndVnode, newStartVnode)) {
            patchVnode(oldEndVnode, newStartVnode);
            api.insertBefore(parentElm, oldEndVnode.el, oldStartVnode.el);
            oldEndVnode = oldCh[--oldEndIdx];
            newStartVnode = newCh[++newStartIdx];
        } else {
            // 使用key时的比较
            if (oldKeyToIdx === undefined) {
                oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx); // 有key生成index表
            }
            idxInOld = oldKeyToIdx[newStartVnode.key];
            if (!idxInOld) {
                api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el);
                newStartVnode = newCh[++newStartIdx];
            } else {
                elmToMove = oldCh[idxInOld];
                if (elmToMove.sel!== newStartVnode.sel) {
                    api.insertBefore(parentElm, createEle(newStartVnode).el, oldStartVnode.el);
                } else {
                    patchVnode(elmToMove, newStartVnode);
                    oldCh[idxInOld] = null;
                    api.insertBefore(parentElm, elmToMove.el, oldStartVnode.el);
                }
                newStartVnode = newCh[++newStartIdx];
            }
        }
    }
    if (oldStartIdx > oldEndIdx) {
        before = newCh[newEndIdx + 1] == null? null : newCh[newEndIdx + 1].el;
        addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx);
    } else if (newStartIdx > newEndIdx) {
        removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx);
    }
}
```

### （五）总结

1. **调用 patch**：传入新旧 `vnode`，调用 `sameVnode`，返回 `true` 说明值得比较，则调用 `patchVnode`，否则直接使用新节点替换旧节点。
2. **调用 patchVnode**：更详细地对比新旧 `vnode`，分以下几种情况：
   - 若新旧 `vnode` 的 `.el` 真实 DOM 属性相同（即引用相同），直接 `return`。
   - 若存在文本节点且内容不同，直接用新 `vnode` 内文本替换旧 `vnode` 文本。
   - 判断子节点：若新 `vnode` 有子节点而旧 `vnode` 没有，则直接把新 `vnode` 的子节点加入到真实 DOM；反之，则删除真实 DOM 的子节点。
   - 若新旧 `vnode` 都有子节点且不同，调用 `updateChildren` 详细对比。
3. **调用 updateChildren**：从两边向中间收拢循环两个新旧 `vnode` 的每个子节点，依次对比头头、尾尾、头尾、尾头。若这四种比对调用 `sameVnode` 得到 `true`，证明值得对比，那么就调用 `patchVnode` 递归继续比较。除这四种情况外，若有 `key`，则会对比 `key`；否则直接替换新元素到真实 DOM。

## 三、结论

通过虚拟 DOM 和 diff 算法，Vue 以及 React 等框架实现了高效的 DOM 更新，减少了直接操作 DOM 带来的性能损耗。理解 diff 算法原理有助于开发者更好地优化前端应用，提升用户体验。在实际应用中，合理利用 key 值、优化数据结构等方式，能进一步发挥 diff 算法的优势，提升页面渲染效率。
