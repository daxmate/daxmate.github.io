---
title: "归档"
permalink: /archive/
layout: archive
author_profile: true
---

{% assign entries_layout = page.entries_layout | default: 'list' %}

{% assign postsByYear = site.posts | group_by_exp: 'post', 'post.date | date: "%Y"' %}
<ul class="taxonomy__index">
  {% for year in postsByYear %}
    <li>
      <a href="#{{ year.name }}">
        <strong>{{ year.name }}</strong> <span class="taxonomy__count">{{ year.items | size }}</span>
      </a>
    </li>
  {% endfor %}
</ul>

{% for year in postsByYear %}
  <section id="{{ year.name }}" class="taxonomy__section">
    <h2 class="archive__subtitle">{{ year.name }}</h2>
    <div class="entries-{{ entries_layout }}">
      {% for post in year.items %}
        {% include archive-single.html locale=page.locale type=entries_layout %}
      {% endfor %}
    </div>
    <a href="#page-title" class="back-to-top">{{ site.data.ui-text[page.locale].back_to_top | default: 'Back to Top' }} &uarr;</a>
  </section>
{% endfor %}
