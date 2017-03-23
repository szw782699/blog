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

![mustache_1.png](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/mustache_1.png)

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

这些类的关系是什么样子呢? 上面的例子执行的流程如下图所示:

![mustache_jiegou](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/mustache_jiegou.png)
 
由下面Writer类的render方法可以看出，mustache 主要是将模板字符串template 通过parse得到tokens数组  
然后renderTokens，其中context上下文中有要渲染的数据。

```js
 Writer.prototype.render = function (template, view, partials) {
    var tokens = this.parse(template);
    var context = (view instanceof Context) ? view : new Context(view);
    return this.renderTokens(tokens, context, partials, template);
  };
```

### tokens的获取

在Writer类的parse方法中，调用了工具函数parseTemplate(),parseTemplate感觉是核心之一了。   
其中的Scanner 用来扫描字符串，通过正则匹配到相应的tags。

```js
 function parseTemplate(template, tags) {
    tags = tags || mustache.tags;
    template = template || '';
    if (typeof tags === 'string') {
      tags = tags.split(spaceRe);
    }
    
    var tagRes = escapeTags(tags); 
    var scanner = new Scanner(template);

    var sections = [];     // Stack to hold section tokens
    var tokens = [];       // Buffer to hold the tokens
    var spaces = [];       // Indices of whitespace tokens on the current line
    var hasTag = false;    // Is there a {{tag}} on the current line?
    var nonSpace = false;  // Is there a non-space char on the current line?

    // Strips all whitespace tokens array for the current line
    // if there was a {{#tag}} on it and otherwise only space.
    function stripSpace() {
      if (hasTag && !nonSpace) {
        while (spaces.length) {
          delete tokens[spaces.pop()];
        }
      } else {
        spaces = [];
      }

      hasTag = false;
      nonSpace = false;
    }

    var start, type, value, chr, token, openSection;
    var loop = 0;
    while (!scanner.eos()) { // 当tail(剩余字符串)为空时，结束循环。
      start = scanner.pos;

      // Match any text between tags.
      // tagRes[0] =   new RegExp(escapeRegExp(tags[0]) + "\\s*"),
      // 通过查看scanUntil方法,可以得到value是匹配到{{ 前的字符串
      // 如上面例子中的"` <h1> hello  `"（忽略了前面的空格）
      value = scanner.scanUntil(tagRes[0]); 
      if (value) {
        for (var i = 0, len = value.length; i < len; ++i) {
          chr = value.charAt(i);

          if (isWhitespace(chr)) {
            spaces.push(tokens.length);
          } else {
            nonSpace = true;
          }
          // 将value中的每一个字符作为一个token 类型为text，并添加至tokens数组
          tokens.push(['text', chr, start, start + 1]);  
          start += 1;

          // Check for whitespace on the current line.
          if (chr === '\n') {
            stripSpace();
          }
        }
      }

      // Match the opening tag.
      // 如果tail的开头没有{{ 则说明tail为空，即扫描结束，可以结束循环了
      if (!scanner.scan(tagRes[0])) break;
      hasTag = true;

      // Get the tag type.
      // tagRe : /#|\^|\/|>|\{|&|=|!/
      // 得到token的类型,如{{data}}的类型就是name ，{{#data}}类型为#
      type = scanner.scan(tagRe) || 'name';
     
      // 当{{ #   list }} 时 ，可以去除#号后面的空格，如果注释这一行，会抛出一个异常“Unclosed tag at”
      // 至于为什么会抛出异常，以为遇到#号时，将token 添加进sections数组。遇到 / 时 将token取出，以value对比，不相等则抛出异常
      // 感觉自己表达能力好差
      scanner.scan(whiteRe); 

      // Get the tag value.
      if (type === '=') {
        value = scanner.scanUntil(eqRe);
        scanner.scan(eqRe);
        scanner.scanUntil(tagRes[1]);
      } else if (type === '{') {
        value = scanner.scanUntil(new RegExp('\\s*' + escapeRegExp('}' + tags[1])));
        scanner.scan(curlyRe);
        scanner.scanUntil(tagRes[1]);
        type = '&';
      } else {
        // tagRes[1]: new RegExp("\\s*" + escapeRegExp(tags[1]))
        // 获取tag的value ，如 {{msg.name.name}} value 就是msg.name.name
        value = scanner.scanUntil(tagRes[1]);
      }

      // Match the closing tag.
      // 如果没有 }} 标签  抛出异常 
      if (!scanner.scan(tagRes[1])) {
        throw new Error('Unclosed tag at ' + scanner.pos);
      }
      
      // 创建token 添加至tokens数组
      token = [ type, value, start, scanner.pos ];
      tokens.push(token);

      if (type === '#' || type === '^') {
        // 如果是区块的标记(#和^)时，将token入栈
        sections.push(token);
      } else if (type === '/') {
        // Check section nesting.
        // 遇到 / 出栈 
        openSection = sections.pop();

        if (!openSection) {
          throw new Error('Unopened section "' + value + '" at ' + start);
        }
        // 这里也就是上面 为什么要 scanner.scan(whiteRe); 的原因  
        // 因为 "   list" != "list"
        if (openSection[1] !== value) {
          throw new Error('Unclosed section "' + openSection[1] + '" at ' + start);
        }
      } else if (type === 'name' || type === '{' || type === '&') {
        nonSpace = true;
      } else if (type === '=') {
        // Set the tags for the next time around.
        tagRes = escapeTags(tags = value.split(spaceRe));
      }
    }

    // Make sure there are no open sections when we're done.
    // 循环结束，如果section栈里还有元素，肯定是标签没有闭合
    openSection = sections.pop();
    if (openSection) {
      throw new Error('Unclosed section "' + openSection[1] + '" at ' + scanner.pos);
    }

    return nestTokens(squashTokens(tokens)); //这里调用了squashTokens()和nestTokens() 对tokens进行处理
  }

```

