---
title: ClashX Pro 自定义域名解析失败
date: 2025-04-03T11:18:19+08:00
draft: false
tags:
  - 计算机科学
ShowToc: true
TocOpen: true
ShowWordCount: true
cover:
  image: "/images/ClashX Pro 自定义域名解析失败/mario-gogh-VBLHICVh-lI-unsplash.jpg"
  alt: "cover image"
  caption: "封面图来自 Unsplash | 作者 Mario Gogh"
  relative: false
---

## 应用介绍

ClashX Pro 是 MacOS 系统即苹果电脑系统下的代理软件客户端，功能强大且支持多种代理协议，如 Shadowsocks(SS)、ShadowsocksR(SSR)、Socks、Snell、V2Ray、Trojan 等代理协议

## 问题剖析

在使用 ClashX Pro 过程中，若设置内网 host 以实现自定义解析域名，却发现该设置失效。这是由于 ClashX Pro 启动后会自动接管 DNS，进而导致自定义解析域名功能无法正常运作。解决此问题的关键在于设置白名单

## 解决方案

> 基于 MacOS Big Sur 11.7.10 ｜ ClashX Pro 1.97.0.4

1. 操作路径：点击顶栏的 ClashX Pro 图标，依次选择 “配置” => “更多配置”，随即会弹出一个设置窗口
2. 白名单设置：在窗口底部，找到标题为 “忽略这些主机与域的代理设置” 的输入框，在此处添加需要忽略的自定义域名。多个域名之间用逗号分隔，同时支持使用通配符，完成添加后记得保存设置
3. 生效验证：进入 “网络” => “当前你的网络” => “高级” => “代理”，查看 “忽略这些主机与域的代理设置” 中是否已包含所设置的域名。若存在，则代表配置生效。需特别留意，必须关闭 ClashX Pro 的 [增强模式](https://github.com/Semporia/Clash/issues/5)，否则白名单将无法发挥作用
