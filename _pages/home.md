---
layout: splash
permalink: /
exceprt: ""
header:
  image: /assets/images/flockenberger.png
author_profile: false
---

# Welcome!
This is my humble little web-page where I'll post and write about all the little things I do.


Enjoy your Stay!


<h3 class="archive__subtitle">{{ site.data.ui-text[site.locale].recent_posts | default: "Recent Posts" }}</h3>

{% if paginator %}
  {% assign posts = paginator.posts %}
{% else %}
  {% assign posts = site.posts %}
{% endif %}

{% assign entries_layout = page.entries_layout | default: 'list' %}
<div class="entries-{{ entries_layout }}">
  {% for post in posts %}
    {% include archive-single.html type=entries_layout %}
  {% endfor %}
</div>