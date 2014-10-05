---
layout: page
title: 分类
permalink: /categories/
---
  <ul class="nav nav-tabs-vertical">
      {% assign categories_list = site.categories %}
      {% if categories_list.first[0] == null %}
        {% for category in categories_list %}
            <li>
                <a href="{{ site.BASE_PATH }}/{{ site.categories_path }}#{{ category | replace:' ','-' }}-ref" data-toggle="tab">
                  {{ category | capitalize }} <span class="badge pull-right">{{ site.categories[category].size }}</span>
               </a>
            </li>
        {% endfor %}
      {% else %}
        {% for category in categories_list %}
            <li>
                <a href="{{ site.BASE_PATH }}/{{ site.categories_path }}#{{ category[0] | replace:' ','-' }}-ref" data-toggle="tab">
                    {{ category[0] | capitalize }} <span class="badge pull-right">({{ category[1].size }})</span>
                </a>
            </li>
        {% endfor %}
      {% endif %}
      {% assign categories_list = nil %}
    </ul>