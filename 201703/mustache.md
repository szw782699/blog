# mustache 源码学习

mustache 是一个轻量级的前端模板引擎，官方说法是logic-less template。 它的语法非常简单，使用起来也比较方便，既可以在前端也可以在后端使用。 
 

## 语法简介

- {{data}}  直接输出值
- {{#data}}{{/data}} 表示一个section，根据上下文中的键值对区块进行渲染，可以用来循环
- {{^data}}{{/data}} 当data的值为false，undefined，null 或data是一个长度为0的数组时，才会展示区块内的内容
- {{>data}}{{/data}} 表示一个子模块
- {{{}}} 和{{&}} 也是输出，但是会将特殊的字符转义
- {{!}} 用来表示注释
- {{.}} 可以用来循环数组,数组中的元素为对象的时候，还是用上面的 # 循环遍历比较好

上一个简单的例子

```js
   <script type="text/template" id="tp">
        <h1> hello  {{msg.name.name}}</h1>
        <ul>
          {{ # list}}<p>{{name}} --> {{info}}</p>{{/list}}
        </ul>
    </script>
    <script>
        var tp = document.getElementById('tp');
        var data = {
            'msg':{
                'time' : 'today',
                'name' : {'name' : 'SZW'},
            },
            'list':[
                {name : 'kk', info : '22'},
                {name : 'dj', info : '23'},
            ]     
        };
        var html = Mustache.render(tp.innerHTML , data);
        console.info(html);
```

打印的结果如下

![mustache_1.png](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/mustache_1.png);

## 大致浏览源码

mustache.js只有接近600行的样子，这也是我选择去看它的原因，毕竟水平实在是有限，太复杂了我搞不懂。 
虽然代码行数不多，但是作为小菜的我看起来也是比较吃力，没办法，慢慢学习吧。   

### 在浏览器端使用时的Mustache 是怎么来的

```js 
    (function (root, factory) {
        if (typeof exports === "object" && exports) {
        factory(exports); // CommonJS
    } else {
        var mustache = {};
        factory(mustache);
        if (typeof define === "function" && define.amd) {
        define(mustache); // AMD
    } else {
        root.Mustache = mustache; // <script>
    }
  }
}(this, function (mustache) {
    ......
}));

```
上面的代码是一个立即执行函数，为了支持Commonjs规范和AMD规范，在浏览器中参入root值为winodw对象，就得到了全局变量Mustache,
感觉写的好神奇。

###  大致分类

* 前面的一些工具函数
* Scanner类
* Writer类
* Context类

这些类的关系是什么样子的呢

![mustache_jiegou](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/mustache_jiegou.png);








