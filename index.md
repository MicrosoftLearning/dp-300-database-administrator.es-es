---
title: Instrucciones hospedadas en línea
permalink: index.html
layout: home
---

# Ejercicios de administración de bases de datos

Estos ejercicios admiten el curso de Microsoft [DP-300: Administración de soluciones de Microsoft Azure SQL](https://docs.microsoft.com/training/courses/dp-300t00).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Módulo | Ejercicio |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

