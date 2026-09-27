.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../intro/overview.rst`.

.. _intro-overview:

=========================
Scrapy en un coup d'œil
=========================

.. tip::
   Vous ne voulez pas écrire de code vous-même ? Le guide :doc:`../guide-claude` explique comment faire écrire vos spiders par Claude, sans avoir besoin de lire cette page technique en détail.

Scrapy (/ˈskreɪpaɪ/) est un framework d'application qui sert à parcourir
(« crawler ») des sites web et à en extraire des données structurées. Ces
données peuvent être utilisées dans un grand nombre d'applications utiles,
comme l'exploration de données (data mining), le traitement d'informations ou
l'archivage historique.

Même si Scrapy a été conçu à l'origine pour le `web scraping`_, il peut aussi
servir à extraire des données à l'aide d'API (comme `Amazon Associates Web
Services`_) ou à jouer le rôle de crawler web généraliste.


Parcours d'un exemple de spider
===============================

Pour vous montrer ce que Scrapy apporte, nous allons parcourir pas à pas un
exemple de Spider Scrapy, en utilisant la manière la plus simple de lancer un
spider.

Voici le code d'un spider qui récupère des citations célèbres sur le site
https://quotes.toscrape.com, en suivant la pagination :

.. code-block:: python

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            "https://quotes.toscrape.com/tag/humor/",
        ]

        def parse(self, response):
            for quote in response.css("div.quote"):
                yield {
                    "author": quote.xpath("span/small/text()").get(),
                    "text": quote.css("span.text::text").get(),
                }

            next_page = response.css('li.next a::attr("href")').get()
            if next_page is not None:
                yield response.follow(next_page, self.parse)

Placez ce code dans un fichier texte, donnez-lui un nom comme
``quotes_spider.py``, puis lancez le spider avec la commande
:command:`runspider` ::

    scrapy runspider quotes_spider.py -o quotes.jsonl

Quand l'exécution est terminée, vous obtenez dans le fichier ``quotes.jsonl``
une liste de citations au format JSON Lines. Elle contient le texte et
l'auteur de chaque citation, et ressemble à ceci ::

    {"author": "Jane Austen", "text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d"}
    {"author": "Steve Martin", "text": "\u201cA day without sunshine is like, you know, night.\u201d"}
    {"author": "Garrison Keillor", "text": "\u201cAnyone who thinks sitting in church can make you a Christian must also think that sitting in a garage can make you a car.\u201d"}
    ...


Que s'est-il passé ?
--------------------

Quand vous avez lancé la commande ``scrapy runspider quotes_spider.py``,
Scrapy a cherché une définition de Spider dans le fichier et l'a exécutée
grâce à son engine de crawl.

Le crawl a commencé par envoyer des requêtes aux URL définies dans l'attribut
``start_urls`` (ici, uniquement l'URL des citations de la catégorie *humor*).
Scrapy a ensuite appelé la méthode de callback par défaut, ``parse``, en lui
passant l'objet réponse en argument. Dans le callback ``parse``, nous
parcourons en boucle les éléments de citation à l'aide d'un sélecteur CSS,
nous produisons (``yield``) un dict Python contenant le texte et l'auteur de la
citation extraits, nous cherchons un lien vers la page suivante et nous
planifions une nouvelle requête en réutilisant la même méthode ``parse`` comme
callback.

Vous remarquez ici l'un des principaux avantages de Scrapy : les requêtes sont
:ref:`planifiées et traitées de manière asynchrone <topics-architecture>`. Cela
signifie que Scrapy n'a pas besoin d'attendre qu'une requête soit terminée et
traitée : il peut envoyer une autre requête ou faire autre chose entre-temps.
Cela signifie aussi que les autres requêtes peuvent continuer même si une
requête échoue ou si une erreur survient pendant son traitement.

Cela permet de réaliser des crawls très rapides (en envoyant plusieurs requêtes
simultanées en même temps, de façon tolérante aux pannes). Mais Scrapy vous
donne aussi le contrôle de la politesse du crawl, c'est-à-dire du respect des
serveurs visités, grâce à :ref:`quelques paramètres <topics-settings-ref>`. Vous
pouvez par exemple définir un délai de téléchargement entre chaque requête,
limiter le nombre de requêtes simultanées par domaine, et même
:ref:`utiliser une extension d'auto-régulation (auto-throttling)
<topics-autothrottle>` qui essaie de déterminer ces paramètres automatiquement.

.. note::

    Cet exemple utilise les :ref:`feed exports <topics-feed-exports>` pour
    générer le fichier JSON Lines. Vous pouvez facilement changer le format
    d'export (XML ou CSV, par exemple) ou le stockage (FTP ou `Amazon S3`_, par
    exemple). Vous pouvez aussi écrire un :ref:`item pipeline
    <topics-item-pipeline>` pour enregistrer les items dans une base de
    données.


.. _topics-whatelse:

Quoi d'autre ?
==============

Vous avez vu comment extraire et enregistrer des items depuis un site web avec
Scrapy, mais ce n'est que la surface. Scrapy propose de nombreuses
fonctionnalités puissantes qui rendent le scraping simple et efficace, comme :

* Une prise en charge intégrée de la :ref:`sélection et de l'extraction
  <topics-selectors>` de données à partir de sources HTML/XML, à l'aide de
  sélecteurs CSS étendus et d'expressions XPath, avec des méthodes utilitaires
  pour extraire des données à l'aide d'expressions régulières.

* Un :ref:`shell interactif en console <topics-shell>` (compatible IPython) pour
  essayer des expressions CSS et XPath afin d'extraire des données. C'est très
  utile pour écrire ou déboguer vos spiders.

* Une prise en charge intégrée de la :ref:`génération de feed exports
  <topics-feed-exports>` dans plusieurs formats (JSON, CSV, XML) et de leur
  stockage sur plusieurs supports (FTP, S3, système de fichiers local).

* Une gestion robuste des encodages, avec détection automatique, pour traiter
  des déclarations d'encodage étrangères, non standard ou erronées.

* :ref:`Une forte capacité d'extension <extending-scrapy>`, qui vous permet
  d'ajouter vos propres fonctionnalités grâce aux :ref:`signals
  <topics-signals>` et à une API bien définie (middlewares,
  :ref:`extensions <topics-extensions>` et :ref:`pipelines
  <topics-item-pipeline>`).

* Un large éventail d'extensions et de middlewares intégrés pour gérer :

  - les cookies et la gestion de session
  - les fonctionnalités HTTP comme la compression, l'authentification, le cache
  - l'usurpation du user-agent (user-agent spoofing)
  - le fichier robots.txt
  - la limitation de la profondeur du crawl
  - et bien plus encore

* Plusieurs moyens d':ref:`exécuter du code à l'intérieur d'un processus Scrapy
  <connect-live-crawl>`, pour l'inspecter et le déboguer.

* Un :ref:`plugin officiel pour les agents de programmation <agents>`.

* Ainsi que d'autres atouts, comme des spiders réutilisables pour parcourir des
  sites à partir de `Sitemaps`_ et de feeds XML/CSV, un pipeline de médias pour
  :ref:`télécharger automatiquement des images <topics-media-pipeline>` (ou tout
  autre média) associées aux items extraits, un résolveur DNS avec cache, et
  bien d'autres choses encore !

Et ensuite ?
============

Les prochaines étapes pour vous sont d':ref:`installer Scrapy <intro-install>`,
de :ref:`suivre le tutoriel <intro-tutorial>` pour apprendre à créer un projet
Scrapy complet, et de `rejoindre la communauté`_. Merci de l'intérêt que vous
portez à Scrapy !

.. _rejoindre la communauté: https://www.scrapy.org/community
.. _web scraping: https://en.wikipedia.org/wiki/Web_scraping
.. _Amazon Associates Web Services: https://affiliate-program.amazon.com/welcome/ecs
.. _Amazon S3: https://aws.amazon.com/s3/
.. _Sitemaps: https://www.sitemaps.org/index.html
