.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/shell.rst`.

.. _topics-shell:

============
Scrapy shell
============

Le Scrapy shell est un shell interactif dans lequel vous pouvez essayer et
déboguer très rapidement votre code de scraping, sans avoir à lancer le spider.
Il est conçu pour tester le code d'extraction de données, mais vous pouvez en
réalité l'utiliser pour tester n'importe quel type de code, puisque c'est aussi
un shell Python ordinaire.

Le shell sert à tester des expressions XPath ou CSS et à voir comment elles
fonctionnent et quelles données elles extraient des pages web que vous essayez
de scraper. Il vous permet de tester vos expressions de façon interactive
pendant que vous écrivez votre spider, sans avoir à lancer le spider pour
tester chaque modification.

Une fois que vous serez à l'aise avec le Scrapy shell, vous verrez que c'est un
outil précieux pour développer et déboguer vos spiders.

.. _shell-config:

Configurer le shell
===================

Avec l'extra :ref:`ptpython <extras>`, le Scrapy shell utilise ptpython_ à la
place du :term:`REPL`. ptpython offre la coloration syntaxique, l'auto-complétion
intelligente, et bien plus encore.

À défaut, avec l'extra :ref:`ipython <extras>`, le Scrapy shell utilise IPython_
à la place. IPython offre l'auto-complétion intelligente, un affichage en
couleurs, et bien plus encore.

Scrapy prend aussi en charge `bpython`_ via l'extra :ref:`bpython <extras>`, et
essaiera de l'utiliser lorsque ni ptpython ni IPython ne sont disponibles.

Grâce aux paramètres de Scrapy, vous pouvez le configurer pour qu'il utilise
l'un de ``ptpython``, ``ipython``, ``bpython`` ou le shell ``python`` standard,
quels que soient ceux qui sont installés. Pour cela, définissez la variable
d'environnement ``SCRAPY_PYTHON_SHELL`` ou définissez-la dans votre fichier
:ref:`scrapy.cfg <topics-config-settings>` :

.. code-block:: ini

    [settings]
    shell = bpython

.. _ptpython: https://github.com/prompt-toolkit/ptpython
.. _IPython: https://ipython.org/
.. _bpython: https://bpython-interpreter.org/

Lancer le shell
===============

Pour lancer le Scrapy shell, vous pouvez utiliser la commande :command:`shell`
comme ceci ::

    scrapy shell <url>

où ``<url>`` est l'URL que vous voulez scraper.

:command:`shell` fonctionne aussi avec des fichiers locaux. C'est pratique si
vous voulez manipuler une copie locale d'une page web. :command:`shell`
comprend les syntaxes suivantes pour les fichiers locaux ::

    # UNIX-style
    scrapy shell ./path/to/file.html
    scrapy shell ../other/path/to/file.html
    scrapy shell /absolute/path/to/file.html

    # File URI
    scrapy shell file:///absolute/path/to/file.html

.. note:: Lorsque vous utilisez des chemins de fichiers relatifs, soyez
    explicite et faites-les précéder de ``./`` (ou de ``../`` selon le cas).
    ``scrapy shell index.html`` ne fonctionnera pas comme on pourrait s'y
    attendre (c'est voulu, ce n'est pas un bug).

    Comme :command:`shell` privilégie les URL HTTP par rapport aux URI de
    fichier, et comme ``index.html`` ressemble syntaxiquement à
    ``example.com``, :command:`shell` traitera ``index.html`` comme un nom de
    domaine et déclenchera une erreur de résolution DNS ::

        $ scrapy shell index.html
        [ ... scrapy shell starts ... ]
        [ ... traceback ... ]
        twisted.internet.error.DNSLookupError: DNS lookup failed:
        address 'index.html' not found: [Errno -5] No address associated with hostname.

    :command:`shell` ne vérifie pas au préalable si un fichier nommé
    ``index.html`` existe dans le répertoire courant. Encore une fois, soyez
    explicite.


Utiliser le shell
=================

Le Scrapy shell est simplement une console Python ordinaire (ou une console
`IPython`_ si elle est disponible) qui fournit quelques fonctions de raccourci
supplémentaires pour plus de commodité.

Raccourcis disponibles
----------------------

-   ``shelp()`` - affiche une aide avec la liste des objets et des raccourcis
    disponibles

-   ``fetch(url[, redirect=True])`` - récupère une nouvelle réponse à partir de
    l'URL donnée et met à jour tous les objets associés en conséquence. Vous
    pouvez demander, si vous le souhaitez, que les redirections HTTP 3xx ne
    soient pas suivies, en passant ``redirect=False``

-   ``fetch(request)`` - récupère une nouvelle réponse à partir de la requête
    donnée et met à jour tous les objets associés en conséquence.

-   ``view(response)`` - ouvre la réponse donnée dans votre navigateur web
    local, pour l'inspecter. Cela ajoute une `balise \<base\>`_ au corps de la
    réponse afin que les liens externes (comme les images et les feuilles de
    style) s'affichent correctement. Notez cependant que cela crée un fichier
    temporaire sur votre ordinateur, qui ne sera pas supprimé automatiquement.

.. _balise <base>: https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/base

Objets Scrapy disponibles
-------------------------

Le Scrapy shell crée automatiquement quelques objets pratiques à partir de la
page téléchargée, comme l'objet :class:`~scrapy.http.Response` et les objets
:class:`~scrapy.Selector` (pour le contenu HTML comme pour le contenu XML).

Ces objets sont :

-    ``crawler`` - l'objet :class:`~scrapy.crawler.Crawler` courant.

-   ``spider`` - le spider connu pour traiter l'URL, ou un objet
    :class:`~scrapy.Spider` s'il n'existe aucun spider pour l'URL courante

-   ``request`` - un objet :class:`~scrapy.Request` correspondant à la dernière
    page récupérée. Vous pouvez modifier cette requête avec
    :meth:`~scrapy.Request.replace` ou récupérer une nouvelle requête (sans
    quitter le shell) grâce au raccourci ``fetch``.

-   ``response`` - un objet :class:`~scrapy.http.Response` contenant la
    dernière page récupérée

-   ``settings`` - les :ref:`paramètres Scrapy <topics-settings>` courants

.. _shell-update-vars:

Ajouter vos propres objets
--------------------------

Pour définir des objets supplémentaires, ou pour exécuter du code chaque fois
qu'une réponse est récupérée, écrivez une :ref:`commande de projet personnalisée
<topics-commands>` dans un module nommé ``shell``, qui remplace la commande
:command:`shell`, puis redéfinissez sa méthode ``update_vars``. Elle est
appelée au démarrage et après chaque ``fetch``, et elle reçoit le dictionnaire
qui associe les noms de variables aux objets :

.. code-block:: python

    from scrapy.commands.shell import Command as ShellCommand


    class Command(ShellCommand):
        def update_vars(self, vars):
            from myproject.utils import parse_product

            vars["parse_product"] = parse_product
            if vars["response"] is not None:
                vars["product"] = parse_product(vars["response"])

``response`` vaut ``None`` lorsque le shell est lancé sans URL.

Exemple de session shell
========================

.. skip: start

Voici un exemple de session shell typique : nous commençons par scraper la page
https://www.scrapy.org/, puis nous passons à la page https://old.reddit.com/.
Enfin, nous changeons la méthode de la requête (Reddit) en POST et nous la
récupérons de nouveau, ce qui provoque une erreur. Nous terminons la session en
tapant Ctrl-D (sur les systèmes Unix) ou Ctrl-Z sous Windows.

Gardez à l'esprit que les données extraites ici peuvent ne pas être les mêmes
lorsque vous l'essaierez, car ces pages ne sont pas statiques et ont pu changer
au moment où vous testerez cet exemple. Le seul but de cet exemple est de vous
familiariser avec le fonctionnement du Scrapy shell.

D'abord, nous lançons le shell ::

    scrapy shell 'https://scrapy.org' --nolog

.. note::

   Pensez à toujours mettre les URL entre guillemets lorsque vous lancez le
   Scrapy shell depuis la ligne de commande, sinon les URL contenant des
   arguments (c'est-à-dire le caractère ``&``) ne fonctionneront pas.

   Sous Windows, utilisez plutôt des guillemets doubles ::

       scrapy shell "https://scrapy.org" --nolog


Ensuite, le shell récupère l'URL (en utilisant le downloader de Scrapy) et
affiche la liste des objets disponibles et des raccourcis utiles (vous
remarquerez que ces lignes commencent toutes par le préfixe ``[s]``) ::

    [s] Available Scrapy objects:
    [s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
    [s]   crawler    <scrapy.crawler.Crawler object at 0x7f07395dd690>
    [s]   item       {}
    [s]   request    <GET https://scrapy.org>
    [s]   response   <200 https://scrapy.org/>
    [s]   settings   <scrapy.settings.Settings object at 0x7f07395dd710>
    [s]   spider     <DefaultSpider 'default' at 0x7f0735891690>
    [s] Useful shortcuts:
    [s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
    [s]   fetch(req)                  Fetch a scrapy.Request and update local objects
    [s]   shelp()           Shell help (print this help)
    [s]   view(response)    View response in a browser

    >>>


Après cela, nous pouvons commencer à jouer avec les objets :

.. code-block:: pycon

    >>> response.xpath("//title/text()").get()
    'Scrapy | A Fast and Powerful Scraping and Web Crawling Framework'

    >>> fetch("https://old.reddit.com/")

    >>> response.xpath("//title/text()").get()
    'reddit: the front page of the internet'

    >>> request = request.replace(method="POST")

    >>> fetch(request)

    >>> response.status
    404

    >>> from pprint import pprint

    >>> pprint(response.headers)
    {'Accept-Ranges': ['bytes'],
    'Cache-Control': ['max-age=0, must-revalidate'],
    'Content-Type': ['text/html; charset=UTF-8'],
    'Date': ['Thu, 08 Dec 2016 16:21:19 GMT'],
    'Server': ['snooserv'],
    'Set-Cookie': ['loid=KqNLou0V9SKMX4qb4n; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                    'loidcreated=2016-12-08T16%3A21%3A19.445Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                    'loid=vi0ZVe4NkxNWdlH7r7; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                    'loidcreated=2016-12-08T16%3A21%3A19.459Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure'],
    'Vary': ['accept-encoding'],
    'Via': ['1.1 varnish'],
    'X-Cache': ['MISS'],
    'X-Cache-Hits': ['0'],
    'X-Content-Type-Options': ['nosniff'],
    'X-Frame-Options': ['SAMEORIGIN'],
    'X-Moose': ['majestic'],
    'X-Served-By': ['cache-cdg8730-CDG'],
    'X-Timer': ['S1481214079.394283,VS0,VE159'],
    'X-Ua-Compatible': ['IE=edge'],
    'X-Xss-Protection': ['1; mode=block']}

.. skip: end


.. _topics-shell-inspect-response:

Appeler le shell depuis les spiders pour inspecter les réponses
===============================================================

Il arrive que vous vouliez inspecter les réponses traitées à un certain point de
votre spider, ne serait-ce que pour vérifier que la réponse attendue y arrive
bien.

Vous pouvez y parvenir en utilisant la fonction ``scrapy.shell.inspect_response``.

Voici un exemple de la façon de l'appeler depuis votre spider :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"
        start_urls = [
            "http://example.com",
            "http://example.org",
            "http://example.net",
        ]

        def parse(self, response):
            # We want to inspect one specific response.
            if ".org" in response.url:
                from scrapy.shell import inspect_response

                inspect_response(response, self)

            # Rest of parsing code.

.. skip: start

Lorsque vous lancez le spider, vous obtenez quelque chose de semblable à ceci ::

    2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.com> (referer: None)
    2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.org> (referer: None)
    [s] Available Scrapy objects:
    [s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
    ...

    >>> response.url
    'http://example.org'

Ensuite, vous pouvez vérifier si le code d'extraction fonctionne :

.. code-block:: pycon

    >>> response.xpath('//h1[@class="fn"]')
    []

Non, ce n'est pas le cas. Vous pouvez donc ouvrir la réponse dans votre
navigateur web et voir si c'est bien la réponse que vous attendiez :

.. code-block:: pycon

    >>> view(response)
    True

Enfin, vous appuyez sur Ctrl-D (ou Ctrl-Z sous Windows) pour quitter le shell et
reprendre le crawl ::

    >>> ^D
    2014-01-23 17:50:03-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.net> (referer: None)
    ...

.. skip: end

Notez que vous ne pouvez pas utiliser le raccourci ``fetch`` ici, car l'engine
de Scrapy est bloqué par le shell. En revanche, une fois que vous quittez le
shell, le spider reprend le crawl là où il s'était arrêté, comme le montre
l'exemple ci-dessus.
