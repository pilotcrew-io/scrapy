.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../intro/tutorial.rst`.

.. _intro-tutorial:

===============
Tutoriel Scrapy
===============

.. tip::
   Vous voulez vous faire aider par Claude plutôt qu'écrire ce code vous-même ? Le guide :doc:`../guide-claude` est fait pour ça.

Dans ce tutoriel, nous supposons que Scrapy est déjà installé sur votre système.
Si ce n'est pas le cas, consultez :ref:`intro-install`.

Nous allons extraire des données du site `quotes.toscrape.com <https://quotes.toscrape.com/>`_, un site
qui recense des citations d'auteurs célèbres.

Ce tutoriel vous guidera à travers les tâches suivantes :

1. Créer un nouveau projet Scrapy
2. Écrire un :ref:`spider <topics-spiders>` pour parcourir un site (crawl) et en extraire des données
3. Exporter les données extraites depuis la ligne de commande
4. Modifier le spider pour qu'il suive les liens de façon récursive
5. Utiliser des arguments de spider

Scrapy est écrit en Python_. Plus vous en apprenez sur Python, plus vous
pourrez tirer parti de Scrapy.

Si vous connaissez déjà d'autres langages et souhaitez apprendre Python rapidement, le
`Python Tutorial`_ (le tutoriel officiel, en anglais) est une bonne ressource.

Si vous débutez en programmation et souhaitez commencer par Python, les livres
suivants peuvent vous être utiles :

* `Automate the Boring Stuff With Python`_

* `How To Think Like a Computer Scientist`_

* `Learn Python 3 The Hard Way`_

Vous pouvez également consulter `cette liste de ressources Python pour les non-programmeurs`_,
ainsi que les `ressources suggérées dans le subreddit learnpython`_.

.. _Python: https://www.python.org/
.. _cette liste de ressources Python pour les non-programmeurs: https://wiki.python.org/moin/BeginnersGuide/NonProgrammers
.. _Python Tutorial: https://docs.python.org/3/tutorial
.. _Automate the Boring Stuff With Python: https://automatetheboringstuff.com/
.. _How To Think Like a Computer Scientist: http://openbookproject.net/thinkcs/python/english3e/
.. _Learn Python 3 The Hard Way: https://learnpythonthehardway.org/python3/
.. _ressources suggérées dans le subreddit learnpython: https://www.reddit.com/r/learnpython/wiki/index#wiki_new_to_python.3F


Créer un projet
===============

Avant de commencer à extraire des données, vous devez créer un nouveau projet
Scrapy. Placez-vous dans un répertoire où vous souhaitez ranger votre code et
exécutez ::

    scrapy startproject tutorial

Cela créera un répertoire ``tutorial`` avec le contenu suivant ::

    tutorial/
        scrapy.cfg            # deploy configuration file

        tutorial/             # project's Python module, you'll import your code from here
            __init__.py

            items.py          # project items definition file

            middlewares.py    # project middlewares file

            pipelines.py      # project pipelines file

            settings.py       # project settings file

            spiders/          # a directory where you'll later put your spiders
                __init__.py

Avant de parcourir quoi que ce soit, ouvrez ``settings.py`` et décommentez la
ligne :setting:`USER_AGENT` afin de vous identifier, par exemple avec un nom de
projet accompagné d'une URL ou d'une adresse e-mail. Les propriétaires de sites
web que votre crawler dérange pourront ainsi vous demander de l'ajuster, plutôt
que de le bloquer.


Notre premier spider
====================

Les spiders sont des classes que vous définissez et que Scrapy utilise pour
extraire des informations d'un site web (ou d'un groupe de sites web). Elles
doivent hériter de :class:`~scrapy.Spider` et définir les requêtes initiales à
effectuer et, éventuellement, la façon de suivre les liens dans les pages et
d'analyser le contenu des pages téléchargées pour en extraire des données.

Voici le code de notre premier spider. Enregistrez-le dans un fichier nommé
``quotes_spider.py`` dans le répertoire ``tutorial/spiders`` de votre projet :

