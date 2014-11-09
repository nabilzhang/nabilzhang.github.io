---
layout: post
title:  "jQuery三种事件绑定方式.bind(),.live(),.delegate()"
date:   2011-09-09 14:25:09
categories: Jquery
---
<div style="color: #000000; font-family: verdana, Arial, Helvetica, sans-serif; font-size: 14px; background-image: initial; background-attachment: initial; background-origin: initial; background-clip: initial; background-color: #ffffff; line-height: 1.5; background-position: initial initial; background-repeat: initial initial; margin: 8px;">
<p><span style="color: #808080;">翻译原文：http://www.alfajango.com/blog/the-difference-between-jquerys-bind-live-and-delegate/</span></p>
<p><span style="color: #808080;"><img class="alignnone size-full wp-image-1490" title="jquery-tags-bind-live-delegate" alt="" src="http://www.alfajango.com/blog/wp-content/uploads/2011/01/jquery-tags-bind-live-delegate.jpg" width="500" height="171" style="border-style: initial; border-color: initial;" /></span></p>
<p><a href="http://api.jquery.com/bind/">.bind()</a>,&nbsp;<a href="http://api.jquery.com/live/">.live()</a>, 和&nbsp;<a href="http://api.jquery.com/delegate/">.delegate()</a>之间的区别并不明显。但是理解它们的不同之处有助于写出更简洁的代码，并防止我们的交互程序中出现没有预料到的bug。</p>
<h3><strong><span style="color: #008000; font-size: 16px;">基础</span></strong></h3>
<h4><strong><span style="color: #008000; font-size: 16px;">DOM树</span></strong></h4>
<p>首先，图形化的HTML文档能帮助我们更好的理解。一个简单的HTML页面看起来应该像这样</p>
<p><img class="alignnone" title="DOM tree sample" alt="DOM tree sample" src="http://chart.apis.google.com/chart?cht=gv&amp;chl=graph{window--document--h1;document--p--span;p--a;document--h2;document--form--input;form--submit;}&amp;chs=550x300" width="550" height="300" style="border-style: initial; border-color: initial;" /></p>
<h4><strong><span style="color: #008000; font-size: 16px;">事件冒泡（也称作事件传递）（Event bubbling aka event propagation）</span></strong></h4>
<p>点击一个链接，触发绑定在链接元素上的&nbsp;<span style="color: #666666;">click</span>&nbsp;事件，进而触发绑定到这个元素的click事件的函数。</p>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('a').bind('click', function() { alert("That tickles!") });
</pre>
</div>
<p>　　</p>
<p>所以一次点击会触发一个alert。<br /><img class="aligncenter" title="trigger the alert" alt="trigger the alert" src="http://chart.apis.google.com/chart?cht=gv&amp;chl=digraph{a[color=red];%22That%20tickles%21%22[shape=rectangle];a-%3E%22That%20tickles%21%22[color=red][label=click][fontcolor=red]}" width="131" height="176" style="border-style: initial; border-color: initial;" /><br />然后，这个&nbsp;<span style="color: #666666;">click</span>&nbsp;事件会从DOM树向上传递，传播到父元素，然后传递给每一个祖先元素。<br /><img class="aligncenter" title="event propagates up the tree" alt="event propagates up the tree" src="http://chart.apis.google.com/chart?cht=gv&amp;chl=digraph{a[color=red];window-%3Edocument[label=%22document%20%3E%20p%20%3E%20a%2Eclick%22][dir=back][color=red][fontcolor=red];document-%3Eh1[dir=none];document-%3Ep[label=%22p%20%3E%20a%2Eclick%22][dir=back][color=red][fontcolor=red];p-%3Espan[dir=none];document-%3Eh2[dir=none];document-%3Eform[dir=none];form-%3Einput[dir=none];form-%3Esubmit[dir=none];p-%3Ea[label=%22a%2Eclick%22][dir=back][color=red][fontcolor=red];}&amp;chs=550x350" width="550" height="350" style="border-style: initial; border-color: initial;" /></p>
<p>在DOM树中，<span style="color: #666666;">&nbsp;document&nbsp;</span>是根节点。<br />现在我们能容易的解释<a href="http://api.jquery.com/bind/">.bind()</a>,&nbsp;<a href="http://api.jquery.com/live/">.live()</a>, 和&nbsp;<a href="http://api.jquery.com/delegate/">.delegate()</a>之间的差别了</p>
<h4><strong><span style="color: #008000; font-size: 16px;">.bind()</span></strong></h4>
<div>
<div id="highlighter_198007" class="syntaxhighlighter  js ie"><span size="3" style="font-size: small;"><span size="3" style="font-size: small;"></span></span>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('a').bind('click', function() { alert("That tickles!") });
</pre>
</div>
<p>　　</p>
</div>
</div>
<p>这是最直接的绑定方法。jQuery 扫描文档找到所有 $(&lsquo;a&rsquo;) 元素，然后给每一个找到的元素的 click 事件绑定处理函数。</p>
<h4><strong><span style="color: #008000; font-size: 16px;">.live()</span></strong></h4>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('a').live('click', function() { alert("That tickles!") });
</pre>
</div>
<p>　　</p>
<p>　　jQuery绑定处理函数到 $(document) 元素，并把 &lsquo;click&rsquo; 和 &lsquo;a&rsquo; 作为函数的参数。有事件冒泡到document节点的时候，检查这个事件是不是 click 事件，target element能不能匹配 &lsquo;a&rsquo; css选择器，如果两个条件都是true，处理函数执行。</p>
<p>live方法也可以绑定到指定的元素（或者说&ldquo;上下文(context)&rdquo;）而不用绑定到document，比如：</p>
<div>
<div id="highlighter_370142" class="syntaxhighlighter  js ie"><span size="3" style="font-size: small;"><span size="3" style="font-size: small;"></span></span>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('a', $('#container')[0]).live(...);
</pre>
</div>
<p>　　</p>
</div>
</div>
<h4><strong><span style="color: #008000; font-size: 16px;">.delegate()</span></strong></h4>
<div>
<div id="highlighter_761686" class="syntaxhighlighter  js ie"><span size="3" style="font-size: small;"><span size="3" style="font-size: small;"></span></span>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('#container').delegate('a', 'click', function() { alert("That tickles!") });
</pre>
</div>
<p>　　</p>
</div>
</div>
<p>jQuery扫描文档找到 $(&lsquo;#container&rsquo;)，绑定处理函数到他的 click 事件，&rsquo;a&rsquo; css选择器作为函数的参数。当有事件冒泡到 $(&lsquo;#container&rsquo;)，检查事件是不是 click，并检查target element是不是匹配css选择器，如果两者都符合，执行函数。</p>
<p>注意这次和 .live() 方法很相似，除了把事件绑定到特定元素与跟元素的区别。精明的JS&rsquo;er 或许会总结成 $(&lsquo;a&rsquo;).live() == $(document).delegate(&lsquo;a&rsquo;)，真的是这样吗？ 不，不全是。</p>
<h4><strong><span style="color: #008000; font-size: 16px;">为什么 .delegate() 比 .live() 好</span></strong></h4>
<p>jQuery 的 delegate方法比 live 方法更应该成为首选有一个原因。考虑以下的场景：</p>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('a').live('click', function() { blah() });
// or
$(document).delegate('a', 'click', function() { blah() });
</pre>
</div>
<p>　　</p>
<h4><strong><span style="color: #008000; font-size: 16px;">速度</span></strong></h4>
<p>上面第二个执行比第一个快，因为第一个会遍历整个文档查找 $(&lsquo;a&rsquo;) 元素，并保存为jQuery对象，但是live方法只需要传一个字符串参数&rsquo;a'而已，$() 方法并不知道我们会用链式表达式在后面用上.live()。</p>
<p>delegate 方法就只需要找到并存贮 $(document)元素就够了。</p>
<blockquote>
<p>有一种hack是在 $(document).ready()之外调用live方法，这样就会立即执行。这时候DOM还没有填充，也就不会查找元素或创建jQuery对象。</p>
</blockquote>
<h4><strong><span style="color: #008000; font-size: 16px;">灵活性和链式语法</span></strong></h4>
<p>这方面live方法依然令人费解。想一下，它链在$(&lsquo;a&rsquo;)对象，但实际上是在$(document)对象起作用。因为这个原因，在链式表达式中使用live让人很不安，我觉得live方法变成一个全局的jQuery方法 $.live(&lsquo;a&rsquo;,&hellip;) 会更有意义。</p>
<h4><strong><span style="color: #008000; font-size: 16px;">只支持css选择器</span></strong></h4>
<p>最后，live方法有一个最大的缺点，只能用css选择器，用起来很不方便。</p>
<blockquote>
<p>有关css选择器的缺点，参看&nbsp;<a href="http://www.alfajango.com/blog/exploring-jquery-live-and-die/">Exploring jQuery .live() and .die()</a>。</p>
</blockquote>
<blockquote>
<p>原作者更新</p>
</blockquote>
<h4><strong><span style="color: #008000; font-size: 16px;">为什么使用 .live() 或 .delegate() 而不用 .bind()</span></strong></h4>
<p>最后，bind 方法看起来更清晰，更直接，是吗？但是这里有两个原因我们推荐 delegate 或 live:</p>
<ul>
<li>绑定事件处理函数到还不存在DOM中的元素。 bind 方法直接绑定函数到每个单独的元素，不能绑定到还没有添加到页面里的元素，如果你写了$(&lsquo;a&rsquo;).bind(&hellip;)，然后用ajax给页面增加了新的链接，新添加的链接不会绑定事件。live 或 delegate 或者其它绑定到祖先元素的事件，让现在有的元素，或者以后增的元素都可以使用。</li>
<li>绑定处理函数到一个元素或者少数几个元素，监听后代元素，而不是绑定100个相同的处理函数到单独的元素。这样更有性能优势。</li>
</ul>
<h4><strong><span style="color: #008000; font-size: 16px;">阻止冒泡</span></strong></h4>
<p>最后注意一下事件冒泡。通常我们能用这样的方法阻止其他处理函数：</p>
<div class="cnblogs_Highlighter">
<pre class="brush:javascript;gutter:true;">$('a').bind('click', function(){   
    e.preventDefault();    
    //or    
    e.stopPropagation();
})
</pre>
</div>
<p>　　</p>
<p>但是在这里，用 live 或 delegate 方法绑定的事件会一直传递到事件真正绑定的地方才会执行。这时其他的函数已经执行过了。</p>
</div>