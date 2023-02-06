---
layout: single
title: "EECS 504 - Computer Vision"
permalink: /coursework/EECS504/
comments: true
author_profile: true
---

This page contains my course work from EEECS 504 (Fall 2022)

- Programming language: Python
- Frameworks: Pytorch
- Topics:

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>