.. code-block:: python

    from pathlib import Path

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        async def start(self):
            urls = [
                "https://quotes.toscrape.com/page/1/",
                "https://quotes.toscrape.com/page/2/",
            ]
            for url in urls:
                yield scrapy.Request(url=url, callback=self.parse)

        def parse(self, response):
            page = response.url.split("/")[-2]
            filename = f"quotes-{page}.html"
            Path(filename).write_bytes(response.body)
            self.log(f"Saved file {filename}")


Comme vous pouvez le voir, notre spider hérite de
:class:`scrapy.Spider <scrapy.Spider>` et définit quelques attributs et
méthodes :

* :attr:`~scrapy.Spider.name` : identifie le spider. Il doit être unique au
  sein d'un projet, c'est-à-dire que vous ne pouvez pas donner le même nom à
  des spiders différents.

* :meth:`~scrapy.Spider.start` : doit être un générateur asynchrone qui produit
  (avec ``yield``) des requêtes (et, éventuellement, des items) pour que le
  spider commence son parcours. Les requêtes suivantes seront générées
  successivement à partir de ces requêtes initiales.

* :meth:`~scrapy.Spider.parse` : une méthode qui sera appelée pour traiter la
  réponse téléchargée pour chacune des requêtes effectuées. Le paramètre
  ``response`` est une instance de :class:`~scrapy.http.TextResponse` qui
  contient le contenu de la page et dispose d'autres méthodes utiles pour le
  traiter.

  La méthode :meth:`~scrapy.Spider.parse` analyse généralement la réponse, en
  extrait les données sous forme de dictionnaires, et trouve aussi de nouvelles
  URL à suivre pour créer de nouvelles requêtes (:class:`~scrapy.Request`) à
  partir de celles-ci.

Comment lancer notre spider
---------------------------

Pour mettre notre spider au travail, placez-vous dans le répertoire racine du
projet et exécutez ::

   scrapy crawl quotes

Cette commande lance le spider nommé ``quotes`` que nous venons d'ajouter, qui
enverra quelques requêtes au domaine ``quotes.toscrape.com``. Vous obtiendrez
une sortie similaire à celle-ci ::

    ... (omitted for brevity)
    2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
    2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
    2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET https://quotes.toscrape.com/robots.txt> (referer: None)
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://quotes.toscrape.com/page/1/> (referer: None)
    2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://quotes.toscrape.com/page/2/> (referer: None)
    2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-1.html
    2016-12-16 21:24:05 [quotes] DEBUG: Saved file quotes-2.html
    2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
    ...

Maintenant, regardez les fichiers présents dans le répertoire courant. Vous
devriez constater que deux nouveaux fichiers ont été créés : *quotes-1.html* et
*quotes-2.html*, contenant le contenu des URL correspondantes, comme le demande
notre méthode ``parse``.

.. note:: Si vous vous demandez pourquoi nous n'avons pas encore analysé le
  HTML, patience : nous y viendrons bientôt.


Que s'est-il passé en coulisses ?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Scrapy envoie les premiers objets :class:`scrapy.Request <scrapy.Request>`
produits par la méthode de spider :meth:`~scrapy.Spider.start`. À la réception
d'une réponse pour chacun d'eux, Scrapy appelle la méthode callback associée à
la requête (ici, la méthode ``parse``) en lui passant un objet
:class:`~scrapy.http.Response`.


Un raccourci pour la méthode ``start``
--------------------------------------

Au lieu d'implémenter une méthode :meth:`~scrapy.Spider.start` qui produit des
objets :class:`~scrapy.Request` à partir d'URL, vous pouvez définir un attribut
de classe :attr:`~scrapy.Spider.start_urls` contenant une liste d'URL. Cette
liste sera alors utilisée par l'implémentation par défaut de
:meth:`~scrapy.Spider.start` pour créer les requêtes initiales de votre
spider.

.. code-block:: python

    from pathlib import Path

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            "https://quotes.toscrape.com/page/1/",
            "https://quotes.toscrape.com/page/2/",
        ]

        def parse(self, response):
            page = response.url.split("/")[-2]
            filename = f"quotes-{page}.html"
            Path(filename).write_bytes(response.body)

