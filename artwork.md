---
layout: default
title: Artwork
permalink: /artwork/
---

<div class="gallery-page">
  <div class="gallery-intro">
    <h1>{{ page.title }}</h1>
  </div>

  {% assign artwork_items = site.artwork | sort: "date" | reverse %}
  {% if artwork_items and artwork_items.size > 0 %}
    <ul class="card-gallery" role="list">
      {% for piece in artwork_items %}
        {% assign piece_alt = piece.alt | default: piece.title | default: "Magic card artwork" %}
        {% assign primary_image = nil %}
        {% assign image_alt = piece_alt %}
        {% assign piece_image_dir = piece.url | replace: '/artwork/', '' | replace: '/', '' %}
        {% assign piece_image_dir = '/images/artwork/' | append: piece_image_dir | append: '/' %}
        {% if piece.images and piece.images.size > 0 %}
          {% assign cover_image = piece.images.first %}
          {% for image in piece.images %}
            {% if image.cover %}
              {% assign cover_image = image %}
              {% break %}
            {% endif %}
          {% endfor %}
          {% assign primary_image = cover_image.src | default: cover_image %}
          {% unless primary_image contains '://' %}
            {% assign primary_image_lead = primary_image | slice: 0 %}
            {% unless primary_image_lead == '/' %}
              {% assign primary_image = piece_image_dir | append: primary_image %}
            {% endunless %}
          {% endunless %}
          {% assign image_alt = cover_image.alt | default: cover_image.caption | default: piece_alt %}
        {% endif %}
        <li class="card-gallery__item">
          <figure class="card-gallery__figure">
            {% if primary_image %}
              <a class="card-gallery__link" href="{{ piece.url | relative_url }}">
                <img
                  class="card-gallery__image"
                  src="{{ primary_image | relative_url }}"
                  alt="{{ image_alt | escape }}"
                  loading="lazy"
                  decoding="async"
                >
              </a>
            {% endif %}
            {% if piece.title or piece.caption %}
              <figcaption class="card-gallery__caption">
                {% if piece.title %}
                  <strong><a href="{{ piece.url | relative_url }}">{{ piece.title }}</a></strong>
                {% endif %}
                {% if piece.caption %}{{ piece.caption }}{% endif %}
              </figcaption>
            {% endif %}
          </figure>
        </li>
      {% endfor %}
    </ul>
  {% else %}
    <p class="gallery-empty">
      Add images to <code>/images/artwork/</code> and create one Markdown file per piece in <code>/_artwork/</code> to populate this gallery.
    </p>
  {% endif %}
</div>
