.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/practices.rst`.

.. _topics-practices:

================
Bonnes pratiques
================

Cette section documente les bonnes pratiques courantes lors de l'utilisation de
Scrapy. Ce sont des sujets transversaux qui ne trouvent pas naturellement leur
place dans une autre section spécifique.

.. skip: start

.. _run-from-script:

Lancer Scrapy depuis un script
==============================

Vous pouvez utiliser l':ref:`API <topics-api>` pour lancer Scrapy depuis un
script, au lieu de la façon habituelle de lancer Scrapy via ``scrapy crawl``.

Rappelez-vous que Scrapy nécessite un reactor Twisted, ou (avec
:setting:`TWISTED_REACTOR_ENABLED` réglé sur ``False``) une boucle d'événements
asyncio. Vous devez donc en faire tourner un dans votre script pour que cela
fonctionne (les utilitaires décrits ci-dessous peuvent le faire pour vous).

Le premier utilitaire que vous pouvez utiliser pour lancer vos spiders est
:class:`scrapy.crawler.AsyncCrawlerProcess` ou
:class:`scrapy.crawler.CrawlerProcess`. Ces classes démarrent un reactor
Twisted pour vous, configurent la journalisation et définissent des
gestionnaires d'arrêt. Ce sont ces classes qu'utilisent toutes les commandes
Scrapy. Elles offrent des fonctionnalités similaires, avec un style d'API
asynchrone différent : :class:`~scrapy.crawler.AsyncCrawlerProcess` renvoie
des coroutines depuis ses méthodes asynchrones, tandis que
:class:`~scrapy.crawler.CrawlerProcess` renvoie des objets
:class:`~twisted.internet.defer.Deferred`.

Voici un exemple montrant comment lancer un seul spider avec cet utilitaire.

.. code-block:: python

    import scrapy
    from scrapy.crawler import AsyncCrawlerProcess


    class MySpider(scrapy.Spider):
        # Your spider definition
        ...


    process = AsyncCrawlerProcess(
        settings={
            "FEEDS": {
                "items.json": {"format": "json"},
            },
        }
    )

    process.crawl(MySpider)
    process.start()  # the script will block here until the crawling is finished

Vous pouvez définir des :ref:`paramètres <topics-settings>` dans le
dictionnaire passé à :class:`~scrapy.crawler.AsyncCrawlerProcess`. Pensez à
consulter la documentation de :class:`~scrapy.crawler.AsyncCrawlerProcess`
pour bien comprendre les détails de son utilisation.

Si vous êtes à l'intérieur d'un projet Scrapy, quelques utilitaires
supplémentaires vous permettent d'importer ces composants depuis le projet.
Vous pouvez importer automatiquement vos spiders en passant leur nom à
:class:`~scrapy.crawler.AsyncCrawlerProcess`, et utiliser
:func:`scrapy.utils.project.get_project_settings` pour obtenir une instance
de :class:`~scrapy.settings.Settings` contenant les paramètres de votre
projet.

Voici un exemple concret de comment faire cela, en utilisant le projet
`testspiders`_ comme exemple.

.. code-block:: python

    from scrapy.crawler import AsyncCrawlerProcess
    from scrapy.utils.project import get_project_settings

    process = AsyncCrawlerProcess(get_project_settings())

    # 'followall' is the name of one of the spiders of the project.
    process.crawl("followall", domain="scrapy.org")
    process.start()  # the script will block here until the crawling is finished

Il existe un autre utilitaire Scrapy qui offre un contrôle plus fin sur le
processus de crawl : :class:`scrapy.crawler.AsyncCrawlerRunner` ou
:class:`scrapy.crawler.CrawlerRunner`. Ces classes sont de simples
enveloppes qui regroupent quelques utilitaires pratiques pour lancer
plusieurs crawlers, mais elles ne démarrent pas et n'interfèrent pas avec les
reactors déjà existants. Tout comme
:class:`scrapy.crawler.AsyncCrawlerProcess` et
:class:`scrapy.crawler.CrawlerProcess`, elles diffèrent par leur style d'API
asynchrone.

Lorsque vous utilisez ces classes, le reactor doit être explicitement lancé
après avoir programmé vos spiders. Il est recommandé d'utiliser
:class:`~scrapy.crawler.AsyncCrawlerRunner` ou
:class:`~scrapy.crawler.CrawlerRunner` plutôt que
:class:`~scrapy.crawler.AsyncCrawlerProcess` ou
:class:`~scrapy.crawler.CrawlerProcess` si votre application utilise déjà
Twisted et que vous voulez lancer Scrapy dans le même reactor.

Si vous voulez arrêter le reactor ou exécuter n'importe quel autre code juste
après la fin du spider, vous pouvez le faire une fois que la tâche renvoyée
par :meth:`AsyncCrawlerRunner.crawl()
<scrapy.crawler.AsyncCrawlerRunner.crawl>` est terminée (ou que le Deferred
renvoyé par :meth:`CrawlerRunner.crawl() <scrapy.crawler.CrawlerRunner.crawl>`
se déclenche). Dans le cas le plus simple, vous pouvez aussi utiliser
:func:`twisted.internet.task.react` pour démarrer et arrêter le reactor, même
s'il peut être plus simple d'utiliser directement
:class:`~scrapy.crawler.AsyncCrawlerProcess` ou
:class:`~scrapy.crawler.CrawlerProcess`.

Voici un exemple d'utilisation de :class:`~scrapy.crawler.AsyncCrawlerRunner`
avec un code simple de gestion du reactor :

.. code-block:: python

    import scrapy
    from scrapy.crawler import AsyncCrawlerRunner
    from scrapy.utils.defer import deferred_f_from_coro_f
    from scrapy.utils.log import configure_logging
    from scrapy.utils.reactor import install_reactor
    from twisted.internet.task import react


    class MySpider(scrapy.Spider):
        # Your spider definition
        ...


    async def crawl(_):
        configure_logging({"LOG_FORMAT": "%(levelname)s: %(message)s"})
        runner = AsyncCrawlerRunner()
        await runner.crawl(MySpider)  # completes when the spider finishes


    install_reactor()
    react(deferred_f_from_coro_f(crawl))

Le même exemple, mais utilisant :class:`~scrapy.crawler.CrawlerRunner` et un
reactor différent (:class:`~scrapy.crawler.AsyncCrawlerRunner` ne fonctionne
qu'avec :class:`~twisted.internet.asyncioreactor.AsyncioSelectorReactor`) :

.. code-block:: python

    import scrapy
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.log import configure_logging
    from scrapy.utils.reactor import install_reactor
    from twisted.internet.task import react


    class MySpider(scrapy.Spider):
        custom_settings = {
            "TWISTED_REACTOR": "twisted.internet.epollreactor.EPollReactor",
        }
        # Your spider definition
        ...


    def crawl(_):
        configure_logging({"LOG_FORMAT": "%(levelname)s: %(message)s"})
        runner = CrawlerRunner()
        d = runner.crawl(MySpider)
        return d  # this Deferred fires when the spider finishes


    install_reactor("twisted.internet.epollreactor.EPollReactor")
    react(crawl)

.. seealso:: :doc:`twisted:core/howto/reactor-basics`

Voici également des exemples d'utilisation de ces classes avec
:setting:`TWISTED_REACTOR_ENABLED` réglé sur ``False``.

Utilisation simple de :class:`~scrapy.crawler.AsyncCrawlerProcess` :

.. code-block:: python

    import scrapy
    from scrapy.crawler import AsyncCrawlerProcess


    class MySpider(scrapy.Spider):
        # Your spider definition
        ...


    process = AsyncCrawlerProcess(
        settings={
            "TWISTED_REACTOR_ENABLED": False,
        }
    )

    process.crawl(MySpider)
    process.start()  # the script will block here until the crawling is finished

Avec ``TWISTED_REACTOR_ENABLED=False``, vous pouvez utiliser plusieurs
instances de :class:`~scrapy.crawler.AsyncCrawlerProcess` dans le même
processus :

.. code-block:: python

    import scrapy
    from scrapy.crawler import AsyncCrawlerProcess


    class MySpider(scrapy.Spider):
        # Your spider definition
        ...


    process1 = AsyncCrawlerProcess(
        settings={
            "TWISTED_REACTOR_ENABLED": False,
        }
    )
    process1.crawl(MySpider)
    process1.start()

    process2 = AsyncCrawlerProcess(
        settings={
            "TWISTED_REACTOR_ENABLED": False,
        }
    )
    process2.crawl(MySpider)
    process2.start()

Utilisation de :func:`asyncio.run` avec
:class:`~scrapy.crawler.AsyncCrawlerRunner` :

.. code-block:: python

    import asyncio

    import scrapy
    from scrapy.crawler import AsyncCrawlerRunner
    from scrapy.utils.log import configure_logging


    class MySpider(scrapy.Spider):
        # Your spider definition
        ...


    async def main():
        configure_logging({"LOG_FORMAT": "%(levelname)s: %(message)s"})
        runner = AsyncCrawlerRunner(settings={"TWISTED_REACTOR_ENABLED": False})
        await runner.crawl(MySpider)  # completes when the spider finishes


    asyncio.run(main())

.. _run-spiders-in-apps:

Lancer des spiders au sein d'applications existantes
====================================================

Vous voudrez peut-être lancer des spiders Scrapy au sein d'une application
existante. Dans les cas simples (par exemple des files de tâches qui
démarrent un processus pour chaque tâche, ou des applications qui peuvent
exécuter des tâches de façon synchrone dans le même processus), vous pouvez
utiliser la même approche que pour des scripts autonomes (voir
:ref:`run-from-script`). Les cas plus complexes, comme les applications web
asynchrones, présentent des contraintes et des limites supplémentaires.

Si l'application fait tourner son propre reactor Twisted, vous pouvez
utiliser :class:`~scrapy.crawler.AsyncCrawlerRunner` ou
:class:`~scrapy.crawler.CrawlerRunner` pour lancer des spiders avec ce
reactor ; voir :ref:`run-from-script` pour des exemples.

Si l'application ne fait tourner ni reactor Twisted ni boucle d'événements
asyncio (par exemple une application web Django déployée avec un serveur WSGI
comme uWSGI), vous pouvez utiliser
:class:`~scrapy.crawler.AsyncCrawlerProcess` avec
:setting:`TWISTED_REACTOR_ENABLED` réglé sur ``False``, afin que Scrapy
démarre et arrête une boucle d'événements asyncio à chaque lancement de
spider :

.. code-block:: python

    import scrapy
    from django.http import HttpResponse
    from scrapy.crawler import AsyncCrawlerProcess


    class MySpider(scrapy.Spider):
        # Your spider definition
        ...


    def crawl_view(request):
        process = AsyncCrawlerProcess(settings={"TWISTED_REACTOR_ENABLED": False})
        process.crawl(MySpider)
        process.start()  # returns when the spider finishes
        return HttpResponse("Crawling finished")

Si l'application fait tourner sa propre boucle d'événements asyncio (par
exemple une application web Django déployée avec un serveur ASGI comme
uvicorn), vous pouvez utiliser :class:`~scrapy.crawler.AsyncCrawlerRunner`
avec :setting:`TWISTED_REACTOR_ENABLED` réglé sur ``False``, afin que Scrapy
utilise la boucle d'événements existante :

.. code-block:: python

    import scrapy
    from django.http import HttpResponse
    from scrapy.crawler import AsyncCrawlerRunner


    class MySpider(scrapy.Spider):
        # Your spider definition
        ...


    async def crawl_view(request):
        runner = AsyncCrawlerRunner(settings={"TWISTED_REACTOR_ENABLED": False})
        await runner.crawl(MySpider)  # completes when the spider finishes
        return HttpResponse("Crawling finished")

.. note:: Faire tourner Scrapy sans reactor Twisted est expérimental et
    comporte certaines limites, décrites dans :ref:`asyncio-without-reactor`.

.. _run-in-notebook:

Lancer des spiders dans des notebooks Jupyter
=============================================

Vous pouvez lancer des spiders Scrapy dans des notebooks Jupyter. Pour cela,
vous devez utiliser :class:`~scrapy.crawler.AsyncCrawlerRunner` avec
:setting:`TWISTED_REACTOR_ENABLED` réglé sur ``False``, afin que Scrapy
utilise la boucle d'événements fournie par le kernel du notebook. Comme
:class:`~scrapy.crawler.AsyncCrawlerRunner` ne configure pas la
journalisation, et que vous voudrez probablement voir le log du spider dans
le notebook, vous devez appeler
:func:`scrapy.utils.log.configure_logging`. Voici un exemple complet, qui
fonctionne aussi bien en une seule cellule qu'en cellules séparées :

.. code-block:: python

    from scrapy import Spider
    from scrapy.crawler import AsyncCrawlerRunner
    from scrapy.utils.log import configure_logging

    configure_logging()


    class BooksSpider(Spider):
        name = "books"
        start_urls = ["https://books.toscrape.com"]

        def parse(self, response):
            for book in response.css("h3"):
                yield {"title": book.css("a::attr(title)").get()}


    runner = AsyncCrawlerRunner({"TWISTED_REACTOR_ENABLED": False})
    await runner.crawl(BooksSpider)

.. note:: Faire tourner Scrapy sans reactor Twisted est expérimental et
    comporte certaines limites, décrites dans :ref:`asyncio-without-reactor`.

.. _run-multiple-spiders:

Lancer plusieurs spiders dans le même processus
===============================================

Par défaut, Scrapy lance un seul spider par processus lorsque vous exécutez
``scrapy crawl``. Cependant, Scrapy permet de lancer plusieurs spiders par
processus grâce à l':ref:`API interne <topics-api>`.

Chaque appel à ``crawl()`` crée son propre
:class:`~scrapy.crawler.Crawler`, avec ses propres instances du downloader et
des middlewares du spider, et ses propres :ref:`paramètres <topics-settings>`
résolus, y compris les :ref:`paramètres de spider <spider-settings>`. Rien
n'est partagé entre les différents spiders lancés dans le même processus.

Voici un exemple qui lance plusieurs spiders simultanément :

.. code-block:: python

    import scrapy
    from scrapy.crawler import AsyncCrawlerProcess
    from scrapy.utils.project import get_project_settings


    class MySpider1(scrapy.Spider):
        # Your first spider definition
        ...


    class MySpider2(scrapy.Spider):
        # Your second spider definition
        ...


    settings = get_project_settings()
    process = AsyncCrawlerProcess(settings)
    process.crawl(MySpider1)
    process.crawl(MySpider2)
    process.start()  # the script will block here until all crawling jobs are finished

Le même exemple avec :class:`~scrapy.crawler.AsyncCrawlerRunner` :

.. code-block:: python

    import scrapy
    from scrapy.crawler import AsyncCrawlerRunner
    from scrapy.utils.defer import deferred_f_from_coro_f
    from scrapy.utils.log import configure_logging
    from scrapy.utils.reactor import install_reactor
    from twisted.internet.task import react


    class MySpider1(scrapy.Spider):
        # Your first spider definition
        ...


    class MySpider2(scrapy.Spider):
        # Your second spider definition
        ...


    async def crawl(_):
        configure_logging({"LOG_FORMAT": "%(levelname)s: %(message)s"})
        runner = AsyncCrawlerRunner()
        runner.crawl(MySpider1)
        runner.crawl(MySpider2)
        await runner.join()  # completes when both spiders finish


    install_reactor()
    react(deferred_f_from_coro_f(crawl))


Le même exemple, mais en lançant les spiders de façon séquentielle, en
attendant la fin de chacun avant de démarrer le suivant :

.. code-block:: python

    import scrapy
    from scrapy.crawler import AsyncCrawlerRunner
    from scrapy.utils.defer import deferred_f_from_coro_f
    from scrapy.utils.log import configure_logging
    from scrapy.utils.reactor import install_reactor
    from twisted.internet.task import react


    class MySpider1(scrapy.Spider):
        # Your first spider definition
        ...


    class MySpider2(scrapy.Spider):
        # Your second spider definition
        ...


    async def crawl(_):
        configure_logging({"LOG_FORMAT": "%(levelname)s: %(message)s"})
        runner = AsyncCrawlerRunner()
        await runner.crawl(MySpider1)
        await runner.crawl(MySpider2)


    install_reactor()
    react(deferred_f_from_coro_f(crawl))

.. note:: Lorsque vous lancez plusieurs spiders dans le même processus, les
    :ref:`paramètres de journalisation <logging-settings>` et les
    :ref:`paramètres de reactor <reactor-settings>` ne doivent pas avoir de
    valeur différente selon le spider, et les :ref:`paramètres de
    pré-crawler <pre-crawler-settings>` ne peuvent pas être définis par
    spider.

Tous les autres paramètres s'appliquent séparément à chaque crawler. C'est le
cas des paramètres de concurrence et de politesse, comme
:setting:`CONCURRENT_REQUESTS`, :setting:`CONCURRENT_REQUESTS_PER_DOMAIN` et
:setting:`DOWNLOAD_DELAY`, et l':ref:`AutoThrottle <topics-autothrottle>`
régule lui aussi chaque crawler séparément. Lorsque vous crawlez
simultanément, divisez ces valeurs par le nombre de crawlers, afin de garder
inchangée la charge globale sur votre matériel et sur les sites cibles.

Pour cette raison, lancer le même spider plusieurs fois dans le même
processus multiplie ces limites au lieu d'augmenter la capacité de crawl.
Pour crawler plus vite, augmentez plutôt :setting:`CONCURRENT_REQUESTS` sur
un seul crawler.

.. seealso:: :ref:`run-from-script`.

.. skip: end

.. _distributed-crawls:

Crawls distribués
=================

Scrapy ne fournit aucune fonctionnalité intégrée pour lancer des crawls de
façon distribuée (sur plusieurs serveurs). Il existe cependant plusieurs
façons de distribuer des crawls, qui varient selon la manière dont vous
comptez procéder.

Si vous avez beaucoup de spiders, la façon la plus évidente de distribuer la
charge est de mettre en place plusieurs instances Scrapyd et de répartir les
lancements de spiders entre elles.

Si vous voulez plutôt faire tourner un seul (gros) spider sur plusieurs
machines, ce que l'on fait habituellement est de partitionner les URLs à
crawler et de les envoyer à chaque spider séparé. Voici un exemple concret :

D'abord, vous préparez la liste des URLs à crawler et vous les placez dans
des fichiers séparés ::

    http://somedomain.com/urls-to-crawl/spider1/part1.list
    http://somedomain.com/urls-to-crawl/spider1/part2.list
    http://somedomain.com/urls-to-crawl/spider1/part3.list

Ensuite, vous lancez un spider sur 3 serveurs Scrapyd différents. Le spider
reçoit un argument (de spider) ``part`` indiquant le numéro de la partition à
crawler ::

    curl http://scrapy1.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=1
    curl http://scrapy2.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=2
    curl http://scrapy3.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=3

.. _large-project-startup:

Réduire le temps de démarrage sur les gros projets
==================================================

Lorsque vous lancez un spider avec ``scrapy crawl``, Scrapy charge tous les
modules listés dans :setting:`SPIDER_MODULES` pour trouver le spider ciblé.
Dans les gros projets contenant beaucoup de spiders, cela peut augmenter de
façon notable le temps de démarrage et la consommation de mémoire.

Pour éviter de charger chaque module de spider, redéfinissez
:setting:`SPIDER_MODULES` en ligne de commande afin de ne pointer que vers le
module contenant le spider que vous voulez lancer :

.. code-block:: shell

    scrapy crawl myspider -s SPIDER_MODULES=myproject.spiders.myspider

Comme :setting:`SPIDER_MODULES` est un paramètre de type liste, vous pouvez
inclure plusieurs modules en les séparant par des virgules.

.. _bans:

Éviter de se faire bannir
=========================

Les sites web distinguent les visiteurs ordinaires des crawlers en observant
l'allure de leur trafic : les en-têtes qu'il transporte, la vitesse à
laquelle il arrive, le nombre de requêtes venant du même endroit. Un trafic
qui se distingue peut être bloqué, même lorsque le crawl en lui-même serait
bienvenu.

Là où le site web autorise le crawl, la chose la plus efficace à faire est de
vous faire connaître : donnez à :setting:`USER_AGENT` une valeur qui vous
identifie et permet à ses propriétaires de vous contacter, afin qu'ils
puissent vous demander d'ajuster votre crawler plutôt que de le bloquer.

Lorsque cela ne suffit pas, voici comment rendre votre trafic plus proche de
celui d'un visiteur ordinaire :

* faites tourner votre user agent parmi ceux des navigateurs courants, afin
  que vos requêtes ne se ressemblent pas toutes (cherchez sur le web une
  liste à jour)
* désactivez les cookies (voir :setting:`COOKIES_ENABLED`), afin qu'un
  identifiant de session ne relie pas toutes vos requêtes entre elles
* espacez vos requêtes, de 2 secondes ou plus, grâce au paramètre
  :setting:`DOWNLOAD_DELAY`, pour rapprocher votre rythme de celui d'une
  personne qui navigue
* lorsque c'est possible, lisez les pages depuis `Common Crawl`_, qui
  n'envoie aucun trafic au site web
* répartissez vos requêtes sur un ensemble d'adresses IP, afin qu'aucune
  d'entre elles ne porte l'ensemble de votre crawl. Par exemple, le projet
  gratuit `Tor project`_ ou des services payants comme `ProxyMesh`_.
* faites correspondre le comportement TLS à celui d'un navigateur : certains
  sites web répondent différemment selon la version TLS du client, que vous
  pouvez ajuster avec les paramètres
  :setting:`DOWNLOAD_TLS_MIN_VERSION` et :setting:`DOWNLOAD_TLS_MAX_VERSION`
* laissez un service s'occuper de tout cela, comme `Zyte API`_, qui fournit
  un `plugin Scrapy
  <https://github.com/scrapy-plugins/scrapy-zyte-api>`__ et des
  fonctionnalités supplémentaires, comme le `scraping web par IA
  <https://www.zyte.com/ai-web-scraping/>`__