La méthode :meth:`~scrapy.Spider.parse` sera appelée pour traiter chacune des
requêtes vers ces URL, même si nous n'avons pas explicitement demandé à Scrapy
de le faire. Cela s'explique par le fait que :meth:`~scrapy.Spider.parse` est la
méthode callback par défaut de Scrapy : elle est appelée pour les requêtes
auxquelles aucun callback n'a été explicitement assigné.


Extraire des données
--------------------

La meilleure façon d'apprendre à extraire des données avec Scrapy est
d'essayer des sélecteurs dans le :ref:`shell Scrapy <topics-shell>`. Exécutez ::

    scrapy shell 'https://quotes.toscrape.com/page/1/'

.. note::

   N'oubliez pas de toujours placer les URL entre guillemets lorsque vous lancez
   le shell Scrapy depuis la ligne de commande, sinon les URL contenant des
   arguments (c'est-à-dire le caractère ``&``) ne fonctionneront pas.

   Sous Windows, utilisez plutôt des guillemets doubles ::

       scrapy shell "https://quotes.toscrape.com/page/1/"

Vous verrez quelque chose comme ::

    [ ... Scrapy log here ... ]
    2016-09-19 12:09:27 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://quotes.toscrape.com/page/1/> (referer: None)
    [s] Available Scrapy objects:
    [s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
    [s]   crawler    <scrapy.crawler.Crawler object at 0x7fa91d888c90>
    [s]   item       {}
    [s]   request    <GET https://quotes.toscrape.com/page/1/>
    [s]   response   <200 https://quotes.toscrape.com/page/1/>
    [s]   settings   <scrapy.settings.Settings object at 0x7fa91d888c10>
    [s]   spider     <DefaultSpider 'default' at 0x7fa91c8af990>
    [s] Useful shortcuts:
    [s]   shelp()           Shell help (print this help)
    [s]   fetch(req_or_url) Fetch request (or URL) and update local objects
    [s]   view(response)    View response in a browser

Avec le shell, vous pouvez essayer de sélectionner des éléments en utilisant
`CSS`_ sur l'objet response :

.. invisible-code-block: python

    response = load_response('https://quotes.toscrape.com/page/1/', 'quotes1.html')

.. code-block:: pycon

    >>> response.css("title")
    [<Selector query='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]

Le résultat de ``response.css('title')`` est un objet de type liste appelé
:class:`~scrapy.selector.SelectorList`, qui représente une liste d'objets
:class:`~scrapy.Selector` enveloppant des éléments XML/HTML et vous permettant
d'exécuter d'autres requêtes pour affiner la sélection ou extraire les données.

Pour extraire le texte du titre ci-dessus, vous pouvez faire :

.. code-block:: pycon

    >>> response.css("title::text").getall()
    ['Quotes to Scrape']

Il y a deux choses à noter ici. La première : nous avons ajouté ``::text`` à la
requête CSS, ce qui signifie que nous voulons sélectionner uniquement les
éléments texte situés directement à l'intérieur de l'élément ``<title>``. Si
nous ne précisons pas ``::text``, nous obtenons l'élément title complet, avec
ses balises :

.. code-block:: pycon

    >>> response.css("title").getall()
    ['<title>Quotes to Scrape</title>']

La seconde : le résultat de l'appel à ``.getall()`` est une liste. Un sélecteur
peut renvoyer plus d'un résultat, c'est pourquoi nous les extrayons tous.
Lorsque vous savez que vous ne voulez que le premier résultat, comme ici, vous
pouvez faire :

.. code-block:: pycon

    >>> response.css("title::text").get()
    'Quotes to Scrape'

Vous auriez aussi pu écrire :

.. code-block:: pycon

    >>> response.css("title::text")[0].get()
    'Quotes to Scrape'

Accéder à un index d'une instance de :class:`~scrapy.selector.SelectorList`
lève une exception :exc:`IndexError` s'il n'y a aucun résultat :

.. code-block:: pycon

    >>> response.css("noelement")[0].get()
    Traceback (most recent call last):
    ...
    IndexError: list index out of range

Vous pouvez plutôt utiliser ``.get()`` directement sur l'instance de
:class:`~scrapy.selector.SelectorList`, ce qui renvoie ``None`` s'il n'y a aucun
résultat :

.. code-block:: pycon

    >>> response.css("noelement").get()

Il y a une leçon à en tirer : pour la plupart des codes d'extraction, vous
voulez qu'ils résistent aux erreurs dues à des éléments introuvables dans une
page, afin que, même si certaines parties échouent, vous obteniez au moins
**quelques** données.

En plus des méthodes :meth:`~scrapy.selector.SelectorList.getall` et
:meth:`~scrapy.selector.SelectorList.get`, vous pouvez aussi utiliser la méthode
:meth:`~scrapy.selector.SelectorList.re` pour extraire des données à l'aide
d'\ :doc:`expressions régulières <library/re>` :

.. code-block:: pycon

    >>> response.css("title::text").re(r"Quotes.*")
    ['Quotes to Scrape']
    >>> response.css("title::text").re(r"Q\w+")
    ['Quotes']
    >>> response.css("title::text").re(r"(\w+) to (\w+)")
    ['Quotes', 'Scrape']

Pour trouver les bons sélecteurs CSS à utiliser, il peut être utile d'ouvrir la
page de la réponse depuis le shell dans votre navigateur web avec
``view(response)``. Vous pouvez utiliser les outils de développement de votre
navigateur pour inspecter le HTML et en déduire un sélecteur (voir
:ref:`topics-developer-tools`).

`Selector Gadget`_ est également un bon outil pour trouver rapidement le
sélecteur CSS d'éléments choisis visuellement, et il fonctionne dans de
nombreux navigateurs.

.. _Selector Gadget: https://selectorgadget.com/


XPath : une brève introduction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

En plus de `CSS`_, les sélecteurs Scrapy prennent aussi en charge les
expressions `XPath`_ :

.. code-block:: pycon

    >>> response.xpath("//title")
    [<Selector query='//title' data='<title>Quotes to Scrape</title>'>]
    >>> response.xpath("//title/text()").get()
    'Quotes to Scrape'

Les expressions XPath sont très puissantes et constituent le fondement des
sélecteurs Scrapy. En fait, les sélecteurs CSS sont convertis en XPath en
interne. Vous pouvez le constater en lisant attentivement la représentation
textuelle des objets sélecteurs dans le shell.

Bien qu'elles soient peut-être moins populaires que les sélecteurs CSS, les
expressions XPath offrent plus de puissance, car en plus de naviguer dans la
structure du document, elles peuvent aussi examiner son contenu. Avec XPath,
vous pouvez sélectionner des éléments comme : *le lien qui contient le texte
"Next Page"*. Cela rend XPath très adapté à l'extraction de données, et nous
vous encourageons à apprendre XPath, même si vous savez déjà construire des
sélecteurs CSS : cela vous facilitera beaucoup la tâche.

Nous n'allons pas approfondir XPath ici, mais vous pouvez en lire plus sur
:ref:`l'utilisation de XPath avec les sélecteurs Scrapy ici <topics-selectors>`.
Pour en apprendre davantage sur XPath, nous recommandons `ce tutoriel pour
apprendre XPath par l'exemple <http://zvon.org/comp/r/tut-XPath_1.html>`_, et
`ce tutoriel pour apprendre à "penser en XPath"
<http://plasmasturm.org/log/xpath101/>`_ (tous deux en anglais).

.. _XPath: https://www.w3.org/TR/xpath-10/
.. _CSS: https://www.w3.org/TR/selectors

Extraire les citations et les auteurs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maintenant que vous en savez un peu plus sur la sélection et l'extraction,
terminons notre spider en écrivant le code qui extrait les citations de la page
web.

Chaque citation de https://quotes.toscrape.com est représentée par des éléments
HTML qui ressemblent à ceci :

.. code-block:: html

    <div class="quote">
        <span class="text">“The world as we have created it is a process of our
        thinking. It cannot be changed without changing our thinking.”</span>
        <span>
            by <small class="author">Albert Einstein</small>
            <a href="/author/Albert-Einstein">(about)</a>
        </span>
        <div class="tags">
            Tags:
            <a class="tag" href="/tag/change/page/1/">change</a>
            <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
            <a class="tag" href="/tag/thinking/page/1/">thinking</a>
            <a class="tag" href="/tag/world/page/1/">world</a>
        </div>
    </div>

Ouvrons le shell Scrapy et expérimentons un peu pour découvrir comment
extraire les données que nous voulons ::

    scrapy shell 'https://quotes.toscrape.com'

Nous obtenons une liste de sélecteurs pour les éléments HTML des citations avec :

.. code-block:: pycon

    >>> response.css("div.quote")
    [<Selector query="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
    <Selector query="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
    ...]

Chacun des sélecteurs renvoyés par la requête ci-dessus nous permet d'exécuter
d'autres requêtes sur leurs sous-éléments. Assignons le premier sélecteur à une
variable, afin de pouvoir exécuter nos sélecteurs CSS directement sur une
citation en particulier :

.. code-block:: pycon

    >>> quote = response.css("div.quote")[0]

Maintenant, extrayons le ``text``, l'``author`` et les ``tags`` de cette
citation en utilisant l'objet ``quote`` que nous venons de créer :

.. code-block:: pycon

    >>> text = quote.css("span.text::text").get()
    >>> text
    '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'
    >>> author = quote.css("small.author::text").get()
    >>> author
    'Albert Einstein'

Comme les tags forment une liste de chaînes de caractères, nous pouvons
utiliser la méthode ``.getall()`` pour tous les récupérer :

.. code-block:: pycon

    >>> tags = quote.css("div.tags a.tag::text").getall()
    >>> tags
    ['change', 'deep-thoughts', 'thinking', 'world']

.. invisible-code-block: python

  from sys import version_info

Maintenant que nous avons compris comment extraire chaque élément, nous pouvons
parcourir tous les éléments de citation et les rassembler dans un dictionnaire
Python :

.. code-block:: pycon

    >>> for quote in response.css("div.quote"):
    ...     text = quote.css("span.text::text").get()
    ...     author = quote.css("small.author::text").get()
    ...     tags = quote.css("div.tags a.tag::text").getall()
    ...     print(dict(text=text, author=author, tags=tags))
    ...
    {'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”', 'author': 'Albert Einstein', 'tags': ['change', 'deep-thoughts', 'thinking', 'world']}
    {'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”', 'author': 'J.K. Rowling', 'tags': ['abilities', 'choices']}
    ...

Extraire des données dans notre spider
--------------------------------------

Revenons à notre spider. Jusqu'à présent, il n'a extrait aucune donnée en
particulier : il s'est contenté d'enregistrer la page HTML entière dans un
fichier local. Intégrons la logique d'extraction ci-dessus dans notre spider.

Un spider Scrapy génère généralement de nombreux dictionnaires contenant les
données extraites de la page. Pour cela, nous utilisons le mot-clé Python
``yield`` dans le callback, comme vous pouvez le voir ci-dessous :

.. code-block:: python

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            "https://quotes.toscrape.com/page/1/",
            "https://quotes.toscrape.com/page/2/",
        ]

        def parse(self, response):
            for quote in response.css("div.quote"):
                yield {
                    "text": quote.css("span.text::text").get(),
                    "author": quote.css("small.author::text").get(),
                    "tags": quote.css("div.tags a.tag::text").getall(),
                }

Pour lancer ce spider, quittez le shell Scrapy en saisissant ::

    quit()

Puis exécutez ::

   scrapy crawl quotes

Il devrait maintenant afficher les données extraites dans le log ::

    2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 https://quotes.toscrape.com/page/1/>
    {'tags': ['life', 'love'], 'author': 'André Gide', 'text': '“It is better to be hated for what you are than to be loved for what you are not.”'}
    2016-09-19 18:57:19 [scrapy.core.scraper] DEBUG: Scraped from <200 https://quotes.toscrape.com/page/1/>
    {'tags': ['edison', 'failure', 'inspirational', 'paraphrased'], 'author': 'Thomas A. Edison', 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}


.. _storing-data:

Stocker les données extraites
=============================

La façon la plus simple de stocker les données extraites consiste à utiliser
les :ref:`Feed exports <topics-feed-exports>`, avec la commande suivante ::

    scrapy crawl quotes -O quotes.json

Cela générera un fichier ``quotes.json`` contenant tous les items extraits,
sérialisés au format `JSON`_.

L'option de ligne de commande ``-O`` écrase tout fichier existant ; utilisez
plutôt ``-o`` pour ajouter du nouveau contenu à un fichier existant. Cependant,
ajouter du contenu à un fichier JSON rend le contenu du fichier invalide en
tant que JSON. Lorsque vous ajoutez du contenu à un fichier, pensez à utiliser
un autre format de sérialisation, comme `JSON Lines`_ ::

    scrapy crawl quotes -o quotes.jsonl

Le format `JSON Lines`_ est utile car il fonctionne comme un flux : vous pouvez
facilement y ajouter de nouveaux enregistrements. Il n'a pas le même problème
que JSON lorsque vous lancez le spider deux fois. De plus, comme chaque
enregistrement occupe une ligne distincte, vous pouvez traiter de gros fichiers
sans avoir à tout charger en mémoire ; des outils comme `JQ`_ aident à le faire
en ligne de commande.

Dans les petits projets (comme celui de ce tutoriel), cela devrait suffire.
Cependant, si vous souhaitez faire des choses plus complexes avec les items
extraits, vous pouvez écrire un :ref:`Item Pipeline <topics-item-pipeline>`. Un
fichier réservé aux Item Pipelines a été préparé pour vous lors de la création
du projet, dans ``tutorial/pipelines.py``. Vous n'avez toutefois pas besoin
d'implémenter d'item pipeline si vous voulez simplement stocker les items
extraits.

.. _JSON Lines: https://jsonlines.org
.. _JQ: https://stedolan.github.io/jq


Suivre les liens
================

Supposons qu'au lieu de simplement extraire le contenu des deux premières pages
de https://quotes.toscrape.com, vous vouliez les citations de toutes les pages
du site.

Maintenant que vous savez extraire des données des pages, voyons comment suivre
les liens qu'elles contiennent.

La première chose à faire est d'extraire le lien vers la page que nous voulons
suivre. En examinant notre page, nous voyons qu'il existe un lien vers la page
suivante, avec le balisage suivant :

.. code-block:: html

    <ul class="pager">
        <li class="next">
            <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
        </li>
    </ul>

Nous pouvons essayer de l'extraire dans le shell :

>>> response.css('li.next a').get()
'<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'

Cela récupère l'élément ancre, mais nous voulons l'attribut ``href``. Pour cela,
Scrapy prend en charge une extension CSS qui permet de sélectionner le contenu
d'un attribut, comme ceci :

.. code-block:: pycon

    >>> response.css("li.next a::attr(href)").get()
    '/page/2/'

Il existe aussi une propriété ``attrib`` (voir :ref:`selecting-attributes` pour
en savoir plus) :

.. code-block:: pycon

    >>> response.css("li.next a").attrib["href"]
    '/page/2/'

Voyons maintenant notre spider, modifié pour suivre récursivement le lien vers
la page suivante et en extraire les données :

.. code-block:: python

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            "https://quotes.toscrape.com/page/1/",
        ]

        def parse(self, response):
            for quote in response.css("div.quote"):
                yield {
                    "text": quote.css("span.text::text").get(),
                    "author": quote.css("small.author::text").get(),
                    "tags": quote.css("div.tags a.tag::text").getall(),
                }

            next_page = response.css("li.next a::attr(href)").get()
            if next_page is not None:
                next_page = response.urljoin(next_page)
                yield scrapy.Request(next_page, callback=self.parse)


Maintenant, après avoir extrait les données, la méthode ``parse()`` cherche le
lien vers la page suivante, construit une URL absolue complète à l'aide de la
méthode :meth:`~scrapy.http.Response.urljoin` (car les liens peuvent être
relatifs) et produit une nouvelle requête vers la page suivante, en
s'enregistrant elle-même comme callback pour gérer l'extraction des données de
la page suivante et poursuivre ainsi le parcours de toutes les pages.

Ce que vous voyez ici est le mécanisme de suivi de liens de Scrapy : lorsque
vous produisez (avec ``yield``) une Request dans une méthode callback, Scrapy
planifie l'envoi de cette requête et enregistre une méthode callback à
exécuter lorsque cette requête est terminée.

Grâce à cela, vous pouvez construire des crawlers complexes qui suivent les
liens selon des règles que vous définissez, et extraire différents types de
données selon la page visitée.

Dans notre exemple, cela crée une sorte de boucle qui suit tous les liens vers
la page suivante jusqu'à ce qu'il n'y en ait plus -- pratique pour parcourir
des blogs, des forums et d'autres sites paginés.


.. _response-follow-example:

Un raccourci pour créer des Requests
------------------------------------

Comme raccourci pour créer des objets Request, vous pouvez utiliser
:meth:`response.follow <scrapy.http.TextResponse.follow>` :

.. code-block:: python

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            "https://quotes.toscrape.com/page/1/",
        ]

        def parse(self, response):
            for quote in response.css("div.quote"):
                yield {
                    "text": quote.css("span.text::text").get(),
                    "author": quote.css("span small::text").get(),
                    "tags": quote.css("div.tags a.tag::text").getall(),
                }

            next_page = response.css("li.next a::attr(href)").get()
            if next_page is not None:
                yield response.follow(next_page, callback=self.parse)

