.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/spiders.rst`.

.. _topics-spiders:

=======
Spiders
=======

Les spiders sont des classes qui définissent comment un site, ou un groupe de
sites, est scrapé : quelles requêtes envoyer, et comment analyser leurs
réponses pour en extraire des données et pour envoyer des requêtes
supplémentaires.

Un crawl se déroule ainsi :

1.  Scrapy itère sur la méthode :meth:`~scrapy.Spider.start` du spider pour
    obtenir les requêtes initiales. Par défaut, cette méthode produit (avec
    ``yield``) un objet :class:`~scrapy.Request` pour chaque URL de
    :attr:`~scrapy.Spider.start_urls`, avec :meth:`~scrapy.Spider.parse` comme
    :ref:`callback <callbacks>`.

2.  Scrapy télécharge chaque requête et appelle son callback avec la
    :class:`~scrapy.http.Response` obtenue.

3.  Les callbacks analysent la réponse, généralement à l'aide des
    :ref:`sélecteurs <topics-selectors>`, puis retournent ou produisent (avec
    ``yield``) des :ref:`objets item <topics-items>` contenant les données
    extraites ainsi que des objets :class:`~scrapy.Request` pour poursuivre le
    crawl ; ces derniers retournent à l'étape 2. Voir :ref:`callback-output`.

4.  Les items passent par les :ref:`item pipelines <topics-item-pipeline>`, et
    sont généralement enregistrés grâce aux :ref:`topics-feed-exports`.

Scrapy inclut différentes classes de spiders pour différents usages, décrites
ci-dessous.

.. _topics-spiders-ref:

scrapy.Spider
=============

.. autoclass:: scrapy.Spider

   .. attribute:: name

       Une chaîne de caractères qui définit le nom de ce spider. Le nom du
       spider est ce qui permet à Scrapy de le localiser (et de l'instancier),
       il doit donc être unique. Cependant, rien ne vous empêche d'instancier
       plusieurs instances du même spider. C'est l'attribut de spider le plus
       important, et il est obligatoire.

       Si le spider scrape un seul domaine, une pratique courante consiste à
       nommer le spider d'après le domaine, avec ou sans le `TLD`_. Ainsi, par
       exemple, un spider qui parcourt ``mywebsite.com`` s'appellerait souvent
       ``mywebsite``.

   .. attribute:: allowed_domains

       Une liste facultative de chaînes de caractères contenant les domaines que
       ce spider est autorisé à parcourir. Les requêtes pour des URLs
       n'appartenant pas aux noms de domaine indiqués dans cette liste (ni à
       leurs sous-domaines) ne seront pas suivies si
       :class:`~scrapy.downloadermiddlewares.offsite.OffsiteMiddleware` est
       activé.

       .. versionchanged:: 2.18.0
          Les modifications de cet attribut pendant un crawl sont désormais
          prises en compte.

       Supposons que votre URL cible soit ``https://www.example.com/1.html`` :
       ajoutez alors ``'example.com'`` à la liste.

       Vous pouvez modifier cet attribut pendant l'exécution du spider, par
       exemple pour autoriser des domaines que vous ne découvrez qu'à partir
       d'une réponse précédente. La modification affecte les requêtes
       planifiées après elle.

   .. autoattribute:: start_urls

   .. attribute:: custom_settings

      Un dictionnaire de paramètres qui remplaceront ceux de la configuration
      globale du projet lors de l'exécution de ce spider. Il doit être défini
      comme attribut de classe, car les paramètres sont mis à jour avant
      l'instanciation.

      Pour la liste des paramètres intégrés disponibles, voir :
      :ref:`topics-settings-ref`.

   .. attribute:: crawler

      Cet attribut est défini par la méthode de classe :meth:`from_crawler`
      après l'initialisation de la classe, et pointe vers l'objet
      :class:`~scrapy.crawler.Crawler` auquel cette instance de spider est
      liée.

      Les crawlers encapsulent de nombreux composants du projet pour offrir un
      point d'accès unique (comme les extensions, les middlewares, les
      gestionnaires de signaux, etc.). Voir :ref:`topics-api-crawler` pour en
      savoir plus.

   .. attribute:: settings

      La configuration d'exécution de ce spider. C'est une instance de
      :class:`~scrapy.settings.Settings` ; voir la rubrique
      :ref:`topics-settings` pour une introduction détaillée à ce sujet.

   .. attribute:: logger

      Un logger Python créé avec le :attr:`name` du spider. Vous pouvez
      l'utiliser pour envoyer des messages de log, comme décrit dans
      :ref:`topics-logging-from-spiders`.

   .. attribute:: state

      Un dictionnaire que vous pouvez utiliser pour conserver un état du spider
      entre plusieurs exécutions par lots (batches). Voir
      :ref:`topics-keeping-persistent-state-between-batches` pour en savoir
      plus.

   .. method:: from_crawler(crawler, *args, **kwargs)

       C'est la méthode de classe utilisée par Scrapy pour créer vos spiders.

       Vous n'aurez probablement pas besoin de la redéfinir directement, car
       l'implémentation par défaut sert de relais vers la méthode
       :meth:`__init__`, en l'appelant avec les arguments ``args`` et les
       arguments nommés ``kwargs`` donnés.

       Néanmoins, cette méthode définit les attributs :attr:`crawler` et
       :attr:`settings` dans la nouvelle instance, afin qu'ils soient
       accessibles plus tard dans le code du spider.

       Les paramètres de ``crawler.settings`` peuvent être modifiés dans cette
       méthode, ce qui est pratique si vous voulez les modifier en fonction
       d'arguments. En conséquence, ces paramètres ne sont pas les valeurs
       finales, car ils peuvent être modifiés plus tard, par exemple par des
       :ref:`add-ons <topics-addons>`. Pour la même raison, la plupart des
       attributs de :class:`~scrapy.crawler.Crawler` ne sont pas encore
       initialisés à ce stade.

       Les paramètres finaux et les attributs initialisés de
       :class:`~scrapy.crawler.Crawler` sont disponibles dans la méthode
       :meth:`start`, dans les gestionnaires du signal
       :signal:`engine_started` et après.

       :param crawler: crawler auquel le spider sera lié
       :type crawler: :class:`~scrapy.crawler.Crawler` instance

       :param args: arguments passés à la méthode :meth:`__init__`
       :type args: list

       :param kwargs: arguments nommés passés à la méthode :meth:`__init__`
       :type kwargs: dict

   .. classmethod:: update_settings(settings)

       La méthode ``update_settings()`` sert à modifier les paramètres du
       spider ; elle est appelée lors de l'initialisation d'une instance de
       spider.

       Elle prend en paramètre un objet :class:`~scrapy.settings.Settings` et
       peut ajouter ou mettre à jour les valeurs de configuration du spider.
       C'est une méthode de classe : elle est donc appelée sur la classe
       :class:`~scrapy.Spider` et permet à toutes les instances du spider de
       partager la même configuration.

       Les paramètres propres à un spider peuvent être définis dans
       :attr:`~scrapy.Spider.custom_settings`, mais utiliser
       ``update_settings()`` vous permet d'ajouter, de retirer ou de modifier
       des paramètres de façon dynamique, en fonction d'autres paramètres,
       d'attributs du spider ou d'autres facteurs, et d'utiliser des priorités
       de paramètres autres que ``'spider'``. De plus, il est facile
       d'étendre ``update_settings()`` dans une sous-classe en la redéfinissant,
       alors que faire de même avec :attr:`~scrapy.Spider.custom_settings`
       peut être difficile.

       Par exemple, supposons qu'un spider doive modifier :setting:`FEEDS` :

       .. code-block:: python

           import scrapy


           class MySpider(scrapy.Spider):
               name = "myspider"
               custom_feed = {
                   "/home/user/documents/items.json": {
                       "format": "json",
                       "indent": 4,
                   }
               }

               @classmethod
               def update_settings(cls, settings):
                   super().update_settings(settings)
                   settings.setdefault("FEEDS", {}).update(cls.custom_feed)

   .. automethod:: start

   .. automethod:: parse

   .. method:: closed(reason)

       Appelée lorsque le spider se ferme. Cette méthode fournit un raccourci
       vers signals.connect() pour le signal :signal:`spider_closed`.

Voyons un exemple :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "example.com"
        allowed_domains = ["example.com"]
        start_urls = [
            "http://www.example.com/1.html",
            "http://www.example.com/2.html",
            "http://www.example.com/3.html",
        ]

        def parse(self, response):
            self.logger.info("A response from %s just arrived!", response.url)

Retourner plusieurs requêtes (Requests) et items depuis un seul callback :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "example.com"
        allowed_domains = ["example.com"]
        start_urls = [
            "http://www.example.com/1.html",
            "http://www.example.com/2.html",
            "http://www.example.com/3.html",
        ]

        def parse(self, response):
            for h3 in response.xpath("//h3").getall():
                yield {"title": h3}

            for href in response.xpath("//a/@href").getall():
                yield scrapy.Request(response.urljoin(href), self.parse)

Au lieu de :attr:`~.start_urls`, vous pouvez utiliser directement
:meth:`~scrapy.Spider.start` ; et pour donner plus de structure aux données,
vous pouvez utiliser des objets :class:`~scrapy.Item` :

.. skip: next
.. code-block:: python

    import scrapy
    from myproject.items import MyItem


    class MySpider(scrapy.Spider):
        name = "example.com"
        allowed_domains = ["example.com"]

        async def start(self):
            yield scrapy.Request("http://www.example.com/1.html", self.parse)
            yield scrapy.Request("http://www.example.com/2.html", self.parse)
            yield scrapy.Request("http://www.example.com/3.html", self.parse)

        def parse(self, response):
            for h3 in response.xpath("//h3").getall():
                yield MyItem(title=h3)

            for href in response.xpath("//a/@href").getall():
                yield scrapy.Request(response.urljoin(href), self.parse)

.. _spiderargs:

Arguments du spider
===================

Les spiders peuvent recevoir des arguments qui modifient leur comportement.
Parmi les usages courants des arguments de spider : définir les URLs de départ
ou restreindre le crawl à certaines sections du site. Mais ils peuvent servir à
configurer n'importe quelle fonctionnalité du spider.

Les arguments de spider sont passés via la commande :command:`crawl` grâce à
l'option ``-a``. Par exemple ::

    scrapy crawl myspider -a category=electronics

Les spiders peuvent accéder aux arguments dans leurs méthodes `__init__` :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"

        def __init__(self, category=None, *args, **kwargs):
            super().__init__(*args, **kwargs)
            self.start_urls = [f"http://www.example.com/categories/{category}"]
            # ...

La méthode ``__init__`` par défaut prend tous les arguments du spider et les
copie dans le spider sous forme d'attributs. L'exemple ci-dessus peut aussi
s'écrire de la façon suivante :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"

        async def start(self):
            yield scrapy.Request(f"http://www.example.com/categories/{self.category}")

Si vous :ref:`lancez Scrapy depuis un script <run-from-script>`, vous pouvez
indiquer les arguments du spider lors de l'appel à
:meth:`CrawlerProcess.crawl <scrapy.crawler.CrawlerProcess.crawl>` ou à
:meth:`CrawlerRunner.crawl <scrapy.crawler.CrawlerRunner.crawl>` :

.. skip: next
.. code-block:: python

    process = CrawlerProcess()
    process.crawl(MySpider, category="electronics")

Gardez à l'esprit que les arguments de spider ne sont que des chaînes de
caractères. Le spider n'effectue aucune analyse par lui-même. Si vous définissiez
l'attribut ``start_urls`` depuis la ligne de commande, vous devriez l'analyser
vous-même pour en faire une liste, avec quelque chose comme
:func:`ast.literal_eval` ou :func:`json.loads`, puis le définir comme
attribut. Sinon, vous provoqueriez une itération sur une chaîne ``start_urls``
(un piège Python très courant), ce qui ferait que chaque caractère serait vu
comme une URL distincte.

Les arguments de spider peuvent aussi être passés via l'API ``schedule.json``
de Scrapyd. Voir la `documentation de Scrapyd`_.

.. _spiderargs-scrapy-spider-metadata:

Paramètres scrapy-spider-metadata
---------------------------------

Une autre façon de passer des arguments à un spider consiste à utiliser la
bibliothèque `scrapy-spider-metadata`_.

Elle permet aux spiders Scrapy de définir, valider, documenter et pré-traiter
leurs arguments sous forme de modèles Pydantic.

L'exemple montre comment définir des paramètres typés, où un argument de type
chaîne est automatiquement converti en entier :

.. code-block:: python

    import scrapy
    from pydantic import BaseModel
    from scrapy_spider_metadata import Args


    class MyParams(BaseModel):
        pages: int


    class BookSpider(Args[MyParams], scrapy.Spider):
        name = "bookspider"
        start_urls = ["http://books.toscrape.com/catalogue"]

        async def start(self):
            for start_url in self.start_urls:
                for index in range(1, self.args.pages + 1):
                    yield scrapy.Request(f"{start_url}/page-{index}.html")

        def parse(self, response):
            book_links = response.css("article.product_pod h3 a::attr(href)").getall()
            for book_link in book_links:
                yield response.follow(book_link, self.parse_book)

        def parse_book(self, response):
            yield {
                "title": response.css("h1::text").get(),
                "price": response.css("p.price_color::text").get(),
            }

Ce spider peut être lancé depuis la ligne de commande ::

    scrapy crawl bookspider -a pages=2

.. _start-requests:

Requêtes de démarrage
=====================

Les **requêtes de démarrage** (start requests) sont des objets
:class:`~scrapy.Request` produits (avec ``yield``) par la méthode
:meth:`~scrapy.Spider.start` d'un spider, ou par la méthode
:meth:`~scrapy.spidermiddlewares.SpiderMiddleware.process_start` d'un
:ref:`spider middleware <topics-spider-middleware>`.

.. seealso:: :ref:`start-request-order`

.. _start-requests-lazy:

Retarder l'itération des requêtes de démarrage
----------------------------------------------

Scrapy itère sur :meth:`~scrapy.Spider.start` aussi vite que la méthode produit
ses valeurs : toutes les requêtes de démarrage arrivent donc tôt dans le
scheduler au cours du crawl, quel que soit leur nombre. Pour réduire au
minimum le nombre de requêtes présentes dans le scheduler à un instant donné,
et donc la consommation de ressources (mémoire, ou disque si vous utilisez
:setting:`JOBDIR`), redéfinissez :meth:`~scrapy.Spider.start` afin de mettre son
itération en pause tant qu'il y a des requêtes planifiées :

.. code-block:: python

    async def start(self):
        async for item_or_request in super().start():
            if self.crawler.engine.needs_backout():
                await self.crawler.signals.wait_for(signals.scheduler_empty)
            yield item_or_request

.. _start-error:

Gérer les erreurs de démarrage
------------------------------

Une exception levée par :meth:`~scrapy.Spider.start` met fin à son itération :
les items et requêtes de démarrage restants ne sont donc jamais envoyés.
Scrapy enregistre l'exception dans les logs, envoie le signal
:signal:`spider_error` et, une fois les requêtes déjà planifiées terminées,
ferme le spider avec le :stat:`finish_reason` ``start_error``.

.. versionchanged:: 2.18.0
   Le motif de fermeture était auparavant ``finished``, et ni le signal
   :signal:`spider_error` ni la statistique :stat:`spider_exceptions/count`
   ne signalaient l'exception.

Pour que l'itération continue, interceptez vous-même l'exception :

.. code-block:: python

    async def start(self):
        for url in self.start_urls:
            try:
                request = Request(url)
            except ValueError:
                self.logger.exception(f"Skipping start URL {url}")
            else:
                yield request

Pour au contraire arrêter le crawl et choisir votre propre
:stat:`finish_reason`, levez :exc:`~scrapy.exceptions.CloseSpider`.

.. _builtin-spiders:

Spiders génériques
==================

Scrapy est livré avec quelques spiders génériques utiles, dont vous pouvez
faire hériter vos propres spiders. Leur but est de fournir des fonctionnalités
pratiques pour quelques cas de scraping courants, comme suivre tous les liens
d'un site selon certaines règles, parcourir un site à partir de `Sitemaps`_, ou
analyser un feed XML/CSV.

Pour les exemples utilisés dans les spiders suivants, nous supposerons que vous
avez un projet avec un ``TestItem`` déclaré dans un module ``myproject.items`` :

.. code-block:: python

    from dataclasses import dataclass


    @dataclass
    class TestItem:
        id: str | None = None
        name: str | None = None
        description: str | None = None


.. currentmodule:: scrapy.spiders

CrawlSpider
-----------

.. class:: CrawlSpider

   C'est le spider le plus couramment utilisé pour parcourir des sites web
   ordinaires, car il fournit un mécanisme pratique pour suivre des liens en
   définissant un ensemble de règles. Il n'est peut-être pas le mieux adapté à
   vos sites ou à votre projet, mais il est assez générique pour plusieurs cas :
   vous pouvez donc partir de lui et le redéfinir au besoin pour obtenir des
   fonctionnalités plus spécifiques, ou simplement implémenter votre propre
   spider.

   En plus des attributs hérités de Spider (que vous devez renseigner), cette
   classe prend en charge un nouvel attribut :

   .. attribute:: rules

       C'est une liste d'un (ou plusieurs) objets :class:`Rule`. Chaque
       :class:`Rule` définit un comportement particulier pour parcourir le
       site. Les objets Rule sont décrits ci-dessous. Si plusieurs règles
       correspondent au même lien, la première sera utilisée, selon l'ordre
       dans lequel elles sont définies dans cet attribut.

   .. reqmeta:: rule

   Les requêtes générées à partir de :attr:`rules` portent, dans leur clé
   ``rule`` de :attr:`Request.meta <scrapy.Request.meta>`, l'indice de la règle
   correspondante dans :attr:`rules`. :class:`CrawlSpider` a besoin de cette
   clé pour aiguiller la réponse vers la bonne règle ; la copier dans une
   requête générée par une autre règle envoie donc la réponse vers le mauvais
   callback.

   Ce spider expose aussi une méthode que vous pouvez redéfinir :

   .. method:: parse_start_url(response, **kwargs)

      Cette méthode est appelée pour chaque réponse produite pour les URLs de
      l'attribut ``start_urls`` du spider. Elle permet d'analyser les réponses
      initiales et doit retourner soit un :ref:`objet item <topics-items>`,
      soit un objet :class:`~scrapy.Request`, soit un itérable contenant l'un
      ou l'autre.

Règles de crawl
~~~~~~~~~~~~~~~

.. reqmeta:: link_text

.. autoclass:: Rule

   ``link_extractor`` est un objet :ref:`Link Extractor <topics-link-extractors>`
   qui définit comment les liens seront extraits de chaque page parcourue.
   Chaque lien produit servira à générer un objet :class:`~scrapy.Request`, qui
   contiendra le texte du lien dans son dictionnaire ``meta`` (sous la clé
   ``link_text``). S'il est omis, un link extractor par défaut créé sans
   argument sera utilisé, ce qui aboutit à l'extraction de tous les liens.

   ``callback`` est un callable ou une chaîne de caractères (auquel cas une
   méthode de l'objet spider portant ce nom sera utilisée) à appeler pour
   chaque lien extrait avec le link extractor indiqué. Ce callback reçoit une
   :class:`~scrapy.http.Response` comme premier argument et doit retourner
   soit une instance unique, soit un itérable d'
   :ref:`objets item <topics-items>` et/ou d'objets :class:`~scrapy.Request`
   (ou de n'importe quelle sous-classe de ceux-ci). Comme mentionné plus haut,
   l'objet :class:`~scrapy.http.Response` reçu contiendra le texte du lien qui
   a produit la :class:`~scrapy.Request` dans son dictionnaire ``meta`` (sous
   la clé ``link_text``).

   ``cb_kwargs`` est un dictionnaire contenant les arguments nommés à passer à
   la fonction callback.

   ``follow`` est un booléen qui indique si les liens extraits avec cette règle
   doivent être suivis à partir de chaque réponse. Si ``callback`` vaut None,
   ``follow`` vaut ``True`` par défaut ; sinon, il vaut ``False`` par défaut.

   ``process_links`` est un callable, ou une chaîne de caractères (auquel cas
   une méthode de l'objet spider portant ce nom sera utilisée), qui sera appelé
   pour chaque liste de liens extraits de chaque réponse avec le
   ``link_extractor`` indiqué. Il sert principalement à filtrer.

   ``process_request`` est un callable (ou une chaîne de caractères, auquel cas
   une méthode de l'objet spider portant ce nom sera utilisée) qui sera appelé
   pour chaque :class:`~scrapy.Request` extraite par cette règle. Ce callable
   doit prendre cette requête comme premier argument et la
   :class:`~scrapy.http.Response` dont la requête est issue comme second
   argument. Il doit retourner un objet ``Request`` ou ``None`` (pour écarter la
   requête).

   Utilisez ``process_request`` pour définir la
   :attr:`~scrapy.Request.priority` des requêtes générées par une règle, par
   exemple ``process_request=lambda request,
   response: request.replace(priority=10)``.

   ``errback`` est un callable ou une chaîne de caractères (auquel cas une
   méthode de l'objet spider portant ce nom sera utilisée) à appeler si une
   exception est levée pendant le traitement d'une requête générée par la
   règle. Il reçoit une instance de
   :class:`Twisted Failure <twisted.python.failure.Failure>` comme premier
   paramètre.

   ``name`` est une chaîne de caractères qui identifie la règle, afin de
   servir de cible au paramètre ``from_rules`` d'autres règles.

   ``from_rules`` est une chaîne de caractères, ou un itérable de chaînes de
   caractères, contenant le ``name`` d'autres règles. S'il est défini, cette
   règle n'est appliquée qu'aux réponses atteintes via l'une de ces règles, au
   lieu de l'être à toutes les réponses.

   .. versionadded:: VERSION
      Les paramètres ``name`` et ``from_rules``.

   .. warning:: En raison de son implémentation interne, vous devez définir
      explicitement les callbacks des nouvelles requêtes lorsque vous écrivez
      des spiders basés sur :class:`CrawlSpider` ; sinon, des comportements
      inattendus peuvent se produire.

Exemple de CrawlSpider
~~~~~~~~~~~~~~~~~~~~~~

Voyons maintenant un exemple de CrawlSpider avec des règles :

.. code-block:: python

    from scrapy.spiders import CrawlSpider, Rule
    from scrapy.linkextractors import LinkExtractor


    class MySpider(CrawlSpider):
        name = "example.com"
        allowed_domains = ["example.com"]
        start_urls = ["http://www.example.com"]

        rules = (
            # Extract links matching 'category.php' (but not matching 'subsection.php')
            # and follow links from them (since no callback means follow=True by default).
            Rule(LinkExtractor(allow=(r"category\.php",), deny=(r"subsection\.php",))),
            # Extract links matching 'item.php' and parse them with the spider's method parse_item
            Rule(LinkExtractor(allow=(r"item\.php",)), callback="parse_item"),
        )

        def parse_item(self, response):
            self.logger.info("Hi, this is an item page! %s", response.url)
            item = {}
            item["id"] = response.xpath('//td[@id="item_id"]/text()').re(r"ID: (\d+)")
            item["name"] = response.xpath('//td[@id="item_name"]/text()').get()
            item["description"] = response.xpath(
                '//td[@id="item_description"]/text()'
            ).get()
            item["link_text"] = response.meta["link_text"]
            url = response.xpath('//td[@id="additional_data"]/@href').get()
            return response.follow(
                url, self.parse_additional_page, cb_kwargs=dict(item=item)
            )

        def parse_additional_page(self, response, item):
            item["additional_data"] = response.xpath(
                '//p[@id="additional_data"]/text()'
            ).get()
            return item


Ce spider commencerait par parcourir la page d'accueil de example.com, en
collectant les liens de catégories et les liens d'items, puis analyserait ces
derniers avec la méthode ``parse_item``. Pour chaque réponse d'item, des
données seraient extraites du HTML à l'aide de XPath, et un dictionnaire serait
rempli avec ces données.

XMLFeedSpider
-------------

.. class:: XMLFeedSpider

    XMLFeedSpider est conçu pour analyser des feeds XML en les parcourant nœud
    par nœud, selon le nom d'un nœud donné. L'itérateur peut être choisi parmi :
    ``iternodes``, ``xml`` et ``html``. Il est recommandé d'utiliser
    l'itérateur ``iternodes`` pour des raisons de performances, puisque les
    itérateurs ``xml`` et ``html`` génèrent le DOM entier d'un seul coup afin de
    l'analyser. Cependant, utiliser ``html`` comme itérateur peut être utile
    pour analyser du XML dont le balisage est défectueux.

    Pour définir l'itérateur et le nom de la balise, vous devez définir les
    attributs de classe suivants :

    .. attribute:: iterator

        Une chaîne de caractères qui définit l'itérateur à utiliser. Ce peut
        être :

           - ``'iternodes'`` - un itérateur rapide basé sur ``lxml``

           - ``'html'`` - un itérateur qui utilise :class:`~scrapy.Selector`.
             Gardez à l'esprit qu'il s'appuie sur une analyse du DOM et doit
             charger tout le DOM en mémoire, ce qui peut poser problème pour
             de gros feeds. Il analyse aussi le feed avec un parseur HTML, qui
             peut altérer silencieusement les balises que le HTML traite comme
             des éléments vides (void elements), telles que ``<link>``, en
             supprimant leur contenu et leur balise fermante. Utilisez plutôt
             ``xml`` ou ``iternodes`` pour les feeds concernés par ce
             problème.

           - ``'xml'`` - un itérateur qui utilise :class:`~scrapy.Selector`.
             Gardez à l'esprit qu'il s'appuie sur une analyse du DOM et doit
             charger tout le DOM en mémoire, ce qui peut poser problème pour
             de gros feeds

        Sa valeur par défaut est : ``'iternodes'``.

    .. attribute:: itertag

        Une chaîne de caractères contenant le nom du nœud (ou de l'élément) sur
        lequel itérer. Exemple :

        .. code-block:: python

            itertag = "product"

    .. attribute:: namespaces

        Une liste de tuples ``(prefix, uri)`` qui définissent les espaces de
        noms (namespaces) disponibles dans le document qui sera traité par ce
        spider. Le ``prefix`` et l'``uri`` serviront à enregistrer
        automatiquement les espaces de noms grâce à la méthode
        :meth:`~scrapy.Selector.register_namespace`.

        Vous pouvez ensuite indiquer des nœuds avec espaces de noms dans
        l'attribut :attr:`itertag`.

        Exemple :

        .. code-block:: python

            from scrapy.spiders import XMLFeedSpider


            class YourSpider(XMLFeedSpider):

                namespaces = [("n", "http://www.sitemaps.org/schemas/sitemap/0.9")]
                itertag = "n:url"
                # ...

    En plus de ces nouveaux attributs, ce spider possède aussi les méthodes
    suivantes que vous pouvez redéfinir :

    .. method:: adapt_response(response)

        Une méthode qui reçoit la réponse dès son arrivée depuis le spider
        middleware, avant que le spider ne commence à l'analyser. Elle peut
        servir à modifier le corps de la réponse avant son analyse. Cette
        méthode reçoit une réponse et en retourne également une (ce peut être la
        même ou une autre).

    .. method:: parse_node(response, selector)

        Cette méthode est appelée pour les nœuds correspondant au nom de balise
        fourni (``itertag``). Elle reçoit la réponse et un
        :class:`~scrapy.Selector` pour chaque nœud. Redéfinir cette méthode est
        obligatoire. Sinon, votre spider ne fonctionnera pas. Cette méthode doit
        retourner un :ref:`objet item <topics-items>`, un objet
        :class:`~scrapy.Request`, ou un itérable contenant l'un ou l'autre.

    .. method:: process_results(response, results)

        Cette méthode est appelée pour chaque résultat (item ou requête)
        retourné par le spider, et elle est destinée à effectuer tout
        traitement de dernière minute nécessaire avant de renvoyer les
        résultats au cœur du framework, par exemple pour définir les
        identifiants des items. Elle reçoit une liste de résultats et la
        réponse qui a donné naissance à ces résultats. Elle doit retourner une
        liste de résultats (items ou requêtes).

    .. warning:: En raison de son implémentation interne, vous devez définir
       explicitement les callbacks des nouvelles requêtes lorsque vous écrivez
       des spiders basés sur :class:`XMLFeedSpider` ; sinon, des comportements
       inattendus peuvent se produire.


Exemple de XMLFeedSpider
~~~~~~~~~~~~~~~~~~~~~~~~

Ces spiders sont assez faciles à utiliser ; regardons un exemple :

.. skip: next
.. code-block:: python

    from scrapy.spiders import XMLFeedSpider
    from myproject.items import TestItem


    class MySpider(XMLFeedSpider):
        name = "example.com"
        allowed_domains = ["example.com"]
        start_urls = ["http://www.example.com/feed.xml"]
        iterator = "iternodes"  # This is actually unnecessary, since it's the default value
        itertag = "item"

        def parse_node(self, response, node):
            self.logger.info(
                "Hi, this is a <%s> node!: %s", self.itertag, "".join(node.getall())
            )

            item = TestItem()
            item.id = node.xpath("@id").get()
            item.name = node.xpath("name").get()
            item.description = node.xpath("description").get()
            return item

En gros, ce que nous avons fait ci-dessus, c'est créer un spider qui télécharge
un feed depuis les ``start_urls`` données, puis parcourt chacune de ses balises
``item``, les affiche, et stocke quelques données arbitraires dans un
:class:`~scrapy.Item`.

CSVFeedSpider
-------------

.. class:: CSVFeedSpider

   Ce spider est très similaire à XMLFeedSpider, sauf qu'il itère sur des
   lignes, au lieu de nœuds. La méthode appelée à chaque itération est
   :meth:`parse_row`.

   .. attribute:: delimiter

       Une chaîne de caractères contenant le caractère séparateur de chaque
       champ du fichier CSV. Vaut par défaut ``','`` (virgule).

   .. attribute:: quotechar

       Une chaîne de caractères contenant le caractère d'encadrement de chaque
       champ du fichier CSV. Vaut par défaut ``'"'`` (guillemet droit).

   .. attribute:: headers

       Une liste des noms de colonnes du fichier CSV.

   .. method:: parse_row(response, row)

       Reçoit une réponse et un dictionnaire (représentant chaque ligne) avec
       une clé pour chaque en-tête fourni (ou détecté) du fichier CSV. Ce spider
       offre aussi la possibilité de redéfinir les méthodes ``adapt_response``
       et ``process_results``, à des fins de pré-traitement et de
       post-traitement.

Exemple de CSVFeedSpider
~~~~~~~~~~~~~~~~~~~~~~~~

Voyons un exemple similaire au précédent, mais utilisant un
:class:`CSVFeedSpider` :

.. skip: next
.. code-block:: python

    from scrapy.spiders import CSVFeedSpider
    from myproject.items import TestItem


    class MySpider(CSVFeedSpider):
        name = "example.com"
        allowed_domains = ["example.com"]
        start_urls = ["http://www.example.com/feed.csv"]
        delimiter = ";"
        quotechar = "'"
        headers = ["id", "name", "description"]

        def parse_row(self, response, row):
            self.logger.info("Hi, this is a row!: %r", row)

            item = TestItem()
            item.id = row["id"]
            item.name = row["name"]
            item.description = row["description"]
            return item


SitemapSpider
-------------

.. class:: SitemapSpider

    SitemapSpider vous permet de parcourir un site en découvrant les URLs à
    l'aide des `Sitemaps`_.

    Il prend en charge les sitemaps imbriqués et la découverte des URLs de
    sitemaps à partir de `robots.txt`_.

    .. attribute:: sitemap_urls

        Une liste d'URLs pointant vers les sitemaps dont vous voulez parcourir
        les URLs.

        Vous pouvez aussi pointer vers un fichier `robots.txt`_ : il sera
        analysé pour en extraire les URLs de sitemaps.

    .. attribute:: sitemap_rules

        Une liste de tuples ``(regex, callback)`` où :

        * ``regex`` est une expression régulière servant à repérer les URLs
          extraites des sitemaps. ``regex`` peut être soit une chaîne (str),
          soit un objet regex compilé.

        * callback est le callback à utiliser pour traiter les URLs qui
          correspondent à l'expression régulière. ``callback`` peut être une
          chaîne de caractères (indiquant le nom d'une méthode du spider) ou
          un callable.

        Par exemple :

        .. code-block:: python

            sitemap_rules = [("/product/", "parse_product")]

        Les règles sont appliquées dans l'ordre, et seule la première qui
        correspond sera utilisée.

        Si vous omettez cet attribut, toutes les URLs trouvées dans les
        sitemaps seront traitées avec le callback ``parse``.

    .. attribute:: sitemap_follow

        Une liste d'expressions régulières (regex) désignant les sitemaps qui
        doivent être suivis. Cela ne concerne que les sites qui utilisent des
        `fichiers d'index de Sitemap`_ pointant vers d'autres fichiers de
        sitemap.

        Par défaut, tous les sitemaps sont suivis.

    .. attribute:: sitemap_alternate_links

        Indique si les liens alternatifs d'une même ``url`` doivent être suivis.
        Ce sont des liens vers le même site dans une autre langue, transmis dans
        le même bloc ``url``.

        Par exemple :

        .. code-block:: xml

            <url>
                <loc>http://example.com/</loc>
                <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
            </url>

        Avec ``sitemap_alternate_links`` activé, cela récupérerait les deux URLs.
        Avec ``sitemap_alternate_links`` désactivé, seule ``http://example.com/``
        serait récupérée.

        Par défaut, ``sitemap_alternate_links`` est désactivé.

    .. method:: sitemap_filter(entries)

        C'est une fonction de filtre que vous pouvez redéfinir pour sélectionner
        des entrées de sitemap en fonction de leurs attributs.

        Par exemple :

        .. code-block:: xml

            <url>
                <loc>http://example.com/</loc>
                <lastmod>2005-01-01</lastmod>
            </url>

        Nous pouvons définir une fonction ``sitemap_filter`` pour filtrer les
        ``entries`` par date :

        .. code-block:: python

            from datetime import datetime
            from scrapy.spiders import SitemapSpider


            class FilteredSitemapSpider(SitemapSpider):
                name = "filtered_sitemap_spider"
                allowed_domains = ["example.com"]
                sitemap_urls = ["http://example.com/sitemap.xml"]

                def sitemap_filter(self, entries):
                    for entry in entries:
                        date_time = datetime.strptime(entry["lastmod"], "%Y-%m-%d")
                        if date_time.year >= 2005:
                            yield entry

        Cela ne récupérerait que les ``entries`` modifiées en 2005 et les années
        suivantes.

        Les entrées (entries) sont des objets dict extraits du document sitemap.
        Généralement, la clé est le nom de la balise et la valeur est le texte
        qu'elle contient.

        Il est important de noter que :

        - comme l'attribut loc est obligatoire, les entrées sans cette balise
          sont écartées
        - les liens alternatifs sont stockés dans une liste sous la clé
          ``alternate`` (voir ``sitemap_alternate_links``)
        - les espaces de noms sont supprimés : les balises lxml nommées
          ``{namespace}tagname`` deviennent simplement ``tagname``

        Si vous omettez cette méthode, toutes les entrées trouvées dans les
        sitemaps seront traitées, en respectant les autres attributs et leurs
        réglages.


