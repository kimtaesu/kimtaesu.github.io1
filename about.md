---
layout: default
title: about
permalink: /about/
---

# about
<br/>
<div class="contacticon center">
    <!-- Google Authorship Markup -->
    <link rel="author" href="https://plus.google.com/+{{site.gplus_username}}?rel=author">

    <!-- Social: Twitter -->
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:site" content="@{{site.twitter_username}}">
    <meta name="twitter:title" content="{% if page.title %}{{ page.title }}{% else %}{{ site.title }}{% endif %}">
    <meta name="twitter:description" content="{% if page.description %}{{ page.description | strip_html | strip_newlines | truncate: 160 }}{% else %}{{ site.description }}{% endif %}">
    {% if page.image %}
    <meta property="twitter:image:src" content="{{ site.url }}{{page.image }}">
    {% else %}
    <meta property="twitter:image:src" content="{{ "/assets/img/blog-image.png" | prepend: site.baseurl | prepend: site.url }}">
    {% endif %}

    <!-- Social: Facebook / Open Graph -->
    <meta property="og:url" content="{{ page.url | replace:'index.html','' | prepend: site.baseurl | prepend: site.url }}">
    <meta property="og:title" content="{% if page.title %}{{ page.title }}{% else %}{{ site.title }}{% endif %}">
    {% if page.image %}
    <meta property="og:image" content="{{ site.url }}{{page.image }}">
    {% else %}
    <meta property="og:image" content="{{ "/assets/img/blog-image.png" | prepend: site.baseurl | prepend: site.url }}">
    {% endif %}
    <meta property="og:description" content="{% if page.description %}{{ page.description | strip_html | strip_newlines | truncate: 160 }}{% else %}{{ site.description }}{% endif %}">
    <meta property="og:site_name" content="{{ site.title }}">

    <!-- Social: Google+ / Schema.org  -->
    <meta itemprop="name" content="{% if page.title %}{{ page.title }}{% else %}{{ site.title }}{% endif %}"/>
    <meta itemprop="description" content="{% if page.description %}{{ page.description | strip_html | strip_newlines | truncate: 160 }}{% else %}{{ site.description }}{% endif %}">
    <meta itemprop="image" content="{{ "/assets/img/blog-image.png" | prepend: site.baseurl | prepend: site.url }}"/>

    <div class="icons-home">
        <a aria-label="Send email" href="mailto:{{site.email}}"><svg class="icon icon-email"><use xlink:href="#icon-email"></use></svg></a>
        
        <a aria-label="My Linkedin" target="_blank" href="https://www.linkedin.com/{{site.linkedin_username}}"><svg class="icon icon-linkedin"><use xlink:href="#icon-linkedin"></use></svg></a>
        
        <a aria-label="My Twitter" target="_blank" href="https://twitter.com/{{site.twitter_username}}"><svg class="icon icon-twitter"><use xlink:href="#icon-twitter"></use></svg></a>
        <a aria-label="My Google Plus" target="_blank" href="https://plus.google.com/+{{site.gplus_username}}/posts"><svg class="icon icon-google-plus"><use xlink:href="#icon-google-plus"></use></svg></a>
        <a aria-label="My Github" target="_blank" href="https://github.com/{{site.github_username}}"><svg class="icon icon-github-alt"><use xlink:href="#icon-github-alt"></use></svg></a>
        <a aria-label="Use the RSS to get updated" target="_blank" href="{{ "/feed.xml" | prepend: site.baseurl }}"><svg class="icon icon-rss"><use xlink:href="#icon-rss"></use></svg></a>
    </div>
</div>

<div class="col three caption">
I am best reachable by email: haanjack (at) gmail (dot) com
</div>

<br/>
<hr/>

<br/>

어렸을 적 부터 컴퓨터를 좋아해서 컴퓨터 관련 분야에 종사하고 있습니다.

임베디드 프로그래밍을 시작으로 해서, 병렬처리를 거쳐 지금은 머신러닝 분야로 이해의 범위를 넓혀가고 있습니다.

먼저는 의료 분야에서 할 수 있는 것을 찾았으나, 프로그래밍을 잘 하는 것 만으로는 외연을 확장시킬 수가 없어서 다른 방법을 찾고 있습니다.

자전거, 사진, 여행을 좋아하고, 제 아내를 사랑하며 아직 지구에 도착하지 않은 아이를 기다리고 있습니다.

<br/>

<h1>Foot Notes</h1>
<br/>

* 현대중공업 - 수술용 로봇 개발
* 삼성메디슨 - 초음파 기기 제어 및 영상처리 모듈 개발

<br/>
<br/>

<h1>Learning</h1>
<br/>

* [Coursera](http://www.coursera.org)
  * Machine Learning
  * Programming for Everybody (Getting Started with Python)
  * 
* Udacity
  * Deep Learning
  * Introduction to Parallel Programming
  * Rapid Prototyping
  * Machine Learning for Trading
* Udemy
* edX
* Linda

* 서울대학교 컴퓨터공학과 통합설계 및 병렬처리연구실
* 한양대학교 정보통신대학 미디어통신공학과


<!--
<div class="tags">
{% assign tags_list = site.tags %}
  {% if tags_list.first[0] == null %}
    {% for tag in tags_list %}
        <a data-scroll href="#{{ tag | slugify }}">{{ tag }}</a>
    {% endfor %}
  {% else %}
    {% for tag in tags_list %}
        <a data-scroll href="#{{ tag[0] | slugify }}">{{ tag[0] }}</a>
    {% endfor %}
  {% endif %}
{% assign tags_list = nil %}
</div>
-->

<br/>

<hr/>
<br/>
<img class="col one center img-rounded" src="/img/blog-author.jpg">



