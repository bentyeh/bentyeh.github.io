Personal website at [bentyeh.github.io](https://bentyeh.github.io/).

# Build

Build locally: `jekyll build [options]`  
Serve locally: `jekyll serve [options]`

Options
- `--baseurl /~bentyeh`: add this if hosting this site at some non-root domain (e.g., *.com/~bentyeh)
- `--drafts`: to show drafts among the latest posts

# Layouts hierarchy

- compress.html: Jekyll layout that compresses HTML (credits: https://github.com/penibelst/jekyll-compress-html)
  - default.html: used by about, blog, and projects pages
    - home.html: used by index page
    - post.html: used by blog posts and individual projects

# Frontmatter variables

## General

Variable            | Description
------------------- | -----------
`layout`            | name of layout from `_layout/` folder
`title`             | title of page
`description`       | description of page
`permalink`         | relative URL path to serve page
`use_code`          | boolean, set to `true` to include code-highlighting CSS
`use_academicons`   | boolean, set to `true` to include Academicons CSS
`use_math`          | boolean, set to `true` to include MathJax JS
`use_toc`           | boolean, set to `true` to include a table of contents

## Posts and Projects

Variable            | Description
------------------- | -----------
`last_updated`      | date in `YYYY-MM-DD` format
`excerpt`           | text to show on blog / project list and at top of post / project
`hidden`            | boolean, set to `true` to keep the post / project "unlisted" (only accessible by direct URL)

### Project-specific

Variable            | Description
------------------- | -----------
`category`          | type of project (e.g., research, experience, other project)
`setting`           | time and location of project
`team`              | team members
`mentors`           | mentors
`collaborators`     | collaborators
`abstract`          | abstract
`thumbnail`         | link to thumbnail image
`thumbnail_caption` | caption below thumbnail image
`paper`             | link to paper; adds icon
`demo`              | link to demo; adds icon
`github`            | link to GitHub repo; adds icon
`video`             | link to video presentation; adds icon

### Post-specific

Variable            | Description
------------------- | -----------
`category`          | post category (1 per post)
`tags`              | tags (many per post)

Theme borrowed with permission from https://chrisyeh96.github.io.
