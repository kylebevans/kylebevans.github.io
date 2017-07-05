---
layout: none
---
{% include doctype.html %}
{% assign post = site.posts.first %}
{{ post | replace:'<!DOCTYPE html>','' }}

test6