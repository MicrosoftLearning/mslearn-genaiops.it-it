---
title: Esercizi di GenAIOps
permalink: index.html
layout: home
---

# Rendere operative le applicazioni di IA generativa

Gli esercizi di avvio rapido seguenti sono progettati per offrire un'esperienza di apprendimento pratica in cui verranno esaminate le attività comuni necessarie per rendere operativo un carico di lavoro di IA generativa in Microsoft Azure.

> **Nota**: per completare gli esercizi, è necessaria una sottoscrizione di Azure in cui si dispone di autorizzazioni e quote sufficienti per effettuare il provisioning delle risorse di Azure e dei modelli di IA generativa necessari. Creazione di un [account di Azure](https://azure.microsoft.com/free), se non già disponibile. È disponibile un'opzione di versione gratuita per i nuovi utenti che include crediti per i primi 30 giorni.

## Esercizi di avvio rapido

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

> **Note**: anche se è possibile completare questi esercizi autonomamente, questi sono progettati per integrare i moduli in [Microsoft Learn](https://learn.microsoft.com/training/paths/operationalize-gen-ai-apps/), in cui è possibile approfondire alcuni dei concetti sottostanti su cui essi si basano.
