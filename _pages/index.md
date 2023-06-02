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

{% assign post = site.posts.first %}
{% assign date_format = site.date_format | default: "%B %-d, %Y" %}

<h1 class="archive__item-title">Latest post from the blog</h1>
<div class="entries-{{ entries_layout }}">
  {% include archive-single.html type=entries_layout %}
  <div class="list__item">
    <p class="page__meta">
      <span class="page__meta-date">
        <time datetime="{{ date | date_to_xmlschema }}">{{ post.date | date: date_format }}</time>
      </span>
    </p>
  </div>
</div>

<hr>
<br />

<p class="small">
The idea behind this site is for me to try to share some knowledge occasionally, almost every day I solve complex issues and I'm probably what they would call a 'ghost-coder' or <a href="https://www.hanselman.com/blog/dark-matter-developers-the-unseen-99" target="_blank">Dark matter developer</a> :)

<br><br>

It means I get stuff done but very rarely writes about it unfortunately. Some of the reasons for this is that I almost exclusively work on company-related stuff that I can't share with the world, but the idea behind this site is to write more general tips and tricks.
</p>
