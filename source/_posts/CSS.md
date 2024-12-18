---
title: CSS
date: 2024-11-14 08:53:44
tags:
---
# CSS
## 语法
- 选择器 声明
- @media
## 布局 

研究屏幕平面如何分布

### 文档流

默认布局方式

### 脱离文档流

浮动、绝对定位、相对定位

### float布局(IE专用)

父元素 .clearfix 子元素 float:left

### flex布局(一维)

父元素 display:flex

#### 1.`flex`布局概念

容器默认存在两根轴：水平向右的**主轴**和垂直向下的**交叉轴**

- 默认为水平向右不换行

#### 2.容器属性

| **属性名**      | 说明                                        |
| --------------- | ------------------------------------------- |
| flex-direction  | 决定主轴的方向                              |
| flex-wrap       | 是否换行&换行方向                           |
| flex-flow       | flex-direction属性和flex-wrap属性的简写形式 |
| justify-content | 主轴上的对齐方式                            |
| align-items     | 交叉轴上的对齐方式                          |
| align-content   | 多根轴线的对齐方式                          |

1. `flex-direction` 主轴的方向

决定主轴的方向，即容器内元素排列方向

| 值             | 描述                     |
| -------------- | ------------------------ |
| row            | **`默认值`**主轴水平向右 |
| row-reverse    | 主轴水平向               |
| column         | 主轴垂直向下             |
| column-reverse | 主轴垂直向上             |

2.`flex-wrap` 换行&方向

| 值           | 描述                                 |
| ------------ | ------------------------------------ |
| nowrap       | **`默认值`**不换行                   |
| wrap         | 正向换行 (主轴水平向下 主轴垂直向右) |
| wrap-reverse | 反向换行 (主轴水平向上 主轴垂直向左) |

3.`flex-flow` 上述两种属性简写

flex-flow: [flex-direction] [flex-wrap]

4.`justify-content` 主轴上的对齐方式

| 值            | 描述                             |
| ------------- | -------------------------------- |
| flex-start    | **`默认值`** 起点对齐            |
| flex-end      | 终点对齐                         |
| center        | 居中                             |
| space-between | 两端对齐，子元素之间的间隔都相等 |
| space-around  | 间隔对齐，子元素两侧的间隔相等   |

5.`align-items` 交叉轴的对齐方式

| 值         | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| stretch    | **`默认值`** 如果项目未设置高度或设为auto，将占满整个容器的高度 |
| flex-start | 起点对齐                                                     |
| flex-end   | 终点对齐                                                     |
| center     | 居中                                                         |
| baseline   | *项目的第一行文字的基线对齐(不常用)*                         |

6.`align-content`多根轴线的对齐方式

| 值            | 描述                             |
| ------------- | -------------------------------- |
| stretch       | **`默认值`** 轴线占满整个交叉轴  |
| flex-start    | 起点对齐                         |
| flex-end      | 终点对齐                         |
| center        | 居中                             |
| space-between | 两端对齐，子元素之间的间隔都相等 |
| space-around  | 间隔对齐，子元素两侧的间隔相等   |



#### 3.子元素属性

| 属性        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| order       | 排列顺序, 数值越小，排列越靠前，默认为0                      |
| flex-grow   | 放大比例，默认为0                                            |
| flex-shrink | 缩小比例,默认为1,空间不足，该项目将缩小                      |
| flex-basis  | 分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。 |
| flex        | flex-grow, flex-shrink 和 flex-basis的简写,后两个属性可选。  |
| align-self  | 单个项目有与其他项目不一样的对齐方式，可覆盖align-items属性  |



### grid布局(二维)

父元素 display:grid

## 定位
- 研究垂直于屏幕如何分布
    1. position:static 默认值
    2. position:relative 相对定位
    3. position:absolute 绝对定位
    4. position:fixed 固定定位
## 动画