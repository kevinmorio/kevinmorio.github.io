---
layout: base
---

<div class="home">

  {% if site.paginate %}
    {% assign notes = paginator.notes %}
  {% else %}
    {% assign notes = site.notes %}
  {% endif %}


  {%- if notes.size > 0 -%}
    {%- if page.list_title -%}
      <h2 class="post-list-heading">{{ page.list_title }}</h2>
    {%- endif -%}
    {{ content }}
    <ul class="post-list">
      {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
      {%- for note in notes -%}
      <li>
        <span class="post-meta">{{ note.date | date: date_format }}</span>
        <h3>
          <a class="post-link" href="{{ note.url | relative_url }}">
            {{ note.title | escape }}
          </a>
        </h3>
        {%- if site.show_excerpts -%}
          {{ note.excerpt }}
        {%- endif -%}
      </li>
      {%- endfor -%}
    </ul>

    {% if site.paginate %}
      <div class="pager">
        <ul class="pagination">
        {%- if paginator.previous_page %}
          <li><a href="{{ paginator.previous_page_path | relative_url }}" class="previous-page">{{ paginator.previous_page }}</a></li>
        {%- else %}
          <li><div class="pager-edge">•</div></li>
        {%- endif %}
          <li><div class="current-page">{{ paginator.page }}</div></li>
        {%- if paginator.next_page %}
          <li><a href="{{ paginator.next_page_path | relative_url }}" class="next-page">{{ paginator.next_page }}</a></li>
        {%- else %}
          <li><div class="pager-edge">•</div></li>
        {%- endif %}
        </ul>
      </div>
    {%- endif %}

  {%- endif -%}

</div>

