<ul>
{% assign sorted_pages = site.pages | sort: 'path' %}
{% for page in sorted_pages %}
  {% unless page.path contains '_' or page.path == 'index.md' %}
    {% assign clean_name = page.name | remove: '.md' | replace: '-', ' ' %}
    <li><a href="{{ page.url }}">{{ clean_name }}</a></li>
  {% endunless %}
{% endfor %}
</ul>



<ul>
{% for page in site.pages %}
  <li>{{ page.path }} - <a href="{{ page.url }}">{{ page.name | remove: '.md' | replace: '-', ' ' }}</a></li>
{% endfor %}
</ul>