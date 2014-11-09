---
layout: post
title:  "jQuery的.live()和.die()"
date:   2011-09-09 14:25:09
categories: Jquery
---
<div style="color: #000000; font-family: verdana, Arial, Helvetica, sans-serif; font-size: 14px; background-image: initial; background-attachment: initial; background-origin: initial; background-clip: initial; background-color: #ffffff; line-height: 1.5; background-position: initial initial; background-repeat: initial initial; margin: 8px;">
<p>翻译原文地址：http://www.alfajango.com/blog/exploring-jquery-live-and-die/</p>
<p><img class="alignnone size-full wp-image-829" style="border-style: initial; border-color: initial;" title="jquery-tags-live-die" src="http://www.alfajango.com/blog/wp-content/uploads/2010/10/jquery-tags-live-die.jpg" alt="jQuery .live(), .die()" width="500" height="171" /></p>
<p>很多开发者都知道jQuery的.live()方法，他们大部分知道这个函数做什么，但是并不知道是怎么实现的，所以用的并不那么舒适。而且他们却从未听过还有解除绑定的.live()事件的.die()方法。即使你熟悉这些，但是你意识到.die()了吗？</p>
<h2><strong><span style="font-size: 16px;">什么是 .live()</span></strong></h2>
<p>.live方法类似于.bind()，除此之外，它允许你将事件绑定到DOM元素上，可以将事件绑定到DOM中还不存在的元素上，看看下面的例子：</p>
<p>比方说当用户在点击链接时及想提示他们正在离开站点。</p>
<div class="codecolorer-container javascript twitlight" style="width: 100%; overflow: auto; white-space: nowrap;">
<div class="javascript codecolorer">
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$(document).ready( function() {
    $('a').click( function() {
        alert("You are now leaving this site");
        return true;
    });
});</pre>
</div>
</div>
</div>
<p>注意，.click()仅仅是个实现更一般.bind()的简单方法，下面和上面的代码相当于上面的实现。</p>
<div class="codecolorer-container javascript twitlight" style="width: 100%; overflow: auto; white-space: nowrap;">
<div class="javascript codecolorer">
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$(document).ready( function() {
    $('a').bind( 'click', function() {
        alert("You are now leaving this site");
        return true;
    });
});</pre>
</div>
</div>
</div>
<p>但是现在通过javascript添加一个链接到页面。</p>
<div class="codecolorer-container javascript twitlight" style="width: 100%; overflow: auto; white-space: nowrap;">
<div class="javascript codecolorer">
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('body').append('&lt;div&gt;&lt;a href="..."&gt;Check it out!&lt;/a&gt;&lt;/div&gt;');</pre>
</div>
</div>
</div>
<p>然而当用户点击那个链接是，方法将不会被调用，因为那个链接当你将click事件绑定到页面的所有&lt;a&gt;节点时还并不存在，所以我们就用.live()替换.bind()：</p>
<div class="codecolorer-container javascript twitlight" style="width: 100%; overflow: auto; white-space: nowrap;">
<div class="javascript codecolorer">
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$(document).ready( function() {
    $('a').live( 'click', function() {
        alert("You are now leaving this site");
        return true;
    });
});</pre>
</div>
</div>
</div>
<p>现在如果你添加一个新的链接到页面上，绑定就也可以运行了。</p>
<h2><strong><span style="font-size: 16px;">.live() 如何工作</span></strong></h2>
<p>.live()背后神奇的地方就在于它并不将事件绑定到你选定的elements上，而实际上是绑定到了DOM树的跟节点（例子中是$(document)）,而是在element中就像一个参数一样进行传递。</p>
<p>那么当你点击一个元素时，click事件就会在DOM树上往上传递，直至到达根节点。这个.click()事件的触发器已经在根节点被.live()创建。这个触发方法将首先检测被点击的目标看是否和.live()调用的选择器相匹配。所以上面的例子中，会检查点击的元素是否和$('a').live()中的$('a')相匹配，如果匹配，那么绑定的方法就会执行了。</p>
<p>因为不管你在根节点内点击了什么，根节点的.click()事件都会被触发，当你点击加入到根节点的任何元素时这个检查都会发生。</p>
<h2><strong><span style="font-size: 16px;">所有.live() 都可以.die()</span></strong></h2>
<p>如果你知道.bind()，那么你肯定知道.unbind()。那么，.die()和.live()就是类似的关系了。为了接触上面的绑定（不希望用户点击链接时弹出对话框），我们这么做：</p>
<div class="codecolorer-container javascript twitlight" style="width: 100%; overflow: auto; white-space: nowrap;">
<div class="javascript codecolorer">
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('a').die();</pre>
</div>
</div>
</div>
<p>更具体点，如果还有其他的事件被绑定且需要保留，例如hover或其他，可以只解除click事件绑定。</p>
<div class="codecolorer-container javascript twitlight" style="width: 100%; overflow: auto; white-space: nowrap;">
<div class="javascript codecolorer">
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('a').die('click');</pre>
</div>
</div>
</div>
<p>再具体些，如果已经定义了方法名，可以解除绑定指定的方法。</p>
<div class="codecolorer-container javascript twitlight" style="width: 100%; overflow: auto; white-space: nowrap;">
<div class="javascript codecolorer">
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">specialAlert = function() {
    alert("You are now leaving this site");
    return true;
}

$(document).ready( function() {
    $('a').live( 'click', specialAlert );
    $('a').live( 'click', someOtherFunction );
});

// then somewhere else, we could unbind only the first binding
$('a').die( 'click', specialAlert );</pre>
</div>
</div>
</div>
<p><strong><span style="font-size: 16px;">关于 .die()的问题</span></strong></p>
<p>使用这些函数时，.die()方法会有一个缺点。只可以使用.live()方法中用到的元素选择器，例如，不可以像下面这样写：</p>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$(document).ready( function() {
   $('a').live( 'click', function() {
     alert("You are now leaving this site");
     return true;
   });
 });
 
// it would be nice if we could then choose specific elements
 //   to unbind, but this will do nothing
 $('a.no-alert').die();
</pre>
</div>
<p>　　</p>
<p>.die()事件看起来好像是匹配到了目标选择权并解除了.live()的绑定，但事实上，$<span class="br0">(</span><span class="st0">'a.no-alert'</span><span class="br0">)并不存在绑定，所以jquery找不到任何绑定去去掉，就不会起作用了。</span></p>
<p>更糟的是下面这个:</p>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$(document).ready( function() {
     $('a,form').live( 'click', function() {
         alert("You are going to a different page");
         return true;
    });
});

// NEITHER of these will work
$('a').die();
$('form').die();

// ONLY this will work
$('a,form').die();</pre>
</div>
<h2><strong><span style="font-size: 16px;">如何修复 .die()</span></strong></h2>
<p>在我下篇文章中，我将会建议一种.die()执行的新方法，它可以提供一个向后兼容的语气的行为。或许我有时间的话会去建议jQuery核心开发团队在下一个release中接受我的建议并进行修改，希望多一点我刚写的这些方法，包括可选的context参数，允许自定义事件附加的节点，而不是根节点。</p>
<p>如果想得到更多的信息和例子，可以查查jQuery&nbsp;<a href="http://api.jquery.com/live/">.live()</a>&nbsp;and&nbsp;<a href="http://api.jquery.com/die/">.die()</a>.的文档</p>
<p>同时注意下&nbsp;<a href="http://api.jquery.com/delegate/">.delegate()</a>&nbsp;和<a href="http://api.jquery.com/undelegate/">.undelegate()</a>，他们可以替代.live()和.die()，它们联系很紧密。</p>
</div>