---
layout: default
---

<div class="blog-index">  
  {% assign post = site.posts.first %}
  {% assign content = post.content %}
  {% assign commentIssueId = post.commentIssueId %}
  {% include post_detail.md %}
</div>

hi {{ commentIssueId }}