Contrairement à :class:`scrapy.Request`, ``response.follow`` prend directement
en charge les URL relatives : pas besoin d'appeler urljoin. Notez que
``response.follow`` se contente de renvoyer une instance de Request ; vous
devez toujours produire (avec ``yield``) cette Request.

.. skip: start

Vous pouvez aussi passer un sélecteur à ``response.follow`` au lieu d'une chaîne
de caractères ; ce sélecteur doit extraire les attributs nécessaires :

.. code-block:: python

    for href in response.css("ul.pager a::attr(href)"):
        yield response.follow(href, callback=self.parse)

Pour les éléments ``<a>``, il existe un raccourci : ``response.follow`` utilise
automatiquement leur attribut href. Le code peut donc être raccourci
davantage :

.. code-block:: python

    for a in response.css("ul.pager a"):
        yield response.follow(a, callback=self.parse)

Pour créer plusieurs requêtes à partir d'un itérable, vous pouvez utiliser
plutôt :meth:`response.follow_all <scrapy.http.TextResponse.follow_all>` :

.. code-block:: python

    anchors = response.css("ul.pager a")
    yield from response.follow_all(anchors, callback=self.parse)

ou, en le raccourcissant encore :

.. code-block:: python

    yield from response.follow_all(css="ul.pager a", callback=self.parse)

