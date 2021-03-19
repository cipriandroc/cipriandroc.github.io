---
layout: page
title: powershell
permalink: /powershell/
---

        {% for pwsh in site.powershell %}
        {{ pwsh.name }}
        {{ pwsh.content}}
        {% endfor %}
