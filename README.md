# Install Jekyll

We want to [use Jekyll to create a GitHub Pages site](https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/setting-up-a-github-pages-site-with-jekyll). GitHub Pages relies on a set of dependencies, which can be found [here](https://pages.github.com/versions/). To set up a local Jekyll environment in sync with GitHub Pages, we can simply install the `github-pages` Ruby gem ([GitHub repo](https://github.com/github/pages-gem); [RubyGems](https://rubygems.org/gems/github-pages)) provided by GitHub.

To install the `github-pages` Ruby gem within a conda environment, a couple options are suggested below.

1. (Recommended) Create a new conda environment named `ruby` with required dependencies using the `env_ruby.yml` requirements file provided in the repo.

```bash
conda env update -f env_ruby.yml --prune
conda activate ruby
gem install github-pages
```

If `gem install github-pages` fails, there may be missing system libraries and compilation tools. If running Ubuntu, try the following:

```bash
sudo apt update  # update package index
sudo apt upgrade build-essential  # compilation tools
```

2. (Easier, but may be outdated) Directly install the [compiled `github-pages` gem from the `conda-forge` channel](https://anaconda.org/conda-forge/rb-github-pages).

```bash
conda install conda-forge::rb-github-pages
```

# Build

Build locally: `jekyll build [options]`  
Serve locally: `jekyll serve [options]`

Options
- `--baseurl /~<name>`: add this if hosting this site at some non-root domain (e.g., *.com/~\<name\>)
- `--drafts`: to show drafts among the latest posts

# Layouts hierarchy

- compress.html: Jekyll layout that compresses HTML (see [Credits](#Credits))
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
`abstract`          | abstract (only shown on project list page, not on the post page; compare with `excerpt`)
`thumbnail`         | path to thumbnail image
`thumbnail_caption` | caption below thumbnail image
`media`             | unordered list of external content; specify `url`, `name`, and `type` fields (edit `_pages/projects.md` to support additional types)

### Post-specific

Variable            | Description
------------------- | -----------
`tags`              | tags (many per post)

# Credits

- Theme: borrowed with permission from https://chrisyeh96.github.io
- `_layouts/compress.html`: https://github.com/penibelst/jekyll-compress-html
- `_layouts/toc.html`: Table of contents generator from https://github.com/allejo/jekyll-toc