---
layout: page
title: Archive
subtitle: Every entry, oldest month last.
permalink: /archive/
---

{% assign months = site.posts | group_by_exp: "post", "post.date | date: '%Y-%m'" %}
{% for month in months %}
<section class="archive-month">
  <h2 class="archive-month__title">{{ month.items.first.date | date: "%B %Y" }}</h2>
  <ol class="archive-list">
    {%- for post in month.items %}
    <li class="archive-list__item">
      {%- include day-count.html date=post.date day=post.day format="inline" -%}
      <a class="archive-list__link" href="{{ post.url | relative_url }}">{{ post.title | smartify }}</a>
      <time class="archive-list__date" datetime="{{ post.date | date_to_xmlschema }}">{%- include ap-date.html date=post.date -%}</time>
    </li>
    {%- endfor %}
  </ol>
</section>
{% else %}
<p>No entries yet.</p>
{% endfor %}
