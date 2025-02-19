---
title: "Flockenberger's Website"
layout: archive

excerpt: ""

header:
image: assets/images/flockenberger.jpg
author_profile: false

---
<p>
Welcome to my page :)
</p>
<p>
Enjoy your stay!
</p>
{{ content }}

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

{% include paginator.html %}