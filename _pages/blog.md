---
layout: page
title: Blog
permalink: /blog/
---
<section class="c-archives">
  <ul class="c-archives__list">
  {% for post in site.posts  %}
    <li class="c-archives__item">
      <div class="c-archives__item-content">
        <h3>
          <a href="{{ post.url | prepend: site.baseurl }}">{{post.title}}</a>
          <small>{{ post.date | date: "%b %-d, %Y" }}</small>
        </h3>
        <p>{{ post.description }}</p>
      </div>
    </li>
  {% endfor %}
  </ul>
</section>
