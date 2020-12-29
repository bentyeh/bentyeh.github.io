---
title: Projects
layout: default
permalink: /projects/
categories:
  - research
  - experience
  - projects
use_academicons: true
---

{%- for category in page.categories -%}
  <h1 class="section-title">{{ category | capitalize }}</h1>
  <ul class="post-list">
  {%- for project in site.projects -%}
    {%- if project.category != category -%}{% continue %}{%- endif -%}
    {%- if project.hidden -%}{%- continue -%}{%- endif -%}
    <li>

      <div class="row content-row">
        <div class="col-3">
          {%- if project.thumbnail -%}
            <img class="thumbnail rounded" src="{{ project.thumbnail }}">
          {%- endif -%}
          {%- if project.thumbnail_caption -%}
            <span class="caption">{{ project.thumbnail_caption }}</span>
          {%- endif -%}
        </div>

        <div class="col-9">
          <h2>
            {%- assign content = project.content | strip_newlines -%}
            {%- if content == "" -%}
              {{ project.title | escape | markdownify | remove: '<p>' | remove: '</p>' }}
            {%- else -%}
              <a class="post-link" href="{{ project.url | relative_url }}" title="{{ project.title }}">{{ project.title | escape | markdownify | remove: '<p>' | remove: '</p>' }}</a>
            {%- endif -%}
          </h2>

          {%- if project.setting -%}
            <p><i>{{ project.setting }}</i></p>
          {%- endif -%}

          {%- if project.team -%}
            <p><strong>Team:</strong> {{ project.team }}</p>
          {%- endif -%}

          {%- if project.mentors -%}
            <p><strong>Mentor(s):</strong> {{ project.mentors }}</p>
          {%- endif -%}

          {%- if project.collaborators -%}
            <p><strong>Collaborator(s):</strong> {{ project.collaborators }}</p>
          {%- endif -%}

          {%- if project.abstract -%}
            {{ project.abstract | markdownify }}</p>
          {%- endif -%}

          {%- if project.excerpt -%}
            {{ project.excerpt | markdownify }}
          {%- endif -%}

          {%- for media in project.media -%}
            <a href="{{ media.url }}" class="btn btn-light mr-2">
              {%- if media.type == "file" -%}<i class="fas fa-file"></i>
              {%- elsif media.type == "biorxiv" -%}<i class="ai ai-biorxiv"></i>
              {%- elsif media.type == "github" -%}<i class="fab fa-github"></i>
              {%- elsif media.type == "video" -%}<i class="fab fa-youtube"></i>
              {%- else -%}<i class="fas fa-external-link-square-alt"></i>
              {%- endif -%}
              &nbsp;{{ media.name }}
            </a>
          {%- endfor -%}
        </div>
      </div>

    </li>
  {%- endfor -%}
  </ul>
{%- endfor -%}
