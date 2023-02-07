---
layout: single
title: "EECS 504 - Computer Vision"
permalink: /coursework/EECS504/
comments: true
author_profile: true

feature_row:
  - image_path: /assets/images/coursework/EECS504/hw1/hw1_upload_6_14.png
    title: "1. Pet Edge Detection"
    excerpt: "Topic: Edge detection, Convolution"
    url: "/coursework/EECS504/hw1/"
    btn_label: "Read More"
    btn_class: "btn--primary"	
---

This page contains my course work from EEECS 504 (Fall 2022)

- Programming language: Python
- Frameworks: Pytorch, SciPy
- Topics: 

## Homeworks -

<ul>
  {% for post in site.categories.EECS504 %}
    <li>
      <a href="{{ post.url }}"><img src = "{{ pst.thumbnail }}" />
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>

 {% include feature_row %}