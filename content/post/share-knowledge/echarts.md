---
title: "Echarts常见问题"
description: 
date: 2025-01-08T07:08:05Z
image: 
draft: true
tags: [echarts]
categories: []
---

## echarts 首次渲染高度塌陷问题

echart的高度如果给的是一个百分比，元素高度可能会出现塌陷，因为DOM元素的高度并没有及时计算好。

## echarts 添加可拖拽标签

使用echarts的graphic功能，画出标签，并监听graphic的ondrag、ondragstart、ondragend事件
