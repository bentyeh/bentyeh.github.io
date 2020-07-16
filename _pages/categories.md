---
title: Categories
layout: default
hidden: true
permalink: /categories/
---

{% assign sorted_categories = site.categories | sort %}

{%- for cat in sorted_categories -%}
  <a href="#{{ cat[0] }}" class="post-tag"><i class="fas fa-bookmark"></i>{{ cat[0] }}</a>
{%- endfor -%}

{% raw %}
<hr>
{% endraw %}

{%- for cat in sorted_categories -%}
  <a name="{{ cat[0] }}" class="post-tag"><i class="fas fa-bookmark"></i>{{ cat[0] }}</a>
  <ul class="post-list">
    {%- for post in cat[1] -%}
      {%- if post.hidden -%}{%- continue -%}{%- endif -%}
      <li>
        <span class="post-meta">{{ post.date | date: site.date_format }}</span>
        <h2>
        <a class="post-link" href="{{ post.url | relative_url }}" title="{{ post.title }}">{{ post.title | escape }}</a>
        </h2>
        {{ post.excerpt | markdownify | truncatewords: 50 }}
      </li>
    {%- endfor -%}
  </ul>
{%- endfor -%}