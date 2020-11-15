---
layout: books
title: Libros
---
<ul class="posts">
    {% for book in site.books %}
        <h2>{{ book.name }} - {{ book.event}}</h2>
    <p>{{ book.content | strip_html | truncate: 400 }}</p>
    {% endfor %}
</ul>
