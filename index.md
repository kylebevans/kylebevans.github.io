---
layout: none
---
 
{% assign post = site.posts.first %}

<!DOCTYPE html>

{{ post | replace:'<!DOCTYPE html>','' }}