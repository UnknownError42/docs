---
layout: article
language: 'es-es'
version: '4.0'
title: 'Índice del API'
---
## Índice del API

{% for apiPage in site.pages %} {% if page.language == apiPage.language and page.version == apiPage.version %} {% assign stub = apiPage.name | slice: 0, 8 %} {% if "Phalcon_" == stub %} {% assign linkUrl = apiPage.name | replace: '.md', '' %} {% assign linkName = linkUrl | replace: '_', '\' | replace: '.md', '' %} * [{{ linkName }}](/{{ apiPage.version }}/{{ apiPage.language }}/api/{{ linkUrl }}) {% endif %} {% endif %} {% endfor %}