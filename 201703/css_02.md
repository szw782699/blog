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
固定定位是绝对定位的一种，差异在于固定定位相对于视窗(viewport)而定位

![fixed](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/fixed.png)

> (position:fixed;left:200px;top:200px;)


## 浮动

任何元素都可以设置为float，块级元素div等可以，span/strong这样的行内元素也可以。任何被设为float的元素被自动块级化，可以有width和height属性  

当一个元素被设置为float，，它就脱离正常的文档流，向左或向右移动，直到碰到包含框的边界或者另一个浮动元素的边界

- 不浮动

![not_float](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/not_float.png);

- 将1设为float: right

![float1](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/float1.png);

- 将1设为float: left

![float11](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/float11.png);

将1 float left 后出现了 2 和3 重合的现象， 按正常的情况来说，应该是元素块2 隐藏在元素块1的下面(实际上检查元素也是如此)。

那为什么2这个字母会出现在元素块3的位置？ 

**浮动的元素脱离文档流，不再影响不浮动的元素。但是实际上，如果浮动的元素后面有一个文档流中的元素(如上图中的浮动元素1后面有元素块2)，那么这个元素(如2)的框表现的像浮动元素不存在一样，但是框内的文本内容会受到浮动元素的影响，会移动以留出空间。(如元素块2中的文字) 实际上浮动的本职工作应该是文字环绕显示**

- 将1,2,3都设置为float
  
  1. 父元素的宽度能容纳三个总宽度

  ![float_all_1](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/float_all_1.png);

  2. 父元素宽度不能容纳三个元素

  ![float_all_2](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/float_all_2.png);



*** 

##　浮动带来的问题

浮动会造成高度塌陷: 只包含浮动元素的父元素如果没有指定高度，那么父元素的高度就始终是0，
如果想为父元素添加背景就比较尴尬了。如下图(父元素设置了 padding:30px;margin:30px;):

![高度塌陷](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/height_taxian.png);


### 清除浮动

#### clear: both

![clear](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/clear_both_code.png);![clear](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/clear_both_code1.png);

![clear](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/clear_both.png);

实际上为什么最后一个div属性设置为clear:both 能达到效果呢？
因为浏览器在元素顶上添加足够的外边距，使元素顶边缘垂直下降到浮动元素的下面

#### overflow:auto|hidden

父元素添加属性 overflow :auto 

**overflow有三个值可以取 visible 不能清除浮动**

这种方法在某些情况下会产生情况下会产生滚动条或截断内容

![overflow_auto](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/overflow_auto.png);

上图将1的文字增多,overflow:auto 产生滚动条，若为hidden则截断内容。

#### :after伪类

父元素设置

```css
xxx:after{
    content:".";
    display:block;/**必须的**/
    visibility: hidden;
    clear:both;   
}
```

#### 使父容器形成BFC

##### 什么是BFC : Blocking Formatting Context(块级格式化上下文)。  
首先Formatting Context是指页面中的一块渲染区域，有自己的渲染规则，它决定了子元素如何定位，以及和其他元素的相互作用。  
BFC :是一个独立的的渲染区域，只有block-level的元素参与，规定了内部的block-level box 如何布局，并且这个区域与外部毫不相干。

##### 怎样触发BFC

1. float的值不为null
1. overflow的值不为‘visible’
1. display的值为“table-cell”，“table-caption”或者“inline-block”中的任何一个
1. position的值不为static或relative中的任一个

##### BFC 的布局规则

* 内部的box在垂直方向，一个接一个放置
* box垂直方向的距离由margin决定，属于同一个BFC的两个相邻BOX的margin会发生重叠
* 每个元素margin的左边，与包含块border的左边相接触，即使存在浮动也是如此
* BFC 的区域不会与相邻float box重叠(可以实现自适应两栏布局)
* 计算BFC的高度时，浮动元素也参与计算（所以可以通过bfc来清除浮动）


















