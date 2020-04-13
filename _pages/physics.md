---
classes: wide

layout : single
permalink: /physics/
title: "Physics"
author_profile: true
header:
 #image: "/images/12.png"

---




{% for tag in group_names %}
  {% assign posts = group_items[forloop.index1] %}
  <h2 id="{{ tag | slugify }}" class="archive__subtitle">{{ tag }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}
