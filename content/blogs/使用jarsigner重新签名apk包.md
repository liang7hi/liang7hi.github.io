---
title: 使用jarsigner重新签名apk包
date: 2020-10-26T16:41:13+08:00
draft: false
tags:
  - 计算机科学
ShowToc: true
TocOpen: true
ShowWordCount: true
cover:
  image: "/images/使用jarsigner重新签名apk包/cover.jpg"
  alt: "cover image"
  caption: "封面图来自 Unsplash | 作者 iMattSmart"
  relative: false
---

## 环境

- macos 10.14
- iterm 2

## 使用

### 生成密钥

keytool -genkeypair -alias <别名> -keystore <密钥库存储位置和名称>

```shell
keytool -genkeypair -alias demokey -keystore demo.keystore
```

内容填写后，生成完毕

### 生成签名

jarsigner -verbose -keystore <使用密钥库位置> -signedjar <签名后的包保存位置和名字> <要签名的包保存位置和名字> <别名，对应上面的别名>

```shell
jarsigner -verbose -keystore /Users/liangqi/Desktop/apk/demo.keystore -signedjar ./tap_signed.apk ./tap_unsign.apk demokey
```

即可拿到签名后的包
