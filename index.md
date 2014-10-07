---
layout: default
---

<div class="home">

  <h1>文章</h1>

  <ul class="posts">
    {% for post in site.posts %}
      <li>
        <span class="post-date"><a href="{{ post.categories[0] || prepend:"/categories/#" || prepend: site.baseurl }}-ref">{{post.categories[0]}}</a>   {{ post.date | date: "%Y年%m月%d日" }}</span>
        <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
      </li>
    {% endfor %}
  </ul>
</div>
