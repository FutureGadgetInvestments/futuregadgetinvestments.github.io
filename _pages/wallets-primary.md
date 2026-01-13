---
layout: page
title: Primary Wallet (Black)
permalink: /wallets/primary/
---

{% assign date_format = site.date_format | default: "%B %-d, %Y" %}

<div class="posts-list">
  {% for post in site.posts %}
  {% if post.path contains 'wallets/primary' %}
  <article class="post-preview">
    <a href="{{ post.url | relative_url }}">
      <h2 class="post-title">{{ post.title }}</h2>

      {% if post.subtitle %}
        <h3 class="post-subtitle">
        {{ post.subtitle }}
        </h3>
      {% endif %}
    </a>

    <p class="post-meta">
      Posted on {{ post.date | date: date_format }}
    </p>

    {% if post.excerpt %}
    <div class="post-entry">
      {{ post.excerpt | strip_html | xml_escape | truncatewords: site.excerpt_length }}
      {% assign excerpt_word_count = post.excerpt | number_of_words %}
      {% if post.content != post.excerpt or excerpt_word_count > site.excerpt_length %}
        <a href="{{ post.url | relative_url }}" class="post-read-more">[Read&nbsp;More]</a>
      {% endif %}
    </div>
    {% endif %}

   </article>
  {% endif %}
  {% endfor %}
</div>
