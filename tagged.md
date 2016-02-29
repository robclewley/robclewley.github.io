---
layout: page
title: Blog Posts by Tag
---

<div>

{% for tag in site.data.tags %}
   <h3>{{ tag.name }}</h3>
   <ul>
    {% for post in site.posts %}
	  {% if post.tags.size > 0 %}
		  {% for ptag in post.tags %}
			  {% if ptag == tag.slug %}
				  <li> <a href="{{ post.url }}"> {{ post.title }}  - <small>{{ post.date | date_to_string }}</small>
				    </a></li>
			  {% endif %} <!-- ptag -->
		  {% endfor %} <!-- ptag -->
	  {% endif %} <!-- size -->
    {% endfor %} <!-- post -->
	</ul>
{% endfor %} <!-- tag -->

</div>
