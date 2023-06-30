---
layout: splash
permalink: /
hidden: true
header:
  overlay_color: "#434654"
excerpt: >
  Developer blog about anything developer related.<br />
feature_row:
  - alt: "Latest posts"
    title: "Latest posts"
    excerpt: "View my most recent posts here"
    url: "/blog"
    btn_class: "btn--primary"
    btn_label: "Visit blog"
  - alt: "Here goes latest post"
    title: "Here goes latest post"
    excerpt: "The latest post except"
    url: "/blog/"
    btn_class: "btn--primary"
    btn_label: "Read more"
  - alt: "About me"
    title: "About me"
    excerpt: "Just short about me"
    url: "/about"
    btn_class: "btn--primary"
    btn_label: "Read more"
---
<!-- {% include feature_row %} -->

{% assign date_format = site.date_format | default: "%B %-d, %Y" %}

<h1>Latest from the blog</h1>
<div class="entries-{{ entries_layout }}">
  {% for post in site.posts limit:3 %}
    {% include archive-single.html type=entries_layout %}
    <div class="list__item">
      <p class="page__meta">
        <span class="page__meta-date">
          <time datetime="{{ date | date_to_xmlschema }}">{{ post.date | date: date_format }}</time>
        </span>
      </p>
    </div>
  {% endfor %}
</div>
