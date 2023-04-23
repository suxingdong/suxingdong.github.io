---
title: Unity 抗锯齿
author: sille_bille
date: 2021-01-03 18:32:00 -0500
categories: [Blogging, Tutorial]
tags: [google analytics, pageviews]
---

# 抗锯齿2 (Anti-aliasing)
 抗锯齿效果使图形更加平滑。锯齿是线条出现锯齿状或“楼梯”外观的效果（如下面左图所示）。如果图形输出设备没有足够高的分辨率来显示直线，则会发生这种情况。

抗锯齿用中间色调将这些锯齿状线条包围起来，从而可以降低锯齿明显程度。虽然这种做法减轻了线条的锯齿状外观，但也会使它们变得更模糊。

抗锯齿算法根据图像而定。此算法在传统多重采样算法不受支持时很有用，例如延迟渲染着色路径，或 Unity 5.5 或更早版本前向渲染路径中的 HDR。这些选项位于 Editor 的 Quality settings 窗口中。

后期处理 (Post Processing) 包中提供的算法包括：

快速近似抗锯齿 (Fast Approximate Anti-aliasing, FXAA)：一种适用于移动端以及不支持运动矢量的平台的快速算法。
亚像素形态抗锯齿 (Subpixel Morphological Anti-aliasing, SMAA)：一种适用于移动端以及不支持运动矢量的平台的高质量但慢速算法。
时间抗锯齿 (Temporal Anti-aliasing, TAA)：一种需要运动矢量的先进技术。适用于桌面平台和游戏主机平台。
如需了解 Unity 的抗锯齿偏好设置的详细信息，请参阅后期处理 (Post-Processing) 包中的抗锯齿 (Anti-aliasing) 文档。
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=475479888&auto=1&height=66"></iframe>
