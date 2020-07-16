---
title: Tags
layout: default
hidden: true
permalink: /tags/
---

{% assign sorted_tags = site.tags | sort %}

{%- for tag in sorted_tags -%}
  <a href="#{{ tag[0] }}" class="post-tag"><i class="fas fa-tag"></i>{{ tag[0] }}</a>
{%- endfor -%}

{% raw %}
  <hr>
{% endraw %}

{%- for tag in sorted_tags -%}
  <a name="{{ tag[0] }}" class="post-tag"><i class="fas fa-tag"></i>{{ tag[0] }}</a>
  <ul class="post-list">
    {%- for post in tag[1] -%}
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
{%- endfor -%}