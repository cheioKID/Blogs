---
date: 2018-03-25
title: "Deploy Hugo on github.io"
tags:
    - github
    - note
categories:
    - note
comment: true
---

Finally it works!

Here's the steps


### generate new blogs
```shell
hugo new site blogs
cd blogs
git init
git remote add origin git@github.com:cheioKID/Blogs.git
```


### change the baseURL to github.io
```toml
baseurl = "https://cheioKID.github.io/"
```


### apple the theme
```shell
git submodule add -b master https://github.com/xianmin/hugo-theme-jane.git themes/jane
cp -r themes/jane/exampleSite/content ./
cp themes/jane/exampleSite/config.toml ./
```


### add markdown files
with head like this
```yaml
---
title: "For example"
date: 2018-3-25
#lastmod: 2017-08-31T15:43:48+08:00
draft: false
tags: ["preview", "English", "tag-2"]
categories: ["English", "index"]
author: "cheio"
---
```
or

```yaml
---
date: 2018-03-22
title: "example"
tags:
    - hello
    - goodbye
categories:
    - foo
comment: true
---
```


### push to remote
```shell
git add .
git commit -m "hello"
git push origin master
```


### add submodule
```shell
git submodule add -f -b master https://github.com/cheioKID/cheioKID.github.io.git public
git add .
git commit -m "Initial commit"
git push -u origin master
```

if `'public' already exists and is not a valid git repo` is arised, try `rm -rf public` and `git rm -r --cached public/`


### push github.io
```shell
hugo --cleanDestinationDir
cd public
git add .
git commit -m "Build website"
git push origin master
cd ..
```

