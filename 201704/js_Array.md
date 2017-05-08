# Array

javaScript 中的Array是一个神奇的存在，不像其他的语言那样仅仅是一个有序列表，它的每一项都可以保存不同的数据。Array本身也是一个对象，同样可以有自己的属性。

## 创建一个数组

* 使用构造函数，如下，表示创建一个长度为29的数组。

```
var arr = new Array(29);

```

* 直接数组字面量

```
var arr = [1,2];
```

## Array的属性

* length : 自然是数组的长度，随着数组元素的增加/减少会动态改变

## 方法

* Array.from

可以从一个类数组或者可迭代对象创建一个新的数组实例,例如set，map，和函数的参数arguments

```js
var s = new Set(["foo","bar"]);

var arr = Array.from(s);
arr; // ["foo",""bar]

var m = new Map([['name','张三'],['age',18]]);

arr = Array.from(m);
arr; // [['name','张三'],['age',18]]


function example(){
    var arr = Array.from(arguments);
    return arr;
}

example(1,2,3,4,5); // [1,2,3,4,5]


```

* Array.isArray()

该方法用于判断参数是否是一个数组

* Array.of()

将传递的参数，按顺序作为数组的元素，返回这个数组,与new Array的不同之处在于 单个参数并且为整数时

```js

var arr = Array.of(1,3,'jj',{'name':'szw'});
arr;//[1, 3, "jj", Object]


arr = new Array(3); // [undefined x 3]

arr = Array.of(3); // [3]

```

