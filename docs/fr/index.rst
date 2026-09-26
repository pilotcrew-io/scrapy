==========================================
Scrapy en français : le guide du débutant
==========================================

.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. Cette sélection de pages est une traduction fidèle : rien n'est résumé ni retiré. En cas de doute ou d'ambiguïté, la référence reste la `documentation officielle en anglais <https://docs.scrapy.org>`_ (les fichiers d'origine sont dans le dossier ``docs/`` de ce dépôt).

Scrapy est un framework Python rapide et de haut niveau pour le *web crawling* (parcourir des sites web) et le *web scraping* (en extraire des données structurées). Il sert à des usages très variés, de l'exploration de données à la surveillance et aux tests automatisés.

Par où commencer ?
==================

Si vous découvrez Scrapy, lisez les pages dans cet ordre.

1. `Vue d'ensemble <intro/overview.rst>`_ : comprendre ce qu'est Scrapy et à quoi il sert.
2. `Installation <intro/install.rst>`_ : installer Scrapy sur votre ordinateur.
3. `Tutoriel <intro/tutorial.rst>`_ : écrire votre premier projet Scrapy pas à pas.
4. `Exemples <intro/examples.rst>`_ : en apprendre plus avec un projet déjà prêt.

Les notions de base
===================

Une fois le tutoriel terminé, ces pages détaillent les briques que vous utiliserez tout le temps.

* `Outil en ligne de commande <topics/commands.rst>`_ : gérer votre projet avec la commande ``scrapy``.
* `Spiders <topics/spiders.rst>`_ : écrire les règles pour parcourir vos sites.
* `Sélecteurs <topics/selectors.rst>`_ : extraire les données des pages web.
* `Items <topics/items.rst>`_ : définir les données que vous voulez récupérer.
* `Shell interactif <topics/shell.rst>`_ : tester votre code d'extraction en direct.
* `Item pipeline <topics/item-pipeline.rst>`_ : traiter et enregistrer les données récupérées.
* `Requêtes et réponses <topics/request-response.rst>`_ : comprendre les classes qui représentent les échanges HTTP.
* `Paramètres (settings) <topics/settings.rst>`_ : configurer Scrapy et voir tous les paramètres disponibles.

Et le reste de la documentation ?
=================================

Les autres sujets (cookies, exports de données, middlewares, déploiement, etc.) ne sont pas encore traduits. Ils sont disponibles en anglais dans la `documentation officielle <https://docs.scrapy.org>`_.