.. skip: end


Autres exemples et modèles
--------------------------

Voici un autre spider qui illustre les callbacks et le suivi de liens, cette
fois pour extraire des informations sur les auteurs :

.. code-block:: python

    import scrapy


    class AuthorSpider(scrapy.Spider):
        name = "author"

        start_urls = ["https://quotes.toscrape.com/"]

        def parse(self, response):
            author_page_links = response.css(".author + a")
            yield from response.follow_all(author_page_links, self.parse_author)

            pagination_links = response.css("li.next a")
            yield from response.follow_all(pagination_links, self.parse)

        def parse_author(self, response):
            def extract_with_css(query):
                return response.css(query).get(default="").strip()

            yield {
                "name": extract_with_css("h3.author-title::text"),
                "birthdate": extract_with_css(".author-born-date::text"),
                "bio": extract_with_css(".author-description::text"),
            }

Ce spider démarre depuis la page principale, suit tous les liens vers les pages
des auteurs en appelant le callback ``parse_author`` pour chacun d'eux, et
suit aussi les liens de pagination avec le callback ``parse``, comme nous
l'avons vu précédemment.

Ici, nous passons les callbacks à
:meth:`response.follow_all <scrapy.http.TextResponse.follow_all>` en tant
qu'arguments positionnels pour raccourcir le code ; cela fonctionne aussi pour
:class:`~scrapy.Request`.

