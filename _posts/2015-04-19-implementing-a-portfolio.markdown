---
layout: post
title: Implementing a portfolio
---

Developers should showcase their work, and a portfolio is probably the best way to do it. A portfolio should be most possibly up-to-date and provide insights into one's personality.

Until now, my blog was way too blog-centric for
the frequency with which I wrote posts. Which was about once a month.

> Most developer pages are way too blog-centric

A blog may make sense if you are a technical writer and have a topic that
you attend to with regularity. In my case that just does not fit.
I want people to gain insights into my life, my personality and what I am capable of.
I want them to be able to catch a glimpse of my personality.

A lot of research has shown that there are many blogging platforms out there,
but virtually none for combining writing with a personal touch.
There is [about.me](http://about.me), which allows you to create something
similar to a business card on the web. Take a look, they are about as
unique and personalized a profile picture on LinkedIn (plus the theme is not flat...).
Just do a google search like ['about.me felix'](https://www.google.de/?q=about.me+felix)

Then there are tools like Ghost and Wordpress. CRMs tuned to blogging.
What is sad about them that they offer very little options as to custom fields
and their values.

I tried to provide a developer portfolio theme
for Ghost ([developium](https://github.com/flxw/developium)), but I noticed
that anyone using this theme would not come around editing source code.
So there was virtually no advantage of using the Ghost administration UI
and no reason to put up with the overhead of the system.

That is why I reverted to the use of [harp.js](https://harpjs.com), a static site generator.
The landing page is what most people will see, so it should capture most
facets of me.

![](/posts/img/landing-page-redesign.png)

Until now, three different tiles have been implemented:

- A Github tile which shows my most recent activity on that network
- What I am currently doing so that people will get a general impression of what my days look like
- My latest post. The others can be listed by clicking the correct link

The heading is designed to be quite dominant to remind people that this page
is primarily a representation of me and that its content should be associated with me.

More development and information about the proceedings will follow in due time.
Eventually I might move this blog over to github pages and open source the templating code :)
