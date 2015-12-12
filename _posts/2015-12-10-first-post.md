---
title: How This Blog Was Set-Up
layout: post
author: Chris Zeeb
excerpt: An explaination of all the steps required to set this blog up at GitHub using Jekyll.
categories:
  - general
tags:
  - jekyll
  - cloudflare
  - github
  - disqus
  - googleanalytics
  
---
Setting up this blog has been a fun experience.  Even though for a living I manage the the day to day care and feeding of servers, the last thing I want do was manage yet another server.  A static site generator to generate the blog along with hosting on [GitHub Pages] was the perfect fit for me.  The really cool part for me was that I could manage a blog entirely from GitHub.  Automatic backups and revision control for blog posts built in!  Yes please.

Which static site generator to choose from?  Ultimately I liked both [Jekyll] and [Sculpin].  I almost went with [Sculpin] but ultimately it came down to the ease of deploying the blog to [GitHub Pages] with [Jekyll].

## Setup Steps

### Getting Started with Jekyll!

Pretty easy.  Just follow the [quick-start guide](http://jekyllrb.com/docs/quickstart/)!

### Domain Setup

Next up was the domain.  I didn't want to just use czeeb.github.io.  So I bought talkingtotheduck.ca and then followed [these instructions](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/).  I ended up using [CloudFlare] in front of the blog.  GitHub recommends using a CNAME with a subdomain for [GitHub Pages].  I saw no reason that blog.talkingtotheduck.ca was any worse than talkingtotheduck.ca.  [CloudFlare] allowed me to set up forwarding on the apex record by following [these](https://support.cloudflare.com/hc/en-us/articles/200172286-How-do-I-perform-URL-forwarding-or-redirects-with-CloudFlare-) instructions.

### Disqus

This setup was pretty easy.  The [javascript configuration variables](https://help.disqus.com/customer/en/portal/articles/472098-javascript-configuration-variables) KB Article at Disqus was helpful in understanding what they did and why they were important.

* Set up a [Disqus] account
* See [this commit](https://github.com/czeeb/czeeb.github.io/commit/faf154e22f52df1269572297e1cd2286b80e1b44) for changes to add [Disqus] to posts.  Make sure to update your `disqus_site_shortname` in `_config.yml`!  I elected not to set up categories in [Disqus] for now.

### Google Analytics

I set out to include [Google Analytics] only in production.  I didn't want to be tracking myself when working on blog posts or the blog itself.  Hidden away a little bit was the [repository metadata on GitHub Pages](https://help.github.com/articles/repository-metadata-on-github-pages/) KB article.  From there it was easy.  Just check if `site.github` is not nil.  If true, it's production!
 
* Set up a [Google Analytics] account and new property
* See [this commit](https://github.com/czeeb/czeeb.github.io/commit/8d36a14cdacba3cc26d42952bed61bd76043b88f) for changes made to this repository.  Don't forget to update your `googleanalytics_tracking_id` in `_config.yml` if you copy the commit! 

### SSL

Thankfully others had the same idea!  I found [easy to follow instructions](https://rck.ms/jekyll-github-pages-custom-domain-gandi-https-ssl-cloudflare/).  I also wanted to enforce SSL for users who may come to the blog over http.  [This post](https://konklone.com/post/github-pages-now-sorta-supports-https-so-use-it) by Eric Mill seems to be fairly widely adopted.  It is not a perfect solution but it does the trick to get users redirected from http to https.

You can view my [commit](https://github.com/czeeb/czeeb.github.io/commit/d3716e2c45830033d7d335ef27cfb8e052055064) for how it's implemented here.

It's not full end to end SSL, but I'm okay with that.  Nobody is transmitting passwords or other confidential information in requests to this blog.

## Conclusion

By the end I had a rudimentary blog up and running.  It's in revision control and backed up by pushing to GitHub.  Serving the blog is completely free with no server to manage.  I'm also future proofed a bit should I ever want to do more with the domain than just a blog.  Not bad for what amounted to just a few easy steps and some RTFM.  Of course there was some hacking involved set things up exactly how I wanted, but it was fun!

[GitHub Pages]: https://pages.github.com/
[jekyll]: https://jekyllrb.com/
[sculpin]: https://sculpin.io/
[CloudFlare]: https://www.cloudflare.com/
[Disqus]: https://disqus.com/
[Google Analytics]: https://www.google.com/analytics/