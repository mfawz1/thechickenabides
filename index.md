---
title: The Chicken Abides
---

# Tech Blog?



<ul>
  {% for post in site.posts %}
    <li>
      <a href="/thechickenabides{{ post.url }}">{{ post.title }} [{{ post.date | date: "%a, %b %d, %Y"}}]</a>
    </li>
  {% endfor %}
</ul>

