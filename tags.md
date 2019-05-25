---
layout: page
permalink: /tags/
title: Tags
---
<h2>
    {% for tag in site.tags %}
    <a href="/tags/{{ tag[0] }}/" 
    style="font-size: {{ tag[1] | size | times: 2 | plus: 10 }}px;color:#C70039">
        {{ tag[0] }}
    </a>
    {% endfor %}
</h2>