---
title: Azure AI Vision の演習
permalink: index.html
layout: home
---

# Azure AI Vision の演習

次の演習は、Microsoft Learn のモジュールをサポートするように設計されています。


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %}

{% for activity in labs  %} {% if activity.url contains 'ai-foundry' %} {% continue %} {% endif %}
  - [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
