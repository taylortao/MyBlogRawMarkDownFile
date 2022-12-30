---
title: Set up your personal blog with hexo and github
date: 2013-04-20 18:37:25
categories:
 - IT Technology
tags:
 - Blog
---

### Install

Install git
Install nodes
Install Hexo
`$ npm install -g hexo-cli`

### Hexo
Create blog
`Mkdir blog`
`cd blog`
Run init: `hexo init`
 
Generate static blog files: `hexo generate`
Test and Debug local: `hexo server`
And then browse: http://localhost:4000

Add a new blog post: `hexo new "Hello Hexo”`

Or generate through github repository

```shell
cd .. && git clone https://github.com/taylortao/MyBlogRawMarkDownFile.git

cd blog/source
ln -sf ../../MyBlogRawMarkDownFile/about about
rm -rf _posts && ln -sf ../../MyBlogRawMarkDownFile/_posts _posts
ln -sf ../MyBlogRawMarkDownFile/hexoConfig/_config.yml _config.yml

// install jacman theme
git clone https://github.com/wuchong/jacman.git themes/jacman
cd themes/jacman && ln -sf ../../../MyBlogRawMarkDownFile/jacmanConfig/_config.yml _config.yml

```
 
Support feed and sitemap
`$ npm install hexo-generator-feed --save`
`$ npm install hexo-generator-sitemap --save`


### Set deploy destination to GitHub

<!-- more -->
 
Add a new repository，and its name must be `your_user_name.github.io`
 
And then modify _config.yml:
 
`vim _config.yml`

In deploy section


```
deploy:
     type: git
     repo:https://github.com/taylortao/taylortao.github.io.git
     branch:master
```

And then run
 
`$ npm install hexo-deployer-git --save`
 
`hexo deploy`
 
Browse: http://taylortao.github.io/
 
Regenerate:
`hexo clean`
`hexo generate`
`hexo deploy`