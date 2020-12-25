---
layout: page
title: Blog
permalink: /blog/
---
<section class="c-archives">
  <ul class="c-archives__list">
  {% for post in site.posts  %}
    <li class="c-archives__item">
      <h3>
        <a href="{{ post.url | prepend: site.baseurl }}">{{post.title}}</a>
        <br>
        <small>{{post.description}}</small>
      </h3>
      <p>{{ post.date | date: "%b %-d, %Y" }}</p>
    </li>
  {% endfor %}
  </ul>
</section>
