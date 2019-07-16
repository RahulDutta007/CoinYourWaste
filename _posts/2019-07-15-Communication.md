---
layout: post
title:  "Illustration of interaction with the CYW Smart Contract"
date:   2019-07-15 09:04:43 +0530
categories: jekyll update
---

{% for file in site.static_files %}
 	{% if file.image %}
 		<img src ="{{file.path}}" alt="{file.name}" width = "900" height="10000">
 	{% endif %}
{% endfor %}
