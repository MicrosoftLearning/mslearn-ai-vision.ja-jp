---
title: Azure AI Vision の演習
permalink: index.html
layout: home
---

# Azure AI Vision の演習

次の演習は、Microsoft Learn のモジュールをサポートするように設計されています。


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %}

{% for activity in labs  %} {% if activity.lab.title contains "Azure AI Custom Vision" %}  
    {% continue %}  
  {% endif %} 
  - [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
