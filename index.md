---
layout: none
---
{{ <!DOCTYPE html> }}
{% assign post = site.posts.first %}
{{ post | replace:'<!DOCTYPE html>','' }}

test8