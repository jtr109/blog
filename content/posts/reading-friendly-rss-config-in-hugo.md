---
title: "Reading Friendly RSS Config in Hugo"
date: 2020-06-14T10:41:57+08:00
typora-root-url: ../../static
categories:
  - tech
series:
  - Blog Notes
tags:
  - Hugo
  - blog
  - RSS
---

## Background

[Hugo](https://gohugo.io/) has RSS support by default. But the implementation is not friendly for reading in RSS subscriber. If we open [the RSS page](/index.xml) we will see that the content in `<description>` are summary with plaintext.

![summary-without-html-tags](/images/reading-friendly-rss-config-in-hugo.assets/image-20200614105452252.png)

## Solution

If we want to show *full content* in our RSS pages, we need to override [the RSS template](https://gohugo.io/templates/rss/#the-embedded-rssxml). Copy [the default rss template](https://github.com/gohugoio/hugo/blob/master/tpl/tplimpl/embedded/templates/_default/rss.xml) and save it in the local file `layouts/_default/rss.xml`. Then, find out the [`.Summary` in `<description>`](https://github.com/gohugoio/hugo/blob/master/tpl/tplimpl/embedded/templates/_default/rss.xml#L35) and replace it into `.Content`. [The RSS page](/index.xml) shows full content in description with HTML style.

![image-20200614110430952](/images/reading-friendly-rss-config-in-hugo.assets/image-20200614110430952.png)

## References

* https://yongfu.name/2018/12/13/hugo_rss.html