---
layout: default
---

<div class="blog-index">  
  {% assign post = site.posts.first %}
  {% assign content = post.content %}
  {% assign page = site.posts.first %}
  {% include post_detail.md %}
</div>

hi {{ page.commentIssueId }}