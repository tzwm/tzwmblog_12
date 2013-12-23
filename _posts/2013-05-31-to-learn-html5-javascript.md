---
layout: post
title: HTML5&amp;Javascript学习
date: 2013-05-31 14:29:39
tags: [前端, Learn, HTML5, Javascript]
---

这几天在写Web客户端课程项目，也就是做个围棋游戏吧。因为不想写后端，所以就想把前端写得cool一点。  

为了各种动画效果，所以整个棋盘都是用HTML5的canvas给画出来的。画这种东西其实还是比较方便的，几个矩形一堆线条就搞定了。但在学canvas的时候，发现网上教程大多是用clearRect方法来擦除canvas进行重新绘制来达到动画效果。但我实际使用，效果非常差。不知道clearRect具体是如何实现的，貌似是擦除以后填充透明元素，反正我用起来感觉是在原来的canvas上重新绘制，非常浪费资源，cpu占用有点高。后来我无意中发现，其实想要重置一个canvas，直接重新设置一下它的尺寸，也就是长宽属性就好了。这相当于重新给你new了一个canvas的感觉，相当便捷省资源，直接新的canvas上重画就好了。至少我目前使用来说，还没发现什么不好的地方，先这么用着吧。  

这两天学Javascript也被它的对象弄得狼狈。“类”的几种声明方式什么的就算了，我被一个作用域的问题纠结了很久（果然是我弱爆了）。  

{% highlight javascript %}
function Test(){
    this.a = 0;

    this.callA = function(){
        if(this.a>2)
            return;
        alert(this.a);
        this.a++;
    };
    
    this.call = function(){
        setInterval(this.callA(), 100);
    };
}

var test = new Test();
test.call();
{% endhighlight %}

比如上面这段代码，我是想在通过setInterval执行3次callA方法，用Test的成员a来控制次数。但实际运行就会发现，在用setInterval执行this.callA()的时候，是找不到this.a这个类变量的。原因很简单，debug的时候就会发现，实际执行到callA()里时，this.a是找不到的，this已经指向了window。在网上我看教程是说是因为this是动态晚绑定的，按我的理解就是this是指向最近的一层对象。  

解决这个问题的方法好像很多，有用命名空间的，看起来比较美。但我图简单，用的是下面这种。  

{% highlight javascript %}
function Test(){
    this.a = 0;

    this.callA = function(){
        if(this.a>2)
            return;
        alert(this.a);
        this.a++;
    };
    
    this.call = function(){
        tmp = this;
        setInterval("tmp.callA()", 100);
    };
}

var test = new Test();
test.call();
{% endhighlight %}

也就是先用一个全局变量存下当前这个对象的this，然后用碉堡了的eval给setInterval，这里要切记加引号。  


====2013-12-18 Update=====

时隔几个月，我重新把这个项目clone下来重新跑的时候，居然发现出bug了。

Bug在于，我下的棋子在棋盘上显示不出来了。但我确实又能检测出那个点确实已经是下了子的。

然后我就想到，这估计是棋子的图片没有加载出来的问题。

我首先查了下文档，排除了这短短几个月来，canvas发生了改变导致加载图片的方式不同了，这种狗血的极低概率事件。

接着我就去写了个小demo，想要直接在canvas中加载图片出来，发现居然也不行。这就比较悲剧了，我又换了个浏览器，排除了浏览器的问题。

最后，还是不得不去搜了一下，发现是图片的异步加载问题。当定义了一个新的Image元素时，在设置了src以后，他其实是异步加载的，所以如果浏览器里没有缓存什么的话，确实会导致图片显示不出来的问题。

后来想想，图片显然应该是异步加载的，所以只能说是我当时写的时候太嫩了，没有想到这个问题。至于当时没有发生这个问题，我估计是因为我当时一步步写下来的时候，之前已经有浏览过那张图片资源，浏览器中已经有缓存了，所以没有发生这种情况。

最后，解决方案也很简单，网上的好像都是去判断是否加载完成。我因为这个图片加载并不是网页加载出来的时候就必要的，所以即时性并不是要求很高，所以直接在加载网页的时候定义了这个Image。这样，在之后用的时候，基本上也就加载完成了。其实，我感觉我这个方案虽然极其简单，但其实不是很好，我也就偷个懒了……



参考：  

[关于setInterval调用对象方法的问题](http://www.cnblogs.com/sparon/archive/2011/04/26/2029268.html)

