---
layout: single
title: "EECS 504 - Computer Vision"
permalink: /coursework/EECS504/
comments: true
author_profile: true
---

This page contains my course work from EEECS 504 (Fall 2022)

- Programming language: Python
- Frameworks: Pytorch, SciPy
- Topics: 

## Homeworks -

<ul>
  {% for post in site.categories.EECS504 %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>