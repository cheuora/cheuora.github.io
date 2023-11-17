---
layout: default
title: Tag
permalink: /tagarchive/
---


<h2>Archive</h2>
{% capture temptags %}
  {% for tag in site.tags %}
    {{ tag[1].size | plus: 1000 }}#{{ tag[0] | replace: ' ', '_' }}#{{ tag[1].size }}
  {% endfor %}
{% endcapture %}
{% assign sortedtemptags = temptags | split:' ' | sort | reverse %}
<div>
{% for temptag in sortedtemptags %}
  {% assign tagitems = temptag | split: '#' %}
  {% capture tagname %}{{ tagitems[1] }}{% endcapture %}
  {% unless tagname == '1' %}
  <a href="/tag/{{ tagname }}"><code class="highligher-rouge"><nobr>{{ tagname }}</nobr></code></a>
  {% endunless%}
{% endfor %}
</div>