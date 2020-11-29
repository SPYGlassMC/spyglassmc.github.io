---
layout: page
title: Wiki
permalink: /wiki/
---

{% capture home %}{% include_relative _wiki/Home.md %}{% endcapture %}
{% capture sidebar %}{% include_relative _wiki/_Sidebar.md %}{% endcapture %}

{{ home | markdownify }}

{{ sidebar | markdownify }}

<!-- <ul class="wikiMenu">
	{% for p in site.pages %}
		{% if p.menu == "wiki" %}
		<li><a class="post-link" href="{{ p.url | prepend: site.baseurl }}">{{ p.title }}</a></li>
		{% endif %}
	{% endfor %}
</ul> -->
