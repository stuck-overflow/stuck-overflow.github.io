---
title: Setting up our digital garden on GitHub pages, with a custom domain
---

Setting up the website you're looking at didn't take us too long, but if you,
like us, are unfamiliar with Jekyll and GitHub Pages and GitHub Actions, it can
take longer than you'd expect.

# Template and local testing

Configuring the template was fairly easy. We just went through the
[instructions](https://maximevaillancourt.com/blog/setting-up-your-own-digital-garden-with-jekyll)
provided by the author of the template. Having never used Ruby in my life I was
positively impressed by the simplicity of Jekyll. We used the repository
template provided by the author and created a `stuck-overflow.github.io`
repository under our user. To test the content (even to preview the note I'm
writing now!) just run:

```
$ bundle exec jekyll serve
```

and connect to `http://localhost:4000/` to preview the website.

# Configuring GitHub Pages

By default GitHub Pages claims to support Jekyll, however their integrated
support only work with Jekyll 3.x templates. It seems like they never got
around to implement [support for Jekyll
4](https://github.com/github/pages-gem/issues/651). Since our template uses
Jekyll 4.x, we needed an alternative way. Our solution derives from [this
document on the Jekyll
website](https://jekyllrb.com/docs/continuous-integration/github-actions/). We
also consulted the documentation of [the GitHub Action that can build a Jekyll
4 website](https://github.com/helaili/jekyll-action).

The resulting solution is a GitHub Action that on every commit to the `master`
branch re-builds that website pages and commits them to a `gh-pages` branch for
the same repository. To achieve that, it's sufficient to publish a workflow
`.yml` file under `.github/workflows` directory. Our workflow is in a file
named
[`github-pages.yml`](https://github.com/stuck-overflow/stuck-overflow.github.io/blob/master/.github/workflows/github-pages.yml)
and it looks like this at the time of writing:

```yml
{% raw %}
name: Build and deploy Jekyll site to GitHub Pages

on:
  push:
    branches:
      - master

jobs:
  github-pages:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - uses: actions/cache@v2
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile') }}
          restore-keys: |
            ${{ runner.os }}-gems-
      - uses: helaili/jekyll-action@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          target_branch: 'gh-pages'
{% endraw %}
```

That's it. `GITHUB_TOKEN` is a token automatically provided to GitHub Actions
so there's not much more to configure here. At the next commit to `master` you
should be able to see under the GitHub Actions tab whether the action is run
and succeed. The `gh-pages` branch is created automatically on first run.

The next step is to tell the GitHub Pages system to use the `gh-pages` branch
as repository of the pages to be served. You can do so by change the setting in
the repository Settings, under the Pages section, Source subsection.

# Custom domain

This was surprisingly simple. Just follow the excellent
[docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site)
provided by GitHub.
