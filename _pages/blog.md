---
title: Blog
layout: default
permalink: /blog/
---

<ul class="post-list">
{%- for post in site.posts -%}
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