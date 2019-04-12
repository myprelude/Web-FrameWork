>作者：myprelude@github  
原文链接： https://github.com/myprelude/Web-FrameWork/blob/master/README.md  
转载请注明出处，保留原文链接和作者信息。
# JavaScript中的面向对象
网上一直流行着--”js中一切都是对象“。这句话其实不能算完全对。Es中明确有String Number Boolean null undefined基本类型（Es6新增了Symbol）;Object只是一种复杂的类型。但是从对象定义来说--对象是一个包含键值对的容器，它拥有自己对应的属性和方法。好像又觉得这句话是对的。

### 创建一个Object
我们一般通过三种方式来创建对象；如下：
```
let obj = {}  //  通过字面量方式
let obj = new Object() //  new实例化
let obj = Object.create() // es6
```

一般我们可以通过`typeof`简单的判断js的类型.如下：
```
typeof 123  //  'number'
typeof '123'  //  'string'
typeof undefined  // 'undefined'
typeof null  //  'object'
typeof {} // 'object'
typeof [] // 'object'
typeof /\s/  //  'object'
typeof function(){}  //  'function'
```
在`typeof`角度来看 null , {} , [] , reg …… 都是对象；