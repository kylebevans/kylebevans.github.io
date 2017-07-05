---
layout: null
---
 
{% assign post = site.posts.first %}

{% raw %}
<!DOCTYPE html>
{% endraw %}

{{ post | replace:'<!DOCTYPE html>','' }}