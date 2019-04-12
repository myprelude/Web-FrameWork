# UED说这个是1px
>作者：myprelude@github  
原文链接： https://github.com/myprelude/Web-FrameWork/blob/master/README.md  
转载请注明出处，保留原文链接和作者信息。


UED: 这个怎么这线怎么这么粗，这个框怎么这么粗。。。，我要的是1px；

前端: 我就是1px！

UED: 你看人家的1px，你这个1px不是我需要的1px。

前端: 。。。。

### 为啥会有1px的问题

### 解决方案
```
<div style='height:100px,width:100px' class='px'></div>
```
* background + linear-gradient
```
.px{
    background:

    linear-gradient(180deg, black, black 50%, transparent 50%) top left / 100% 1px no-repeat,

    linear-gradient(90deg, black, black 50%, transparent 50%) top right / 1px 100% no-repeat,

    linear-gradient(0, black, black 50%, transparent 50%) bottom right / 100% 1px no-repeat,

    linear-gradient(-90deg, black, black 50%, transparent 50%) bottom left / 1px 100% no-repeat;
}
```
原理：通过50%透明的渐变将1px变成一半透明就产生了变细的错觉。
缺点：代码量太大，不易记住，如果实现边框有倒圆角就没有办法实现了。

* CSS3 box-shadow
```
.px{
    -webkit-box-shadow:0 1px 1px -1px rgba(0, 0, 0, 0.5);
    box-shadow:0 1px 1px -1px rgba(0, 0, 0, 0.5);
}
```
优点：代码量少，开发方便，可以实现倒圆角。

* background-image
```
.px{
    background: url(../img/line.png) repeat-x left bottom;
    background-size: 100% 1px;
}

```
缺点：多引入图片，修改颜色时需更换图片麻烦。

* transform + :after
```
.px{
  position: relative;
}
.px:after{  // 单边框
  content: '';
  position: absolute;
  bottom: 0;
  background: #000;
  width: 100%;
  height: 1px;
  transform: scaleY(0.5);
  transform-origin: 0 0;
}
.px-all:after{  // 全边框
  content: '';
  position: absolute;
  width:200%;
  height:200%;
  border:1px solid #eee;
  border-radius:inherit; // 继承父元素倒圆角
  transform: scale(0.5);
  transform-origin: 0 0;
}
```
优点：便于控制，好理解，修改样式倒圆角，一个类就搞定。
**在项目中使用transform + :after会更加方便，建议大家都可以试试。**

* viewport + rem 实现
这个大佬有很好的文章：[使用Flexible实现手淘H5页面的终端适配](https://github.com/amfe/article/issues/17)