Le callback ``parse_author`` définit une fonction utilitaire pour extraire et
nettoyer les données issues d'une requête CSS, et produit le dict Python
contenant les données de l'auteur.

Ce spider montre un autre point intéressant : même s'il y a de nombreuses
citations du même auteur, nous n'avons pas à nous soucier de visiter plusieurs
fois la même page d'auteur. Par défaut, Scrapy filtre les requêtes dupliquées
vers des URL déjà visitées, ce qui évite le problème de solliciter trop les
serveurs à cause d'une erreur de programmation. Cela peut être configuré avec
le paramètre :setting:`DUPEFILTER_CLASS`.

J'espère qu'à ce stade vous avez une bonne compréhension de l'utilisation du
mécanisme de suivi de liens et des callbacks avec Scrapy.

Comme autre exemple de spider exploitant le mécanisme de suivi de liens,
consultez la classe :class:`~scrapy.spiders.CrawlSpider`, un spider générique
qui implémente un petit moteur de règles sur lequel vous pouvez vous appuyer
pour écrire vos crawlers.

Par ailleurs, un motif courant consiste à construire un item avec des données
provenant de plusieurs pages, en utilisant une :ref:`astuce pour transmettre des
données supplémentaires aux callbacks <callback-data>`.


Utiliser les arguments des spiders
==================================

