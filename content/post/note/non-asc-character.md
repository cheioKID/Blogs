---
date: 2018-09-14
title: "匹配非ASC字符"
tags:
    - note
categories:
    - Note
comment: true
---

复制的代码里面有非ASC II的字符，百度了一下，可以通过匹配正则表达式`[^\x00-\x7f]`来查找替换

```
[^\x00-\x7f]
```