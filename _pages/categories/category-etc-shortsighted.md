---
title: "생각이 나서~"
layout: archive
permalink: /categories/shortsighted
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.ShortSighted %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}