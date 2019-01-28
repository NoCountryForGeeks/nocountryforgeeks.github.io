# No country for Geeks

## Getting started

### Launch in localhost

1. Install a full [Ruby development environment](https://jekyllrb.com/docs/installation/)

2. Install Jekyll and bundler gems

`gem install jekyll bundler`

3. Run `bundle install` to install missing gems.

4. Build the site and make it available on a local server

`bundle exec jekyll serve -d _site`

5. Now browse to http://localhost:4000

**Note:** you can build the site with draft using `--draft`. 

`bundle exec jekyll serve --draft -d _site`

### Create Author

To create author you have to add markdown file with author's information at the author folder.

```markdown
---
layout: author
username: maktub82
displayname: Sergio Gallardo Sales
location: Madrid
url_full: https://twitter.com/maktub82
url: twitter.com/maktub82
bio: Delivery Manager en Plain Concepts
picture: assets/images/blog/authors/maktub82.jpeg
facebook: False
twitter: maktub82
feed: False
cover: False
robots: noindex
---
```

Then you have to modify the `_data/authors.yml` to add the author information.

```yml
maktub82:
  username: maktub82
  name: Sergio Gallardo Sales
  location: Madrid
  url_full: https://twitter.com/maktub82
  url: twitter.com/maktub82
  bio: Delivery Manager en Plain Concepts
  picture: assets/images/blog/maktub82.jpeg
  facebook: False
  twitter: maktub82
  feed: False
  cover: False
```

### Create tag

To add a new tag you have to create a file in `tag` folder with this format.

```
---
layout: tag
title: "C#"
description: False
tag: csharp
robots: noindex
---
```

## Posts

To create new post you have to create a new file in `_posts` folder.

File: `2017-01-08-hemos-tenido-un-sueno.md`

```md
---
layout: post
current: post
cover: assets/images/posts/2017-01-08-hemos-tenido-un-sueno/header.jpg
navigation: True
title: "Hemos tenido un sueño"
date: 2017-01-08 12:00:00
tags: nocountryforgeeks
class: post-template
subclass: 'post'
author: aclopez
---

Here the post content
```

You have to add all images in the `assets/images/post/name_of_posts/` folder.

### Drafts

If you want to cretea a draft you have to create file in _draft folder. To launch blog in localhost with draft you have to use --draft when launch the blog.

## Jasper2

[![Build Status](https://travis-ci.org/jekyller/jasper2.svg?branch=master)](https://travis-ci.org/jekyller/jasper2)
[![Ruby](https://img.shields.io/badge/ruby-2.4.2-blue.svg?style=flat)](http://travis-ci.org/jekyller/jasper2)
[![Jekyll](https://img.shields.io/badge/jekyll-3.6.2-blue.svg?style=flat)](http://travis-ci.org/jekyller/jasper2)

This is a full-featured port of Ghost's default theme [Casper](https://github.com/tryghost/casper)
*v2.1.9* for [Jekyll](https://jekyllrb.com/) / [GitHub Pages](https://pages.github.com/).

## Copyright & License

Same licence as the one provided by Ghost's team. See Casper's theme [license](GHOST.txt).

Copyright (C) 2015-2018 - Released under the MIT License.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
