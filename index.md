---
layout: null
---
 
{% assign post = site.posts.first %}

{{ post | replace:'<!DOCTYPE html>','' }}