---
layout: default
title: Blog archive
---
<div class="page-content wc-container">
  <h1>Blog Archive</h1>  
  {% for post in site.posts %}
  	{% capture currentyear %}{{post.date | date: "%Y"}}{% endcapture %}
	ABC
  	{% if currentyear != year %}
		123
    	{% unless forloop.first %}</ul>{% endunless %}
			ABC123
			<h5>{{ currentyear }}</h5>
    		<ul class="posts">
    		{% capture year %}{{currentyear}}{% endcapture %} 
  		{% endif %}
    <li><a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a></li>
{% endfor %}
</div>