Si votre crawler continue d'être bloqué, pensez à contacter le
`commercial support`_ (support commercial, en anglais).

.. _static-analysis:

Analyse statique
================

Pensez à utiliser :doc:`scrapy-lint <scrapy-lint:index>`, un linter pour les
projets Scrapy qui détecte les erreurs courantes et les anti-patterns.

.. _connect-live-crawl:

Se connecter à des crawls en cours
==================================

Il est utile de pouvoir se connecter à des crawls en cours de longue durée,
que ce soit pour suivre leur progression en détail ou pour enquêter sur des
problèmes. Scrapy fournit les outils suivants pour cela :

- :ref:`Console Telnet <topics-telnetconsole>` : se connecter à un processus
  de crawl avec un client telnet et y exécuter du code Python.
- :ref:`Serveur MCP Scrapy <using-mcp-server>` : pointer un agent de codage
  vers un processus de crawl afin qu'il puisse y exécuter du code Python.

.. _Tor project: https://www.torproject.org/
.. _commercial support: https://www.scrapy.org/companies
.. _ProxyMesh: https://proxymesh.com/
.. _Common Crawl: https://commoncrawl.org/
.. _testspiders: https://github.com/scrapinghub/testspiders
.. _Zyte API: https://docs.zyte.com/zyte-api/get-started.html
