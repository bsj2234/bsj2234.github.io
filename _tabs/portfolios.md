---
layout: page
title: Portfolios
icon: fas fa-briefcase
order: 4
---

<div class="portfolio-grid">
  {% assign sorted_portfolios = site.portfolios | sort: "order" %}
  {% for portfolio in sorted_portfolios %}
    <div class="portfolio-item">
      {% if portfolio.image %}
        <div class="portfolio-image">
          <img src="{{ portfolio.image }}" alt="{{ portfolio.title }}">
        </div>
      {% endif %}
      <div class="portfolio-content">
        <a href="{{ portfolio.url | relative_url }}">
          <h2>{{ portfolio.title }}</h2>
          <p>{{ portfolio.description }}</p>
        </a>
      </div>
    </div>
  {% endfor %}
</div>
