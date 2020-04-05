---
layout: sessions
title: Sesiones
---
<ul class="posts">
    {% for session in site.sessions %}
        <h2>{{ session.name }} - {{ session.event}}</h2>
    <p>{{ session.content | strip_html | truncate: 400 }}</p>
    {% endfor %}
</ul>
