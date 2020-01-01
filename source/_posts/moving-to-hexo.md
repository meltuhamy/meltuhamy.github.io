---
title: Moving to hexo
date: 2016-02-07 21:48:09
thumbnail: /images/hexo-logo.svg
tags:
  - hexo
  - wordpress
  - blog
  - markdown
categories:
  - tech
---

Today I migrated my blog from Wordpress to [Hexo](https://hexo.io/).
Hexo is a statically generated blogging framework, meaning you write posts locally using [Markdown](https://en.wikipedia.org/wiki/Markdown).

## Why move from WordPress?

![WordPress. You did well, but you're not fast enough](images/wp-logo.svg)

WordPress is great for managing a blog and is a fully-fledged all in one solution. It's also the most popular blogging platform, with many sites on the web using it to manage blogs and content. So why move away from something that's powerful and trusted?

The answer for me is the combination of *price* and *performance*. Put simply, WordPress gives you loads of features and power, but at the price of performance. If you want your website and blog to scale, you'll have to spend money. The more features and plugins you add, the slower your site will get. If you want to speed it up you'll have to invest in some better servers.

My original set up was that I've got a very simple blog that doesn't use a plethora of plugins. Even with my simple site which has heavy caching on content and CDN-hosted images and resources, it is still noticably slow. It was hosted by [TSO Host](http://my.tsohost.com/aff.php?aff=2735), which gave me all I needed and more - but the main drawback was performance. It simply wasn't instant.

There was two options in front of me:

1. Improve performance by going for a cloud based hosting solution with CDN

2. Greatly simplify my blog by switching to a statically generated site.

I investigated option option 1: a cloud based solution. I found a [great website](http://publishingwithwordpress.com/estimating-costs-wordpress-amazon-aws/) that describes what's needed and how much it is. The conclusion was that it would cost 10-20 USD and a great deal of hassle setting everything up. Sure, I have experience with linux and LAMP - but even then, I'm managing my blog in my spare time and don't want to spend my weekends fixing nginx server issues. And with that, I'm paying money to get this done. For me, the hassle was not worth it.

I started to look into option 2: a statically generated blog. This means that I write my blog locally, save it, pass it through a generator with a templating engine, and publish the result. This means I can't just blog from anywhere. I have to use a computer, save files, commit them to git, run a program and push the changes. Luckily, I'm used to this. In fact I used a Markdown plugin for WordPress that allowed me to write posts using Markdown.

Because it's statically generated, the speed was good. There is no server-side processing involved whatsoever.

Because it's statically generated, the price was good. I'm able to host my blog _free_ on GitHub. I just need to pay for my domain.

I got what I was looking for. Speed and performance

## The move to hexo
Next, I looked for what platform to use. There are [loads](https://www.staticgen.com/) to pick from.

![Hexo. You're written in JavaScript. I chose you.](/images/hexo-logo.svg)

I decided to use hexo because... well, I just liked that it was written in JavaScript and I found a cool theme :) I agree, not really an engineering approach to picking a platform but who cares, this has what I need. I figured if I got bored I can easily switch.

Luckily, hexo had an official [WordPress importer](https://hexo.io/docs/migration.html). So moving my blog was a few clicks and terminal commands away.

Once imported, I tweaked a few posts, copied my images, set up comments using [Disqus](https://disqus.com/), set up Google Analytics, etc. These are all options in a .yaml file. It's that easy.

Once I finished with my blog locally, I created a repo for it on [GitHub](https://pages.github.com/) and followed instructions to get it [working with my own domain name](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/).

My experience over all was pretty sweet and straight forward. Hooray!

## The price and performance
The price saving is easy to work out. I'm saving more than £5 a month.

In a later blog post, we'll analyse performance gains.