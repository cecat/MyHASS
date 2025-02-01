---
layout: default
#title: Adventures in Home Automation
---

# Blog Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}
    </li>
  {% endfor %}
</ul>


*Note: In the course of describing things I have built I often point to specific products, 
since they have worked for me.  That does not mean that they are the best (or only)
products, not that my source is the best place to buy them.  I receive no compensation
from the companies that make or sell these products.*
