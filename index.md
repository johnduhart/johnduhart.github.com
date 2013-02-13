---
layout: default
---

<div class="text">
    <h2>Posts</h2>
    <ul>
    {% for post in site.posts  %}
      <li><a href="{{site.baseurl}}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
    </ul>
</div>