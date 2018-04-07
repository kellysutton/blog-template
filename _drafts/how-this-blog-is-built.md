---
layout: post
title: "How This Blog is Built"
date: 2018-04-07T07:41:00-07:00
excerpt: What’s the tech behind this blog?
draft: true
---

I usually try to avoid navel-gazing too much, but a few folks have asked me how this blog is built and maintained. This also would be a good time to talk about the _why_.

This blog post covers the Principles, Technology, and Process behind this blog.

## Principles

After giving it a thought, I apply 3 principles to decisions around how this blog is built and maintained.

0. Make reading posts a great experience.
0. Make writing posts frictionless.
0. Don’t sharecrop.

Let’s dive into each briefly before getting to the technical setup. Without the above, it will be difficult to discuss the technical setup.

### 1. Make reading posts a great experience

Many sites looking to turn a profit need to extract something from the user. They need some impressions on ads or signups to a newsletter. Thankfully, this blog does not need to turn a profit.

As such, the experience can be kept pure. Ads aren't required and getting signups on [the newsletter](https://buttondown.email/kellysutton) is a nice-to-have but not a requirement for future employment.

To make the experience of reading great, I try to use the simplest technoligies possible and remove as much as possible. It’s my belief that the writing should speak for itself. A better design will not make the content better.

### 2. Make writing posts frictionless

It’s hard enough to keep a regular cadence of writing. Writing is much like product development: you can guess what might be popular, but you’ll never really know. As you write more, you develop a better muscle. Still, you can’t predict the future.

It’s for this reason that I keep writing posts as frictionless as possible. Writing is one of those things where biasing toward more quantity can also improve the quality.

### 3. Don’t sharecrop

This blog used to be on Tumblr. Before that, it might have been on Blogspot or Blogger. Today, it could be on Medium.

I’ve gotten to the point where I don’t write on these other platforms. It’s not that they aren’t nice, it’s that they often conflict with the above 2 principles. Things usually start out well enough, but those companies have a business to run. Many times, the needs of the business run counter to my interests.

I’ve recently begun cross-publishing a few posts on Medium, but other platforms aren’t the first places things get published.

## Technology

Alright, enough of the hand-wavy. Let’s get down to it. This blog uses the following technology:

- This blog is written in [Markdown](https://en.wikipedia.org/wiki/Markdown),
- compiled using [Jekyll](https://jekyllrb.com/),
- hosted on [Amazon S3](https://aws.amazon.com/s3/),
- deployed using [s3_website](https://github.com/laurilehmijoki/s3_website),
- accelerated by [Amazon Cloudfront](https://aws.amazon.com/cloudfront/),
- with fonts from [Google Fonts](https://fonts.google.com/),
- with images accelerated and modified by [imgix](https://www.imgix.com/),
- with a newsletter managed by [Buttondown](https://buttondown.email/),
- all tied together with some simple boilerplate.

### Markdown

I find [Markdown](https://en.wikipedia.org/wiki/Markdown) the most ergonomic for me. Sometimes I’ll drop into HTML to accomplish something, usually around images and adding captions.

### Jekyll

[Jekyll](https://jekyllrb.com/) is a Ruby-based static site generator. It gives you some of the convenience of other CMS systems (reusable partials and layours) while providing a bare bondes simplicity. Not many things are faster than serving flat files.

For how I have Jekyll configured, please take a look at the boilerplate section.

### Amazon S3

[Amazon S3](https://aws.amazon.com/s3/) is the workhorse of the web. Because Jekyll crunches your site into static files, they just need a place to live with no extra logic. S3 works well for that, but so could any other blob storage system.

### Amazon Cloudfront

While S3 is the workhorse, it’s not particularly designed for speed. [Amazon Cloudfront](https://aws.amazon.com/cloudfront/) is the “good enough” CDN for this site. It works well with S3 and provides some nice things like automatic certificate rotation for making HTTPS easy to maintain and now also has limited HTTP/2 support.

As a footnote, I’m also using [Route 53](https://aws.amazon.com/route53/) for DNS.

### Google Fonts

My design eye often needs some work. For this blog I use [Montserrat](https://fonts.google.com/specimen/Montserrat) for headlines and [Crimson Text](https://fonts.google.com/specimen/Crimson+Text) for body text. Both of these are served from [Google Fonts](https://fonts.google.com).

If you have any suggested tweaks or have a better pairing in mind, [please let me know](mailto:michael.k.sutton@gmail.com).

### imgix

I use [imgix](https://www.imgix.com/) for making sure the images are the best size for the device they are being served to. This is done by using `srcset` and `sizes` attributes for each image I embed on the blog. I wrote about the approach in the post [Boiling the Ocean with Markup](https://kellysutton.com/2016/06/23/boiling-the-ocean-with-markup.html).

Any diagrams I create are usually done with [OmniGraffle](https://www.omnigroup.com/omnigraffle). Diagrams are exported at a high resolution and then crunched down as necessary by imgix.

### Buttondown

I used to use Tinyletter, but now I use [Buttondown](https://buttondown.email/). The experience is much better.

### Boilerplate

Now for the things that tie all of this together. I’ve got 2 scripts that help me to write and publish: `bin/serve` and `bin/deploy`. These are simple shell scripts that help me not remember the Jekyll incantations required to do what I want to do.

I’ve modified the stock Jekyll layouts a bit to handle drafting functionality. Blog posts are better when they’ve been read by a few people first, so drafting is a core part of my workflow.

imgix doesn’t quite have everything I needed in their [jekyll-imgix](https://github.com/imgix/jekyll-imgix) so I’ve forked it.

If you want to see all of this in action, please take a look at the [GitHub repo](https://github.com/kellysutton/blog-template).

## Process

I keep a running list of potential blog posts that I want to write. The list is sorted and reordered depending on how I’m feeling, how past posts perform, and what the current weather is.

Generally, the process of writing a post is the following:

0. Pull off a post from the backlog.
0. Write a draft of the post.
0. Take an editing pass at the post.
0. Circulate the draft for feedback by soft publishing the post. (You can only get to the draft with a URL, it’s not listed on the front page.)
0. Collect and incorporate feedback.
0. Publish the post.
0. Mention the post on Twitter. Maybe also LinkedIn and Facebook depending on the content.
0. Send out a newsletter mentioning the post

All in, each post takes somewhere between 4-8 hours of work to move through this process. As with any creative endeavor, there are false starts. Just because something makes it to a certain step doesn’t guarantee the following steps will occur.

## Conclusion

Hopefully this is useful for you as you write your own blog. I’m happy to answer questions about this set up on [Twitter](https://twitter.com/kellysutton) or [via email](mailto:michael.k.sutton@gmail.com).

Again, check out the [GitHub repo](https://github.com/kellysutton/blog-template) if you want to see how this is set up from a technical perspective.

---

Special thanks to X for providing feedback on early drafts of this post.

{% include newsletter.html %}
