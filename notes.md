---
title: Field Notes
layout: default
permalink: /notes/
---

<div class="page-banner">
    <div class="container">
        <a href="/" class="page-back-link">&larr; Back to Portfolio</a>
        <h1 class="page-banner-title">Field Notes</h1>
    </div>
</div>

<div class="container notes-index">
    {% if site.posts.size > 0 %}
    <ul class="notes-index-list">
        {% for post in site.posts %}
        <li class="notes-index-item">
            <time class="note-date" datetime="{{ post.date | date: '%Y-%m-%d' }}">{{ post.date | date: "%Y-%m-%d" }}</time>
            <a href="{{ post.url | relative_url }}" class="notes-index-link">{{ post.title }}</a>
            {% if post.tags and post.tags.size > 0 %}
            <div class="notes-index-tags">{% for tag in post.tags %}<span class="tag">{{ tag }}</span>{% endfor %}</div>
            {% endif %}
        </li>
        {% endfor %}
    </ul>
    {% else %}
    <p class="notes-empty">No field notes yet. Check back soon.</p>
    {% endif %}
</div>
