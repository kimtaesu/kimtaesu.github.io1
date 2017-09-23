---
layout: page
permalink: /android
title: Android

---

<ul class="post-list">
{% for post in site.android reversed %}
    <li>
      <h2><a class="post-title" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></h2>
      <p class="post-meta">{{ post.description }}</p>
      <p class="post-meta">{{ post.date | date: "%B %-d, %Y" }}</p>
      </li>
{% endfor %}
</ul>