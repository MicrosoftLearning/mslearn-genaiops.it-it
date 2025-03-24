---
title: Istruzioni ospitate online
permalink: index.html
layout: home
---

# Microsoft Learn - Esercizi pratici

Gli esercizi pratici seguenti sono progettati per supportare il training di [Microsoft Learn](https://docs.microsoft.com/training/) .

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %}
| |
| --- | --- | 
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