关于squashTokens() 是将连续的类型为text的token合并到一起

![textToken合并](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/squash_token.png)

这样在渲染的时候减少了循环的次数吧

关于nestTokens() 我觉得这个方法真的很棒，处理标签的嵌套
如 {{#list}}{{name}} --> {{info}}{{/list}}  
将{{#list}}{{/list}}内的token合并到一个数组内，并且添加到[#,list,...]这个token中
应该算是形成了一种树形的结构

![textToken 压缩](https://raw.githubusercontent.com/szw782699/blog/master/assets/201703/nest_token.png)

### renderTokens 

tokens已经生成，接下类就是渲染。这个方法里面用到了Context对象，我觉得主要是用来查找值。

在render方法中，对每个token进行判断，  
如果是类型是name的话直接添加到context.lookup(value)，然后添加到字符串buffer上，就实现了将  
{{msg.name.name}} 替换成相应的值。  
如果是text类型的token也是直接添加到buffer上  
对于类型为#的字符串，处理起来比较麻烦一点，

```js
  case '#':
          value = context.lookup(token[1]);
          if(!value) continue;

          if(isArray(value)){ // 递归渲染实现了循环的效果，这里明白了为什么要nestTokens();
            for(var j = 0, jlen = value.length; j < jlen; ++j){
              buffer += this.renderTokens(token[4],context.push(value[j]),partials,originalTemplate);
            }
          }else if(typeof value === 'object' || typeof value === 'string'){ // 如果不是数组的情况
            buffer += this.renderTokens(token[4],context.push(value),partials,originalTemplate);
          }else if(isFunction(value)){ // 为什么不提一下这个（因为我也不懂呀0.0）
            if(typeof originalTemplate !== 'string'){
              throw new Error('Cannot use higher-order sections without the original template');
            }

            value = value.call(context.view, originalTempalate.slice(token[3],token[5]),subRender);
            if(value != null) buffer += value;
          }else {
            buffer += this.renderTokens(token[4],context,partials,originalTemplate);
          }
          break;
```


## 总结

想比于underscore的_.templates函数是直接拼接字符串通过new Function() 来赋值。  
mustache 根据自己的语法规则进行解析生成tokens，然后tokens + data --> html。   
Context类里面几乎没介绍，context主要是访问数据用cache缓村提高性能。  
看了两天，感觉稍微有了那么点意思，记录下来，用以备忘。










