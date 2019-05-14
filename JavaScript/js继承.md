# js继承
>作者：myprelude@github  
原文链接： https://github.com/myprelude/Web-FrameWork/blob/master/README.md  
转载请注明出处，保留原文链接和作者信息。

**继承是js的中比较重要的知识点，也是面试中的必考点，我们在日常开发中无时无刻都在和它打交道，我们今天就来聊一聊继承这个话题。**

#### 原型链继承
```
function People(name,age){
    this.name = name;
    this.age = age;
}
People.prototype.say = function(){
    console.log(`我的名字是${this.name},我的年龄是${this.age}`)
}
```

