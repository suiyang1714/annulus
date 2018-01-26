## CSS3 实现圆环

刚开始写这个圆环的时候是参照帖子上给出的css代码代入，然后根据自己的需求改，发现圆环可以完美转动了，但是好像没法用百分比控制，所以放弃了随便copy一个现成的想法，该动动脑子还是有必要的。

### 实现原理

css实现圆环的方法很多，我看大部分都是采用边框（border）+ 裁剪（clip：rect（））来实现的，所以我也准备采用这种方式。

主要是布局问题，我看大部分圆环进度条都大同小异，就是布局和转动方式不同


```
// html

<div id="loading" class="loading">
    <div class="circle">
        <div class="percent left"></div>
        <div class="percent right wth0"></div>
    </div>
    <div class="per"></div>
</div>

// css

.loading {
  position: fixed;
  top: 50%;
  left: 50%;
  overflow: hidden;
  z-index: 9999;
  -webkit-transform: translate(-50%,-50%);
          transform: translate(-50%,-50%);
}
.circle{
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
  border:10px solid #fff;
  clip:rect(0,100px,100px,50px);
}
.clip-auto{
  clip:rect(auto, auto, auto, auto);
}
.percent{
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
  top:-10px;
  left:-10px;
}
.left{
  -webkit-transition:-webkit-transform ease;
  transition:-webkit-transform ease;
  transition:transform ease;
  border:10px solid #E54B00;
  clip: rect(0,50px,100px,0);
}
.right{
  border:10px solid #E54B00;
  clip: rect(0,100px,100px,50px);
}
.wth0{
  width:0;
}

// js


$('.left').animate({
    deg: per*3.6
}, {
    step: function(n, fx) {
        if(per>180){
            $('.circle').addClass('clip-auto');
            $('.right').removeClass('wth0');
        }
        $(this).css({
            "-webkit-transform":"rotate("+n+"deg)",
            "-moz-transform":"rotate("+n+"deg)",
            "transform":"rotate("+n+"deg)"
        });
    },
    duration:500
})

```

通过对 *.circle(左右两侧圆环的父元素)* 的裁剪只显示左侧的圆环，右侧的圆环的宽度直接为 0 ，当进度条进行到50%的时候，即左侧圆环转动角度为 *transform: rotate(180deg)* ，将 *.circle* 的裁剪去掉 *(.clip-auto)*，右侧圆环的宽度恢复。基本就是这个套路。

## jQuery的 动画方法 animate() 的 step属性

如果仅用

```
$(this).css({
    "-webkit-transform":"rotate("+n+"deg)",
    "-moz-transform":"rotate("+n+"deg)",
    "transform":"rotate("+n+"deg)"
});
```

来控制角度的转动，没有丝毫动画显得比较生硬，这时候就要考虑给它加个动画了，而jq提供的 *animate* 对只有数字值可创建动画，而字符串类型的值无法创建动画。

这时候需要用到 animate的step属性了。


### step介绍

animate的progress属性是我们经常用的，对数字值的属性进行操作，但是字符串值的属性却并不能用它进行操作，这时候就需要step属性了。

step 顾名思义就是对一段动画进行步骤分解，在animate方法中，每一步具体是怎么分解的，不是由我们设定的CSS属性值和动画时长来决定的，是由系统来决定的。

```
$(".left").animate({left:50},{
    duration:100,
    step:function(now,fx){
        console.log(now) //控制台输出，看看这个动画被分解成了几个片段
    }
});
```
这个地方主要是解释为什么在这里给角度赋值，顺便说明一下它到底将该值分成多少个片段并不是由我们人为控制的。

### step方法的第二个参数——fx


```
$(".demo").animate({
	first:2,
	second:10
}, {
	step:function(n,fx){
		// 动画元素的每个动画属性每一次动画效果的执行都将调用的函数。第1个参数是当前动画正在改变的属性的实时值（每1次动画过程中，属性值的实时反馈呈现）；第2个参数为修改Tween 对象提供了一个机会来改变animate第1个参数中设置的属性在动画结束时的值。
		// fx: jQuery.fx 原型对象的一个引用,其中包含了多项属性,比如
		// 执行动画的元素：elem;
		// 动画正在改变的属性：prop;
		// 正在改变属性的当前值：now;
		// 正在改变属性的结束值：end;
		// 正在改变属性的单位：unit;等

		// 可在这里改变animate第1个参数中设置的属性second在动画结束时的值
		if(fx.prop=="second"){fx.end=5}
		console.log(fx.prop+": "+n);
	},
	duration:2000
})
```

## 结束
> 做这个动画遇到了自己两个知识盲点，clip的运用以及jq的step属性，写这篇文章主要目的还是加深理解，以及分享。