Vous pouvez fournir des arguments de ligne de commande à vos spiders en
utilisant l'option ``-a`` lorsque vous les lancez ::

    scrapy crawl quotes -O quotes-humor.json -a tag=humor

Ces arguments sont transmis à la méthode ``__init__`` du spider et deviennent
par défaut des attributs du spider.

Dans cet exemple, la valeur fournie pour l'argument ``tag`` sera disponible via
``self.tag``. Vous pouvez l'utiliser pour que votre spider ne récupère que les
citations portant un tag précis, en construisant l'URL à partir de l'argument :

.. code-block:: python

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        async def start(self):
            url = "https://quotes.toscrape.com/"
            tag = getattr(self, "tag", None)
            if tag is not None:
                url = url + "tag/" + tag
            yield scrapy.Request(url, self.parse)

        def parse(self, response):
            for quote in response.css("div.quote"):
                yield {
                    "text": quote.css("span.text::text").get(),
                    "author": quote.css("small.author::text").get(),
                }

            next_page = response.css("li.next a::attr(href)").get()
            if next_page is not None:
                yield response.follow(next_page, self.parse)


Si vous passez l'argument ``tag=humor`` à ce spider, vous remarquerez qu'il ne
visitera que les URL du tag ``humor``, comme par exemple
``https://quotes.toscrape.com/tag/humor``.

Vous pouvez :ref:`en savoir plus sur la gestion des arguments de spider ici
<spiderargs>`.

Prochaines étapes
=================

Ce tutoriel n'a couvert que les bases de Scrapy, mais il existe bien d'autres
fonctionnalités qui n'ont pas été abordées ici. Consultez la section
:ref:`topics-whatelse` du chapitre :ref:`intro-overview` pour un aperçu rapide
des plus importantes.

Vous pouvez poursuivre avec la section :ref:`section-basics` pour en savoir
plus sur l'outil en ligne de commande, les spiders, les sélecteurs et d'autres
sujets que le tutoriel n'a pas couverts, comme la modélisation des données
extraites. Si vous préférez jouer avec un projet d'exemple, consultez la
section :ref:`intro-examples`.

Pour transformer des pages entières en texte propre ou en markdown, ou pour
convertir les chaînes de caractères que vous sélectionnez en dates, prix ou
autres valeurs, consultez :ref:`extraction`.

Si vous utilisez un agent de programmation (coding agent), consultez
:ref:`agents`.

.. _JSON: https://en.wikipedia.org/wiki/JSON
