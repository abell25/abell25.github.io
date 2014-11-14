---
layout: post
title: "New website setup"
date: 2014-11-13 22:37:38 -0600
comments: true
categories: 
---

Hi.  I have just set up my new blog.  My original blog was hosted on HostMonster with a php/html custom coded solution.  It was not very pretty so I decided to try something new.  I ended up going with octopress, which is a static blogging framework build on jekyl, which is built on ruby on rails.  It is currently hosted from a s3 bucket and distributed using cloudfront.  So basically the following happens when I want to write a new post:

* run `rake new_post["my post title"]`
* write a post in markdown
* run `rake generate` to physically build the whole site on my computer.
* run `rake deploy` to deploy the site to the s3 bucket.
* The updated bucket changes get pushed via cloudfront to distribution centers around the world

This is a pretty nice framework so far.  You only have to set your information in a config file to get add-ons such as comments, a list of your github repositories, social media 'like/tweet/+1' buttons.  You also have configuration options on a per-post so you can disable comments on a specific post by setting `comments: true` to `false`.  

I also like that the site will have very strong security since I don't actually have any server-side code running.  Security is something I was very cavalier about when I first started coding but now I think about more often.  Since your website is built from scratch on a local machine and deployed to servers as static content it is virtually impossible to hack it.
