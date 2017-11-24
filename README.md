# bentyeh
Personal website

Live websites at [https://bentyeh.github.io/](https://bentyeh.github.io/) and [https://stanford.edu/~bentyeh](https://stanford.edu/~bentyeh).

**Build Locally**

- GitHub Pages: `jekyll build`
- Stanford: `jekyll build --baseurl /~bentyeh`

**Run Locally**

`jekyll serve --baseurl /~bentyeh`

We need the `baseurl` parameter for Stanford because the live website is not at the root domain. However, we don't include `baseurl` in the `_config.yml` file so that [https://bentyeh.github.io/](https://bentyeh.github.io/) also works.

Theme borrowed with permission from [chrisyeh96.github.io](https://chrisyeh96.github.io/).