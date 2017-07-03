---
layout: null
---
 
{% assign post = site.posts.first %}

<!DOCTYPE html>

{{ post | replace:'<!DOCTYPE html>','' }}