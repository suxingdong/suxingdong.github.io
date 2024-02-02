---
title: 六角网格
author: East.Su
date: 2019-08-11 00:34:00 +0800
categories: [Blogging, Tutorial]
tags: [favicon]
---
# 几何学
 六边形是任何 6 边的多边形。正六边形的所有边的长度都相同。我假设我们在这里使用的所有六边形都是规则的。
正六边形的大小可以用与边相接的内圆或与角相接的外圆来描述。在此页面上，我使用外圆的半径作为size. 宽度和高度根据两个圆的直径定义。
## 间距
接下来我们要把几个六边形拼在一起。

在里面平顶方向，相邻六边形中心之间的水平距离为horiz = 3/4 * width = 3/2 * size. 垂直距离为vert = height = sqrt(3) * size.

在里面尖顶方向，相邻六边形中心之间的水平距离为horiz = width = sqrt(3) * size. 垂直距离为vert == 3/4 * height == 3/2 * size.

一些游戏使用像素艺术来绘制与正多边形不匹配的六边形，这些公式必须稍作调整。有关详细信息，请参阅实施指南。

https://www.redblobgames.com/grids/hexagons/