Exemples de SitemapSpider
~~~~~~~~~~~~~~~~~~~~~~~~~

Exemple le plus simple : traiter toutes les URLs découvertes via les sitemaps
avec le callback ``parse`` :

.. code-block:: python

    from scrapy.spiders import SitemapSpider


    class MySpider(SitemapSpider):
        sitemap_urls = ["http://www.example.com/sitemap.xml"]

        def parse(self, response):
            pass  # ... scrape item here ...

Traiter certaines URLs avec un callback donné et d'autres URLs avec un callback
différent :

.. code-block:: python

    from scrapy.spiders import SitemapSpider


    class MySpider(SitemapSpider):
        sitemap_urls = ["http://www.example.com/sitemap.xml"]
        sitemap_rules = [
            ("/product/", "parse_product"),
            ("/category/", "parse_category"),
        ]

        def parse_product(self, response):
            pass  # ... scrape product ...

        def parse_category(self, response):
            pass  # ... scrape category ...

Suivre les sitemaps définis dans le fichier `robots.txt`_ et ne suivre que les
sitemaps dont l'URL contient ``/sitemap_shop`` :

.. code-block:: python

    from scrapy.spiders import SitemapSpider


    class MySpider(SitemapSpider):
        sitemap_urls = ["http://www.example.com/robots.txt"]
        sitemap_rules = [
            ("/shop/", "parse_shop"),
        ]
        sitemap_follow = ["/sitemap_shops"]

        def parse_shop(self, response):
            pass  # ... scrape shop here ...

Combiner SitemapSpider avec d'autres sources d'URLs :

.. code-block:: python

    from scrapy import Request
    from scrapy.spiders import SitemapSpider


    class MySpider(SitemapSpider):
        sitemap_urls = ["http://www.example.com/robots.txt"]
        sitemap_rules = [
            ("/shop/", "parse_shop"),
        ]

        other_urls = ["http://www.example.com/about"]

        async def start(self):
            async for item_or_request in super().start():
                yield item_or_request
            for url in self.other_urls:
                yield Request(url, self.parse_other)

        def parse_shop(self, response):
            pass  # ... scrape shop here ...

        def parse_other(self, response):
            pass  # ... scrape other here ...

.. _scrapy-spider-metadata: https://scrapy-spider-metadata.readthedocs.io/en/latest/params.html
.. _Sitemaps: https://www.sitemaps.org/index.html
.. _fichiers d'index de Sitemap: https://www.sitemaps.org/protocol.html#index
.. _robots.txt: https://www.robotstxt.org/
.. _TLD: https://en.wikipedia.org/wiki/Top-level_domain
.. _documentation de Scrapyd: https://scrapyd.readthedocs.io/en/latest/
