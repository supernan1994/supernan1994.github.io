---
layout: archive
permalink: /architect/
title: "Latest Posts"
image:
    feature: cover.jpeg
    credit: tatiana
    creditlink: https://www.pexels.com/photo/adventure-asphalt-autumn-calmness-614484/
---

<div class="tiles">
{% for post in site.categories.architect %}
	{% include post-grid.html %}
{% endfor %}
</div><!-- /.tiles -->
