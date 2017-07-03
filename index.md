---
layout: null
---
 
{% assign post = site.posts.first %}

<!DOCTYPE html>
hi

{{ post | replace:'<!DOCTYPE html>','' }}