# css 盒模型，浮动和定位

## 盒模型

每个元素表示成一个矩形盒子， 这个盒子由内容，内边距(padding)，边框(border)，和外边距(margin)组成。

![box](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/box.png)

在标准的盒子模型中，width和height指的是内容的宽和高，增加内边距，边框和外边距不会影响内容的尺寸，但是会增加整个矩形盒子的总尺寸。如上图 矩形盒子总大小为380*380  

在早期的IE中，width属性不是内容的宽度，而是指内容，内边距和边框宽度的总和。就是添加的内边距越多，给内容留下的区域越小

可以通过box-sizing 属性定义使用哪种盒子模型

 1. content-box : 标准盒子模型 widht和height只包括内容的宽和高
 1. border-box : width  = border + padding + 内容的宽度
 3. padding-box : width = border + padding ;  只有Firefox实现了这个值，它在Firefox50中被删除

## 定位

position 的值可以取以下几种

 1. static(默认) 没有定位，正常显示
 1. relative(相对定位) : 相对自己原来的位置偏移
 1. absolute(绝对定位) : 相对距离自己最近的已定位的祖先元素位置偏移
 1. fixed(固定) : 相对视窗的位置进行定位

### 可视化格式模型

p、h1、div等元素被称为块级元素，这些元素显示为一块内容。strong和span等元素被称为行内元素，内容显示在行中。

css中有3中定位机制： 普通流、浮动和绝对定位。  

在普通流中，块级元素一个接一个垂直排列。行内元素在一行中水平排列

### 相对定位(position:relative)

相对定位是相对在文档流中原来的位置进行偏移，使用相对定位时，无论是否移动，元素仍然占据原来的空间，移动会导致它覆盖其他元素。  
所以相对定位可以看做是普通流定位的一种

![relative](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/relative.png);

### 绝对定位(position:absolute)

绝对定位使元素的位置与文档流无关，不占据空间。普通文档流中其他元素的布局和绝对定位元素不存在时一样(如下图:相对父元素的边框，和padding无关)

![absolute](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/absolute.png);

### 固定定位(position:fixed)
固定定位时绝对定位的一种，差异在于固定定位相对于视窗(viewport)而定位

![fixed](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/fixed.png)

> (position:fixed;left:200px;top:200px;)


## 浮动


