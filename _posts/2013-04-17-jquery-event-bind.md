---
layout: post
title: 对jQuery事件绑定的一些思考
published: true
---

## 问题

首先我们看下面的一个很常见的事件绑定代码：

    //example
    $('#dom').click(function(e){
      //do something
    });

    $('#dom2').click(function(e){
      //do something
    });

这段代码在事件绑定处理上有一些缺陷：

1.  过多的事件绑定会损耗内存
2.  后期生成HTML会没有事件绑定，需要重新绑定
3.  语法过于繁杂

## 解决方案

对于1、2两点的解决方案，我们首先先了解一下jQuery的事件绑定

jQuery的事件绑定有多个方法可以调用，以click事件来举例：

-   click方法
-   bind方法
-   delegate方法
-   on方法

不管你用的是（click / bind / delegate)之中那个方法，最终都是jQuery底层都是调用on方法来完成最终的事件绑定。因此从某种角度来讲除了在书写的方便程度及习惯上挑选，不如直接都采用on方法来的痛快和直接。

> 关于对方法的详细解释和用例，请直接访问jQuery官网，在这里不一一说明。[api.jquery.com](api.jquery.com)

### 性能

首先我们需要先对不同的事件绑定方式之间的内存占用差距有一个清晰的认识。

对于性能的分析将采用Chrome的Developer Tools。
Profiles --\> Take Heap Snapshot，用这个工具我们可以看到Javascript所占用的内存，能够对性能问题进行分析。

![](https://writingsio.s3.amazonaws.com/attachments/516ff8f94017a4bb1f00038d/3bfaa34fb0c29f647fe3ec8d31c9ab41/1.png)

#### DEMO HTML

    <html>
      <head>
        <script type="text/javascript">
          $(function(){
            $('#btn-add').click(function(){
              $('.ul').prepend('<li><a href="javascript:;">text</a></li>');
            });
          });
        </script>
      </head>
      <body>
        <button id="btn-add">Create Element</button>
        <ul class="ul">
          <li><a href="javascript:;">text</a></li>
          <!-- 2000 line... -->
          <li><a href="javascript:;">text</a></li>
        </ul>
      </body>
    </html>

#### Method 1

    $(function(){    $('.ul a').click(function(e){
            alert('click event');
        });
    });

以下是Method 1的内存分析图

-   内存占用约**3.4M**

![](https://writingsio.s3.amazonaws.com/attachments/517091174017a4bb1f0003dc/08bd5c1f0488af64e659bc348db4cf69/one.png)

#### Method 2

    $(function(){
        $('.ul').on('click', 'a', function(e){
            alert('click event');
        });
    });

以下是Method 2的内存分析图

-   内存占用约**2.0M**

![](https://writingsio.s3.amazonaws.com/attachments/5170913d4017a4298d000668/3fdc55c1b7a9332d1a29132e125a1951/two.png)

#### 结论

1.  Method 1 明显比 Method 2 多耗**1.4M**的内存
2.  Method 1 无法将事件绑定到通过点击button所新增DOM中来，而Method 2可以。

只要on的delegate对象是HTML页面原有的元素，由于是事件的触发是通过Javascript的事件冒泡机制来监测，所以对于所有子元素（包括后期通过JS生成的元素）所有的事件监测均能有效，且由于不用对多个元素进行事件绑定（在这个example中为2000+a标签），能够有效的节省内存的损耗。

## 思考

代码如诗，但很容易变成代码如屎。如何提高代码的优雅程度也是一个很有意思的事情。

以下是一个很普通且普遍的JS文件的代码片段（用于一般网站）

    $('#btn-add').click(function(){
      //do something
    });
    $('.action-box #btn-delete').click(function(){
      //do something
    });
    $('.action-box #btn-sort').mouseenter(function(){
      //do something
    });
    /**
    **more same code
    */

毫不夸张的说，当一个js文件上百行后，类似于上面的代码，你很难从里面发现规律。

1.  可能A喜欢写***\#btn-add***，而B喜欢写***.action-box \#btn-add***来作为选择符。
2.  堆砌着许多不同类型事件，没有一个次序可言
3.  没有运用到我们刚刚所讲的利用事件冒泡来做事件绑定

![](https://writingsio.s3.amazonaws.com/attachments/517019d04017a4b843000424/f54c247af69df22357b254dbcbc25747/2007113134613123_2.jpg)

## 改进

我们来一步步改进一下之前的JS代码

#### Version 1

    $('.action-box').on('click', '#btn-add', function(){
      //do something
    });
    $('.action-box').on('click', '#btn-delete', function(){
      //do something
    });

虽然运用了事件冒泡，不过感觉还是有点累赘，**.action-box**出现多次，感觉不舒服，让我们继续改进

#### Version 2

    $('.action-box').on('click', '#btn-add， #btn-delete', function(){
      if($(this).attr('id') == 'btn-add'){
        //do something
      } else{
        //do something
      }
    });

感觉比刚刚好多了，不过还是需要判断元素来做出相应的处理，能接受，但不完美。

### 灵感

首先看一下css的增强版本sass对于css语法上面的改进

    /*bed css code*/
    .action-box { width: 100%; color: #000; }
    #btn-add { color: blue; }
    #btn-delete { color: red; }

    /*good css code*/
    .action-box { width: 100%; color: #000; }
      .action-box #btn-add { color: blue; }
      .action-box #btn-delete { color: red; }

    /*sass code*/
    .action-box {
      width: 100%;
      color: #000;
      #btn-add {
        color: blue;
      }
      #btn-delete {
        color: red;
      }
    }

我们可以在 **good css code **和 **sass code **从中可以可以很清晰了然的看到文档结构：.action-box 下面有两个button。

这是否能让sass这种代码结构运用到js中来呢？答案当然是可以。

    $('.action-box').coffee({
      click: {
        '#btn-add': function(){
          //do something
        },
        //这是是支持jQuery的':last / [attr] / :eq(0)'等方法的 
        '#btn-delete': function(){
          //do something
        }
      },
      mouseenter: {
        '#btn-sort': function(){
          //do something
        }
      }
    });

喜欢这种结构吗？

1.  清晰明了的文档结构
2.  运用事件冒泡，有效减少内存的占用
3.  第一级别用事件名称来划分
4.  第二级别的属性名相当于选择符。

#### coffee函数的源码

    $.fn.coffee = function(obj){
      for(var eName in obj)
        for(var selector in obj[eName])
          $(this).on(eName, selector, obj[eName][selector]);
    }

聊聊数行代码，就可以做成一个很美妙的**语法糖**

**Enjoy yourself !  \^\_\^**