* 原型上的方法(Array.prototype)

  * concat : 用于合并多个数组，不改变原数组，返回的是一个新数组

  ```js
  var a1 = [1,2],a2 = [3,4]; 
  var a3 = a1.concat(a2); // a3: [1,2,3,4] 
  ```

  * pop :  从数组中删除最后一个元素，并返回该元素，数组的长度减 1


  * push : 将一个或者多个元素添加到数组的末尾，返回新数组的长度

  * shift : 删除第一元素，返回该元素，数组长度减1

  * unshift : 从数组头部添加一个或多个元素，返回新数组的长度

  * every(callback[,thisArg]) : 测试所有元素是否都通过了callback函数的测试  
    callback的参数有三个 : 元素值，元素索引，原数组。  
    返回true/false, 只有所有元素都符合要求才返回true。

  ```js
  var arr = [1,2,3,"222"];

  arr.every((item)=>{
      return typeof item === 'number'; 
  }); // false

  ```

  * some(callback[,thisArg]) : 测试某些元素是否通过了测试，就是说只有一个元素符合条件就会返回true,  
  同样callback 的参数有三个，元素，索引和原数组。

  ```js
  var arr = ["1","22",3];
  arr.some((item)=>{
      return typeof item === 'number';
  }) // true;

  ```

  * filter(callback[,thisArg]) : 返回通过过滤的元素组成的新数组。
  同样callback三个参数，和上面相同

  ```js
  var arr = [1,2,3 ,6,7,8];

  var newarr = arr.filter((item)=>{
      return item >= 5;
  });

  newarr; // [6,7,8]
  ```

  * find(callback[,thisArg]) :返回满足条件的第一个元素，没有的话就返回undefined

  ```js

  var arr = [1,2,3,4,5];

  arr.find((item)=>{
      return item >= 3;
  }); // 3

  ```

  * findIndex : 返回满足条件的第一个元素的索引，否则返回  -1;  callback的参数同上。

     ```js

        var arr = [1,2,3 ,4,5];
        arr.findIndex((item)=>{
            return item >= 3
    }); // 2

     ```

  * indexOf : 返回在数组中给定元素的第一个索引，没有的话返回-1。

  ```js
    var arr = [1,2,3,4,5];
    arr.indexOf(3); // 2 

  ```

  * lastIndexOf : 返回给定元素在数组中的最后一个索引，没有返回-1

  ```js
    var arr = [1,2,3,3,4,5];
    arr.lastIndexOf(3); // 3

  ```

  * map(callback[,thisArg]) : 返回一个新数组，数组里的每一个元素是原数组的一个映射


  ```js
  var arr = [1,2,3];

  var newarr = arr.map((item)=>{
      return item * 3;
  });
  newarr; // [3,6,9];
  ```

  * reduce(callback,initialValue) : 这个函数感觉有点复杂，是对累加器和数组的每一个元素应用一个函数，最后返回一个值。  
  第二个参数是累加器的初始值，没有指定的话reduce会把索引0的值作为初始值，然后从索引1开始执行回调函数。

  ```js
    // 不指定initialValue
    var arr = [1,2,3,4];
    arr.reduce((acc,item)=>{
        console.log('acc : ' + acc + '   item : ' + item );
        return acc + item;
    }); 
    //acc : 1   item : 2
    //acc : 3   item : 3
    //acc : 6   item : 4
    // 10

    //指定
    arr.reduce((acc,item)=>{
        console.log('acc : ' + acc + '   item : ' + item);
        return acc + item;
    },10);
    /*
     acc : 10   item : 1
     acc : 11   item : 2
     acc : 13   item : 3
     acc : 16   item : 4
     20
    */
  ```

  * reduceRight: 和reduce的执行方向相反。

  ```js
   var arr = [1,2,3,4];
   arr.reduceRight((acc,item)=>{
        console.log('acc : ' + acc + '   item : ' + item );
        return acc + item;
   }); 
   /*
   acc : 4   item : 3
   acc : 7   item : 2
   acc : 9   item : 1
   10
  */
  ```

  * reverse : 改变了原数组，颠倒了原数组的顺序。

  ```js
    var arr = [2,3,4];
    arr.reverse();
    arr; // [4,3,2]

  ```

  * slice(begin,end) : 返回一个从begin到end的新数组   
    如果省略begin，则slice从索引0开始  
    如果省略end，则会提取到数组的末尾  

  ```js
    var arr = [1,2,3,4];
    var newarr = arr.slice(1,3);
    newarr; // [2,3];
  
  ```


  * splice(start,[deleteCount],[item1,item2...]) : 这个方法感觉也有点复杂，通过删除现有元素/或添加新元素类更改数组的内容,返回的是由删除的元素组成的一个数组。  
  start: 表示从哪个位置开始删  
  deleteCount: 表示删除几个元素，包括start

  ```js
    var arr = [1,2,3,4,5];
    var newarr = arr.splice(1,3,4,4,4,4,4);
    arr;//[1, 4, 4, 4, 4, 4, 5]
    newarr; //[2,3,4]

  ```

  * join : 将数组(类数组)的所有元素连接到一个字符串中
    参数为指定的分隔符，不指定的话默认是逗号。


  * toString : 返回的是arr.join()之后的字符串

  * keys : 方法返回一个新的Array迭代器

  * entries :返回一个包含键值对的迭代器

  * copyWithin(target,start,end) :方法浅复制数组的一部分到同一个数组的另一个位置。并返回它，而不修改其大小。  
  target : 复制部分的目标插入位置。  
  start  : 复制部分的开始位置
  end : 复制部分的结束位置，不包括end这个元素。  
  对原数组进行了修改
  ```js

  var arr = ['first','sencond','third','fourth','fifth'];
  arr.copyWithin(1,2,4);
  arr; //["first", "third", "fourth", "fourth", "fifth"] **** 这里会形成覆盖，把target后面的元素会覆盖掉。


  ```

  这样是不是可以利用这个方法在原数组上删除指定位置的元素

  ```js
  var arr = [1,2,3,4];
  function remove(arr,index){
      arr.copyWithin(index,index + 1);
      arr.pop();
      console.log(arr); // [2,3,4]
  } 
  remove(arr,0); // 删除索引为0的元素
  ```

  * sort(compareFunciton) : 根据某种顺序对数组进行排序，不是稳定的。返回的是对排序后原数组的引用

  ```js
  var arr = [3,1,4,5,2];
  // 降序
  arr.sort((a,b)=>{
      return b - a;
  })

  //升序
  arr.sort((a,b)=>{
      return a -b;
  });
  

  ```


## 怎样检测是不是数组

* instanceof 

* Array.isArray();

* arr.constructor === Array

* Object.prototype.toString.call(arr) === "[object Array]"; 

















