---
title: Blog
layout: default
permalink: /blog/
---

<h1 class="page-title">{{ page.title | escape | markdownify | remove: '<p>' | remove: '</p>' }}</h1>

Posts below are sorted by date published. Alternatively, explore posts by <a href="/tags/">tags <i class="fas fa-tag"></i></a>.

<ul class="post-list">
{%- for post in site.posts -%}
  {%- if post.hidden -%}{%- continue -%}{%- endif -%}
  <li>
    <span class="post-meta">{{ post.date | date: site.date_format }}</span>
    {% for tag in post.tags %}
      <a href="/tags/#{{ tag }}" class="post-meta post-tag">
        <i class="fas fa-tag"></i>{{ tag }}
      </a>
    {% endfor %}
    <h2>
      <a class="post-link" href="{{ post.url | relative_url }}" title="{{ post.title }}">{{ post.title | escape }}</a>
    </h2>
    {{ post.excerpt | markdownify | truncatewords: 50 }}
  </li>
{%- endfor -%}
</ul>