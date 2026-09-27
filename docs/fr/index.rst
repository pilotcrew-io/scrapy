==========================================
Scrapy en français : le guide du débutant
==========================================

.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. Cette sélection de pages est une traduction fidèle : rien n'est résumé ni retiré. En cas de doute ou d'ambiguïté, la référence reste la `documentation officielle en anglais <https://docs.scrapy.org>`_ (les fichiers d'origine sont dans le dossier ``docs/`` de ce dépôt).

Scrapy est un framework Python rapide et de haut niveau pour le *web crawling* (parcourir des sites web) et le *web scraping* (en extraire des données structurées). Il sert à des usages très variés, de l'exploration de données à la surveillance et aux tests automatisés.

Vous voulez utiliser Scrapy avec Claude ?
=============================================

Si vous ne savez pas (ou peu) programmer et que vous comptez vous faire aider par Claude pour écrire vos spiders, commencez directement ici plutôt que par le tutoriel technique ci-dessous :

→ `Utiliser Scrapy avec Claude (guide débutant) <guide-claude.rst>`_

Ce guide explique comment installer Scrapy, formuler vos demandes à Claude, lancer et corriger un spider, exporter vos données et rester respectueux des sites visités — sans avoir besoin de lire le code vous-même.

Par où commencer si vous voulez apprendre à coder vous-même ?
==================================================================

Si vous préférez comprendre et écrire le code vous-même, lisez les pages suivantes dans cet ordre.

1. `Vue d'ensemble <intro/overview.rst>`_ : comprendre ce qu'est Scrapy et à quoi il sert.
2. `Installation <intro/install.rst>`_ : installer Scrapy sur votre ordinateur.
3. `Tutoriel <intro/tutorial.rst>`_ : écrire votre premier projet Scrapy pas à pas.
4. `Exemples <intro/examples.rst>`_ : en apprendre plus avec un projet déjà prêt.

Les notions de base
===================

Une fois le tutoriel terminé (ou le guide Claude en main), ces pages détaillent les briques que vous utiliserez souvent. Ce sont des pages de référence technique, plus denses que le guide Claude — utiles quand vous voulez comprendre en détail ce qui a été écrit pour vous, ou affiner vous-même.

* `Outil en ligne de commande <topics/commands.rst>`_ : gérer votre projet avec la commande ``scrapy``.
* `Spiders <topics/spiders.rst>`_ : écrire les règles pour parcourir vos sites.
* `Sélecteurs <topics/selectors.rst>`_ : extraire les données des pages web.
* `Items <topics/items.rst>`_ : définir les données que vous voulez récupérer.
* `Shell interactif <topics/shell.rst>`_ : tester votre code d'extraction en direct.
* `Item pipeline <topics/item-pipeline.rst>`_ : traiter et enregistrer les données récupérées.
* `Requêtes et réponses <topics/request-response.rst>`_ : comprendre les classes qui représentent les échanges HTTP.
* `Exports de feeds <topics/feed-exports.rst>`_ : enregistrer vos données en CSV, JSON, XML...
* `Bonnes pratiques <topics/practices.rst>`_ : respecter les sites que vous visitez (robots.txt, vitesse, blocages).
* `Paramètres (settings) <topics/settings.rst>`_ : configurer Scrapy et voir tous les paramètres disponibles.

Et le reste de la documentation ?
=================================

Les autres sujets (cookies, middlewares, déploiement, architecture avancée, etc.) ne sont pas encore traduits. Ils sont disponibles en anglais dans la `documentation officielle <https://docs.scrapy.org>`_.
