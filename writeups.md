---
layout: default
title: "writeups"
---

## Writeups


<ul class="fa-ul">
    {% for write in site.writeups %}
    <li>
      <h2>
          <a href="{{ write.url }}" _target="blank">{{ write.title }}</a>
      </h2>
      <div class="tagline">
        {% for tag in write.tags%}
            <span class="shield shield-grey"><span><i class="fab fa-slack-hash"></i></span>{{ tag }}</span>
        {% endfor %}
        <span class="shield shield-blue"><span><i class="fas fa-calendar-alt"></i></span>{{ write.year }}</span>
      </div>
      <br><i class="fas fa-caret-right"></i> {{ write.description }}<br><br>
    </li>
  {% endfor %}
</ul>

{% if paginator.total_pages > 1 %}
<ul class="pager">
      {% if paginator.previous_page %}
      <li class="previous">
          <a href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}">&larr; Newer Posts</a>
      </li>
      {% endif %}
      {% if paginator.next_page %}
      <li class="next">
          <a href="{{ paginator.next_page_path | prepend: site.baseurl | replace: '//', '/' }}">Older Posts &rarr;</a>
      </li>
      {% endif %}
</ul>
{% endif %}


<br>


