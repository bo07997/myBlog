---
layout: post
title:  "关于python引用和copy的问题"
date:   2018-01-03 17:55:23
comments: false
tags:
  - python
description: 本文主要在一次开发过程中遇到的关于python引用和队列复制的问题，并且提出一些解决方案                                                         
---
### introduction

由于最近使用的新语言比较多，每次新上手一种语言我都会第一关心它是传值还是引用，因为很多地方都要用到，一不小心可能就会出错，之前很果断的给python定性为复杂对象传引用，导致实际工作中出现一些问题。

### 1. 传对象 or 引用？

这里不细谈，稍微引用一下一篇比较棒的博客，下面是一些摘抄

Python 与众不同的变量储存方法
Python 存储变量的方法跟其他 OOP 语言不同。它与其说是把值赋给变量，不如说是给变量建立了一个到具体值的 reference。

当在 Python 中 a = something 应该理解为给 something 贴上了一个标签 a。当再赋值给 a 的时候，就好象把 a 这个标签从原来的 something 上拿下来，贴到其他对象上，建立新的 reference。 这就解释了一些 Python 中可能遇到的诡异情况：
{% highlight markdown %}  
>>> a = [1, 2, 3]
>>> b = a
>>> a = [4, 5, 6] //赋新的值给 a
>>> a
[4, 5, 6]
>>> b
[1, 2, 3]
# a 的值改变后，b 并没有随着 a 变

>>> a = [1, 2, 3]
>>> b = a
>>> a[0], a[1], a[2] = 4, 5, 6 //改变原来 list 中的元素
>>> a
[4, 5, 6]
>>> b
[4, 5, 6]
# a 的值改变后，b 随着 a 变

{% endhighlight %} 
上面两段代码中，a 的值都发生了变化。区别在于，第一段代码中是直接赋给了 a 新的值（从 [1, 2, 3] 变为 [4, 5, 6]）；而第二段则是把 list 中每个元素分别改变。
而对 b 的影响则是不同的，一个没有让 b 的值发生改变，另一个变了。

怎么用上边的道理来解释这个诡异的不同呢？
首次把 [1, 2, 3] 看成一个物品。a = [1, 2, 3] 就相当于给这个物品上贴上 a 这个标签。而 b = a 就是给这个物品又贴上了一个 b 的标签。
![](https://bo07997.github.io/dianbo/images/Blog/python1/1.png)
**第一种情况：**
a = [4, 5, 6] 就相当于把 a 标签从 [1 ,2, 3] 上撕下来，贴到了 [4, 5, 6] 上。

在这个过程中，[1, 2, 3] 这个物品并没有消失。 b 自始至终都好好的贴在 [1, 2, 3] 上，既然这个 reference 也没有改变过。 b 的值自然不变。
![](https://bo07997.github.io/dianbo/images/Blog/python1/2.png)

**第二种情况：**
a[0], a[1], a[2] = 4, 5, 6 则是直接改变了 [1, 2, 3] 这个物品本身。把它内部的每一部分都重新改装了一下。内部改装完毕后，[1, 2, 3] 本身变成了 [4, 5, 6]。
而在此过程当中，a 和 b 都没有动，他们还贴在那个物品上。因此自然 a b 的值都变成了 [4, 5, 6]。
![](https://bo07997.github.io/dianbo/images/Blog/python1/3.png)

## 2.关于copy和deepcopy

python的copy主要有2中copy方法，一是copy，二是deepcopy，那么它们有什么区别呢？
很简单，copy是浅层次的copy，如果你的copy对象时比较复杂的对象，例如map里面嵌套着map，那么它只会单独copy第一层，深层次的对象会和原来的copy对象共享。

即你改变了一个，另外一个也会改变。
而deepcopy则是完全copy。

以下是copy的例子
{% highlight markdown %}  
    map_test = {}
    map_test["a"] = "aa"
    map_test["b"] = {"bb": "bbb"}
    copy_map = copy.copy(map_test)
    copy_map["b"]["bb"] = "change"
    print copy_map
    print map_test
{% endhighlight %} 
结果
{% highlight markdown %}
{'a': 'aa', 'b': {'bb': 'change'}}
{'a': 'aa', 'b': {'bb': 'change'}}
{% endhighlight %}  
 可以看到。2个对象都改变了
 
## 3.由copy Queue引发的问题

实际问题:由于线上公用一个优先队列（python有实现），每个单独的用户环境要处理这一个队列的时候都要单独的去copy这个队列，从上面知道，我们不能用copy，因为该队列
比较复杂，层数较深，所以使用deepcopy。

但是使用deepcopy又引发了一个致命的错误（暂时没发现根本原因，下面的解决方法也只是规避了该错误）
代码如下:
{% highlight markdown %}
    que = Queue.Queue(maxsize=50)
    que.put("one")
    que.put("two")
    que.put("three")

    copy_que = copy.deepcopy(que)
    copy_que.put("four")
    copy_que.get()
    a = 0
{% endhighlight %} 

`错误提示:TypeError: can't pickle thread.lock objects `

解决方法:手动复制，将queue里面的值复制出来，再新建一个队列（注意:这不是get()方法取出，不会影响原来的队列），取出代码如下
{% highlight markdown %}
    que = Queue.Queue(maxsize=50)
    que.put("one")
    que.put("two")
    que.put("three")
    copy_que = copy.deepcopy(que.que)
 {% endhighlight %}   

`引用:https://iaman.actor/blog/2016/04/17/copy-in-python`






