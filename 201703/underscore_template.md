# undersocre 中的_.template函数

一直以来对前端模板引擎具体实现过程非常好奇，如knockout实现的原理，但是尝试去看knockout源码，发现自己的水平实在有限。  
柿子挑软的捏，了解了underscore的_.template函数之后，就想看一下它的源码，一窥究竟。

## 模板语法

1. <% %> ----  evaluate ：可以执行的javascript 语句
1. <%= %> ---- interpolate ：插入特定的值
1. <%- %> ---- escape ：插入html_escaped之后的值

>可以在调用template函数的时候传入settings参数自己设置模板分隔符(我没试过)


## 模板函数实现的原理

整体思路是将模板字符串经过正则解析，预留需要填入数据的位置，拼接成一个目标字符串，然后通过new Funciton()动态执行这个字符串，执行的过程将数据填入，然后返回填入数据的html字符串，就实现了想要的功能。


如下例：

``` js
  <script type="text/template" id="tp1">
        <ul>
            <% for(var i = 0 ; i < 10 ; i++ ){ %>
                <li><%= data[0].name %></li>
            <% } %>
        </ul>
 </script>
 <script>
        var tp = document.getElementById('tp1');
        var data = [{ name: "lll" }, { name: "orange" },];

        var compiled = _.template(document.getElementById("tp1").innerHTML);
        var html = compiled(data);
        // console.log(html)
    </script>

```

上面的模板字符串为tp.innerHTML

![tp.innerHTML](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/underscore_t_1.png);

经过解析得到的目标字符串为

![目标字符串](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/underscore_t_2.png);

然后将字符串作为传入参数传入 new Function(),最后返回该函数就行了。

### 具体的解析

```js
  text.replace(matcher, function (match, escape, interpolate, evaluate, offset) {
    .....
  } 
```

text是需要解析的字符串，matcher为一个RegExp对象，具体为 /<%-([\s\S]+?)%>|<%=([\s\S]+?)%>|<%([\s\S]+?)%>|$/g。  

> 本来对js中的正则一点都不懂，通过看这个源码，也算是稍微了解了一下正则的知识

全局匹配下的replace会多次调用其中的方法，每一次匹配都会调用，具体的escape为第一个括号匹配的字符串，interpolate为第二个，evaluate为第三个，offset为在源字符串的偏移量。

在匹配之前定义 index=0，source="__p+="，之后的每一次匹配都会执行下面的语句用来截取在模板分隔符之外的html字符串，如上例的"`<ul>`"(省略了空格)

```js
    source += text.slice(index, offset).replace(escapeRegExp, escapeChar); // 转义特殊的符号，让new Function()顺利执行。
    index = offset + match.length; 
```

之后在按照具体匹配的字符串进行处理：

- escape 不为undefined
```js
    source += "'+\n((__t=(" + escape + "))==null?'':_.escape(__t))+\n'";
    // 
```
- interpolate 不为undefined

```js
    source += "'+\n((__t=(" + interpolate + "))==null?'':__t)+\n'";
    //((__t=( data[0].name ))==null?'':__t)
```
- evaluate 不为undefined

```js
    source += "';\n" + evaluate + "\n__p+='";
    // ";\n"+for(var i = 0 ; i < 10 ; i++ ){ + "\n__p+="
```

最后的形成的source就是所说的目标字符串 ： 

```js
    var __t,__p='',__j=Array.prototype.join,print=function(){__p+=__j.call(arguments,'');};
    with(obj||{}){ // 赋值采用了with语句
    __p+='\n        <ul>\n            ';
    for(var i = 0 ; i < 10 ; i++ ){ 
    __p+='\n                <li>'+
    ((__t=( data[0].name ))==null?'':__t)+
    '</li>\n            ';
    } 
    __p+='\n        </ul>\n\n        '+
    ((__t=( data[2].name))==null?'':__t)+
    '\n    ';
    }
return __p; // 最后的html字符串
```

通过执行下面的语句得到渲染函数

```js
    render = new Function(settings.variable || 'obj', '_', source);
    var template = function (data) {
      return render.call(this, data, _);
    };
    return template;
```

到这里_.template函数的大致运行原理已经有了大致的了解。

> 参考文章 : 
[Underscore _.template 方法使用详解](https://github.com/hanzichi/underscore-analysis/issues/26?utm_source=tuicool&utm_medium=referral)







 




