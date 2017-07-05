---
layout: none
---
{% include doctype.html | xml_escape %}
{% assign post = site.posts.first %}
{{ post | replace:'<!DOCTYPE html>','' }}

test8