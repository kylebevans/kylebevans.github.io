---
layout: doctype
---

{% assign post = site.posts.first %}
{{ post | replace:'<!DOCTYPE html>','' }}