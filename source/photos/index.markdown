---
layout: page
title: Photos Archive
footer: false
comments: false
---

<div id="blog-archives">
{% for album in site.albums reverse %}
{% capture this_year %}{{ album.date | date: "%Y" }}{% endcapture %}
{% unless year == this_year %}
  {% assign year = this_year %}
  <h2>{{ year }}</h2>
{% endunless %}
<article>
  {% include archive_post.html %}
</article>
{% endfor %}
</div>
