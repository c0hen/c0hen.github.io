---
title: README
layout: default
tags: siteinfo
---

# c0hen.github.io {#top}

Testing Jekyll and trying to organize my tinkerings.

Install gems via bundler into the project:

```sh
bundle config set --local path 'github/jekyll'
bundle config list
bundle install
```

Serve the site locally:

```sh
bundle exec jekyll serve
```

Highlighting with
Rouge 
https://github.com/jneen/rouge/wiki/List-of-supported-languages-and-lexers

rougify style github > rouge.css

Github.io's Jekyll site rebuilds after a push.

Markdown (`*.md`) comments:

```
[//]: # (This may be the most platform independent comment)

[//]: # (Note to self: no <p> around markdown or parser switches to inline html.)

[//]: # (Class needs to be on the same line with image, otherwise an empty <p> will get the attr.)

[//]: # (![Blue buffalo of spades](/images/favicon-152.png){:class="centered-wrapper" height="100px" width="50px"})
```

Liquid Snippets in the _includes directory.

Disable liquid inline in case of code conflicts using `raw` and `endraw`.

Jump to heading on [another page](/useful-commands/#vim) with id `vim` on this site.
