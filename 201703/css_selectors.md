# css 中选择器的权值计算

学习css的笔记，记录下css的种类以及权值的计算

## css中选择器中的种类

1. 元素选择器(type selectors)
1. id选择器
1. 类选择器(class selectors)
1. 属性选择器
1. 伪类
1. 伪元素
1. 通配符
1. 连接符(Combinators)

如果外部样式，内部样式和内联样式同时作用于同一个元素，优先级如下:

- 外部样式 < 内部样式 <内联样式

选择器的优先权

- !important > 内联样式 > id选择器 > class选择器 > 元素选择器  

## 选择器的权值计算如下

1. 数出里面id选择器的数目 记为a
1. 数出其中类选择器,属性选择器以及伪类的数目 记为b
1. 数出元素选择器和伪元素的数目 记为c
1. 忽略通配符

连接这三个数字abc，即得出选择器的权值，示例如下: 

> \* :   
a = 0 b = 0 c = 0 -> specificity = 0  
 li :   
a = 0 b = 0 c = 1 -> specificity = 1  
 ul ol+li :   
a = 0 b = 0 c = 3 -> specificity = 3  
 \#x34y :   
a = 1 b = 0 c = 0 -> specificity = 100  
 \#s12:not(FOO) :   
a = 1 b = 0 c = 1 -> specificity = 101  

如果两个规则的权值相同，那么后定义的规则优先。  


> 参考 [https://www.w3.org/TR/css3-selectors/#specificity]() 
