---
title: 重学css
urlname: wzu8s0
date: '2022-06-10 13:58:35 +0800'
tags: []
categories: []
---

## 文档流

网页是一个多层的结构，一层摞一层，通过 css 可以分别为每一层设置样式，用户只能看到的是从最顶层到最底层的展示效果。最底层我们称之为文档流，文档流是网页的基础，默认所创建的元素都是在文档流中排列，当一个元素不在文档流中，就叫做脱离文档流

## 盒模型

css 将页面中所有的元素都设置成一个矩形的盒子
盒子 = 内容 content + 内边距 padding + 边框 border + 外边距 margin
默认情况下设置 width，height 指的是内容区域。
box-sizing 可设置盒子模型。

- content-box：默认值，width 和 height 等于元素的内容区域。
- border-box：width 和 height 等于元素的内容区域+内边距+边框
- inherit：规定应从父元素继承 box-sizing 属性的值。

## 内边距，外边距

## 水平方向布局

元素在水平方向的位置受以下几个属性的影响，并完全遵循一个公式。
margin-left + border-left + padding-left + width + padding-right + border-right + margin-right = 父元素内容的宽度
比如以下例子:
可以推算出：0 + 0 + 0 + 200 + 0 + 0 + 0 = 600
这种不相等的情况，叫做过度约束，出现这种情况，浏览器会自动调整

1. 如果有属性设置为 auto，则先调整设置为 auto 的属性(width/margin-left/margin-right)
1. 如果没有 auto 的属性，则调整 margin-right 使等式成立
1. 如果 width，margin 都设为 auto，则优先调整 width
1. 如果两个外边距都是 auto，则两个边距等分需要调整的宽度

调整后的 margin-right 为 400px；公式为：0 + 0 + 0 + 200 + 0 + 0 + 400 = 600
![image.png](https://cdn.nlark.com/yuque/0/2022/png/462392/1645616307972-c25d7753-0ebd-42ad-8c41-ebbc8598c1a2.png#clientId=ue2df96f4-097a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=778&id=ua15041c8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=778&originWidth=1940&originalType=binary∶=1&rotation=0&showTitle=false&size=322541&status=done&style=shadow&taskId=u384f05d3-1782-47bd-b53f-44836e97870&title=&width=1940)
如果其中有值为 auto，那么会优先调整为 auto 的属性，width 默认值为 auto，margin 默认为 0
如下，调整后的 width 为 600px，
![image.png](https://cdn.nlark.com/yuque/0/2022/png/462392/1645617066782-f4f063f6-9588-4dfb-9982-8732c8d158fd.png#clientId=ue2df96f4-097a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=786&id=u9c98a730&margin=%5Bobject%20Object%5D&name=image.png&originHeight=786&originWidth=1930&originalType=binary∶=1&rotation=0&showTitle=false&size=320773&status=done&style=shadow&taskId=ud1340020-dc77-4ef8-9541-7e75f0024de&title=&width=1930)

## 垂直方向布局

在垂直方向上，两个相邻的外边距会发生重叠现象
规则如下：
兄弟元素：

- 两者之间的距离会取两者的外边距中较大的那个值
- 如果两个外边距一正一负，则间距取值为两个外边距的和
- 如果两个外边距都是负值，则间距取值为较小的那个

父子元素：

- 子元素的外边距会传递给父元素(上外边距)
- 解决办法是可以加一个边框，把两个外边距隔离开

## 行内元素

行内元素不支持设置 width，height。大小由内容决定
行内元素可以设置 padding，margin，border，但是在垂直方向上，不会影响布局，会覆盖在其他元素上面。

## 背景

background-image: url('')
**background-repeat** 用来设置背景的重复方式

- repeat 会在 x，y 轴方向重复显示图片，默认
- repeat-x
- repeat-y
- no-repeat 北极图片不重复，只显示一个

**background-position** 设置图片在元素中的位置，一般设置两个值，默认 center。

- top/bottom/left/right/center
- background-position 也设置像素值，表示图片在元素中的偏移，类似于定位，需要设置 x，y 轴两个值，可为负值

**background-clip** 背景裁剪，设置背景在元素中展示的范围，

- border-box 背景会展示在边框，padding，content 区域内
- padding-box 背景会展示在 padding，content 区域内
- content-box 背景会展示在 content 区域内

**background-origin 背景图片的偏移量 background-position 的计算原点**

- border-box 背景图片的偏移量从边框开始算起
- padding-box 默认值，背景图片的偏移量从内边距开始算起
- content-box 背景图片的偏移量从内容区开始算起

**background-size 设置背景图片的大小**

- 第一个值是图片宽度，第二个是高度
- 如果只写一个值，则第二个值默认是 auto
- cover 图片比例不变，图片将整个元素占满
- contain 图片比例不变，将图片在元素中完全展示

background-attachment 设置背景图片是否跟随元素滚动而滚动
有两可选值：fixed，scroll（默认）
注意：background 是简写形式，属性值没有顺序要求，单 position 必须在 size 前，origin 必须在 clip 前

## 零碎

#### display 的属性值：

- inline
- block
- inline-block 行内快，可以设置宽高，但是不会占一整行
- table
- none

display：none 隐藏元素，不渲染，不占位。visibility：hidden 隐藏元素，占位

#### box-sizing 用来设置元素的尺寸计算方式，也就是设置的 width，height 表示的区域范围。

它的属性：

- content-box 默认，width，height 表示内容区域的尺寸
- border-box 设置 width，height 表示从边框到边框的尺寸 width = content + padding + border

#### 浮动

浮动可以设置字体环绕图片的效果，设置浮动的元素会脱离文档流
脱离文档流的元素有一下特点：
块元素： 1.块元素不在独占一行 2.如果没有设置宽高，块元素默认被内容撑开
行内元素：
行内元素脱离文档流以后，会变成块元素，特点和块元素一样

```css
{
  'aaaa': 'aaaaa'
}
```
