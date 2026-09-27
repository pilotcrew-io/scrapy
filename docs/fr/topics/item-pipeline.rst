.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/item-pipeline.rst`.

.. _topics-item-pipeline:

=============
Item Pipeline
=============

Une fois qu'un item a été extrait par un spider, il est envoyé à l'Item
Pipeline, qui le traite à travers plusieurs composants exécutés les uns après
les autres.

Chaque composant d'item pipeline (parfois appelé simplement « Item Pipeline »)
est une classe Python qui implémente une méthode simple. Chaque composant reçoit
un item et effectue une action dessus. Il décide aussi si l'item doit continuer
son chemin dans le pipeline ou s'il doit être abandonné (« dropped ») et ne plus
être traité.

Voici des usages typiques des item pipelines :

* nettoyer des données HTML
* valider les données extraites (vérifier que les items contiennent certains
  champs)
* rechercher des doublons (et les abandonner)
* stocker l'item extrait dans une base de données


Écrire votre propre item pipeline
=================================

Chaque item pipeline est un :ref:`composant <topics-components>` qui doit
implémenter la méthode suivante :

.. method:: process_item(self, item)

   Scrapy appelle cette méthode pour chaque item traité par le composant de
   pipeline.

   `item` est un :ref:`objet item <item-types>`, voir
   :ref:`supporting-item-types`.

   :meth:`process_item` doit soit renvoyer un :ref:`objet item <item-types>`,
   soit lever une exception :exc:`~scrapy.exceptions.DropItem`.

   Les items abandonnés ne sont plus traités par les composants suivants du
   pipeline.

   :param item: l'item extrait
   :type item: :ref:`objet item <item-types>`

De plus, un composant peut implémenter les méthodes suivantes :

.. method:: open_spider(self)

   Cette méthode est appelée à l'ouverture du spider.

   .. versionchanged:: 2.18.0
      Ajout de la prise en charge de :exc:`~scrapy.exceptions.CloseSpider`.

   Elle peut lever :exc:`~scrapy.exceptions.CloseSpider` pour fermer le spider
   avant qu'il ne commence à explorer (« crawl »), par exemple si une ressource
   dont le pipeline a besoin n'est pas disponible.

.. method:: close_spider(self)

   Cette méthode est appelée à la fermeture du spider, avant l'envoi du signal
   :signal:`spider_closed`.

N'importe laquelle de ces méthodes peut être définie comme une fonction
coroutine (``async def``).

:meth:`open_spider` et :meth:`close_spider` s'exécutent en parallèle sur tous
les item pipelines activés ; seule :meth:`process_item` suit l'ordre défini par
:setting:`ITEM_PIPELINES`.


Exemples d'item pipelines
=========================

.. _price-pipeline-example:

Validation des prix et rejet des items sans prix
------------------------------------------------

Examinons le pipeline hypothétique suivant. Il ajuste l'attribut ``price`` des
items qui n'incluent pas la TVA (attribut ``price_excludes_vat``), et il
abandonne les items qui ne contiennent pas de prix :

.. code-block:: python

    from itemadapter import ItemAdapter
    from scrapy.exceptions import DropItem


    class PricePipeline:
        vat_factor = 1.15

        def process_item(self, item):
            adapter = ItemAdapter(item)
            if adapter.get("price"):
                if adapter.get("price_excludes_vat"):
                    adapter["price"] = adapter["price"] * self.vat_factor
                return item
            else:
                raise DropItem("Missing price")


Écrire les items dans un fichier JSON lines
-------------------------------------------

Le pipeline suivant stocke tous les items extraits (de tous les spiders) dans un
unique fichier ``items.jsonl``, qui contient un item par ligne, sérialisé au
format JSON :

.. code-block:: python

   import json

   from itemadapter import ItemAdapter


   class JsonWriterPipeline:
       def open_spider(self):
           self.file = open("items.jsonl", "w")

       def close_spider(self):
           self.file.close()

       def process_item(self, item):
           line = json.dumps(ItemAdapter(item).asdict()) + "\n"
           self.file.write(line)
           return item

.. note:: L'exemple JsonWriterPipeline sert simplement à montrer comment écrire
    des item pipelines. Si vous voulez vraiment stocker tous les items extraits
    dans un fichier JSON, vous devriez utiliser les :ref:`exports de feeds
    <topics-feed-exports>`.

Écrire les items dans MongoDB
-----------------------------

Dans cet exemple, nous écrivons les items dans MongoDB_ avec pymongo_. L'adresse
de MongoDB et le nom de la base de données sont indiqués dans les paramètres de
Scrapy ; la collection MongoDB est indiquée dans un attribut de classe.

Le but principal de cet exemple est de montrer comment :ref:`obtenir le crawler
<from-crawler>` et comment libérer correctement les ressources.

.. skip: next
.. code-block:: python

    import pymongo
    from itemadapter import ItemAdapter


    class MongoPipeline:
        collection_name = "scrapy_items"

        def __init__(self, mongo_uri, mongo_db):
            self.mongo_uri = mongo_uri
            self.mongo_db = mongo_db

        @classmethod
        def from_crawler(cls, crawler):
            return cls(
                mongo_uri=crawler.settings.get("MONGO_URI"),
                mongo_db=crawler.settings.get("MONGO_DATABASE", "items"),
            )

        def open_spider(self):
            self.client = pymongo.MongoClient(self.mongo_uri)
            self.db = self.client[self.mongo_db]

        def close_spider(self):
            self.client.close()

        def process_item(self, item):
            self.db[self.collection_name].insert_one(ItemAdapter(item).asdict())
            return item

.. _MongoDB: https://www.mongodb.com/
.. _pymongo: https://pymongo.readthedocs.io/en/stable/


.. _ScreenshotPipeline:

Prendre une capture d'écran d'un item
-------------------------------------

Cet exemple montre comment utiliser la :doc:`syntaxe des coroutines <coroutines>`
dans la méthode :meth:`process_item`.

Cet item pipeline envoie une requête à une instance de Splash_ qui tourne en
local, afin de générer une capture d'écran de l'URL de l'item. Une fois la
réponse à la requête téléchargée, l'item pipeline enregistre la capture d'écran
dans un fichier et ajoute le nom de ce fichier à l'item.

.. code-block:: python

    import hashlib
    from pathlib import Path
    from urllib.parse import quote

    import scrapy
    from itemadapter import ItemAdapter
    from scrapy.http.request import NO_CALLBACK


    class ScreenshotPipeline:
        """Pipeline that uses Splash to render a screenshot of every Scrapy
        item."""

        SPLASH_URL = "http://localhost:8050/render.png?url={}"

        def __init__(self, crawler):
            self.crawler = crawler

        @classmethod
        def from_crawler(cls, crawler):
            return cls(crawler)

        async def process_item(self, item):
            adapter = ItemAdapter(item)
            encoded_item_url = quote(adapter["url"])
            screenshot_url = self.SPLASH_URL.format(encoded_item_url)
            request = scrapy.Request(screenshot_url, callback=NO_CALLBACK)
            response = await self.crawler.engine.download_async(request)

            if response.status != 200:
                # An error occurred, so return the item.
                return item

            # Save the screenshot to a file; the filename is the hash of the
            # URL.
            url = adapter["url"]
            url_hash = hashlib.md5(url.encode("utf8")).hexdigest()
            filename = f"{url_hash}.png"
            Path(filename).write_bytes(response.body)

            # Store the filename in the item.
            adapter["screenshot_filename"] = filename
            return item

.. _Splash: https://splash.readthedocs.io/en/stable/

Filtre de doublons
------------------

Ce filtre recherche les items en double et abandonne ceux qui ont déjà été
traités. Supposons que nos items possèdent un identifiant unique, mais que notre
spider renvoie plusieurs items avec le même identifiant :

.. code-block:: python

    from itemadapter import ItemAdapter
    from scrapy.exceptions import DropItem


    class DuplicatesPipeline:
        def __init__(self):
            self.ids_seen = set()

        def process_item(self, item):
            adapter = ItemAdapter(item)
            if adapter["id"] in self.ids_seen:
                raise DropItem(f"Item ID already seen: {adapter['id']}")
            else:
                self.ids_seen.add(adapter["id"])
                return item


.. _activating-item-pipeline:

Activer un composant Item Pipeline
==================================

Pour activer un composant Item Pipeline, vous devez ajouter sa classe au
paramètre :setting:`ITEM_PIPELINES`, comme dans l'exemple suivant :

.. code-block:: python

   ITEM_PIPELINES = {
       "myproject.pipelines.PricePipeline": 300,
       "myproject.pipelines.JsonWriterPipeline": 800,
   }

Les valeurs entières que vous associez aux classes dans ce paramètre déterminent
l'ordre dans lequel elles s'exécutent : les items passent des classes ayant la
valeur la plus basse à celles ayant la valeur la plus haute. Il est d'usage de
choisir ces nombres dans la plage de 0 à 1000.

Un exemple complet
==================

Les exemples ci-dessus présentent des composants d'item pipeline isolés. Dans un
projet, un pipeline est l'une des quatre pièces qui travaillent ensemble :
l':ref:`item <topics-items>` que produit votre spider, le :ref:`spider
<topics-spiders>` qui le produit (avec ``yield``), le pipeline qui le traite, et
le paramètre :setting:`ITEM_PIPELINES` qui active le pipeline.

L'exemple suivant relie ces pièces pour valider le prix de livres extraits de
`books.toscrape.com`_, en réutilisant le ``PricePipeline`` de la section
:ref:`price-pipeline-example` ci-dessus.

Définissez l'item dans ``myproject/items.py`` :

.. code-block:: python

    from dataclasses import dataclass


    @dataclass
    class BookItem:
        title: str
        price: float

Produisez (avec ``yield``) des instances de cet item depuis votre spider, par
exemple dans ``myproject/spiders/books.py`` :

.. skip: next
.. code-block:: python

    import scrapy

    from myproject.items import BookItem


    class BooksSpider(scrapy.Spider):
        name = "books"
        start_urls = ["https://books.toscrape.com/"]

        def parse(self, response):
            for book in response.css("article.product_pod"):
                yield BookItem(
                    title=book.css("h3 a::attr(title)").get(),
                    price=float(book.css("p.price_color::text").re_first(r"[\d.]+")),
                )

Placez le ``PricePipeline`` présenté plus haut dans ``myproject/pipelines.py``,
et activez-le dans ``myproject/settings.py`` :

.. code-block:: python

    ITEM_PIPELINES = {
        "myproject.pipelines.PricePipeline": 300,
    }

Une fois ces pièces en place, chaque ``BookItem`` produit par ``BooksSpider``
passe par ``PricePipeline`` avant d'atteindre les :ref:`exports de feeds
<topics-feed-exports>` ou toute autre sortie.

.. _books.toscrape.com: https://books.toscrape.com/


.. _test-item-pipeline:

Tester un item pipeline
=======================

Pour envoyer les items d'une seule URL à travers vos item pipelines, utilisez la
commande :command:`parse` avec l'option ``--pipelines`` ::

    scrapy parse --pipelines "https://books.toscrape.com/"

Pour tester plutôt des données d'item précises, ajoutez un callback qui
construit un item à partir de ses arguments nommés :

.. skip: next
.. code-block:: python

    class BooksSpider(scrapy.Spider):
        # ...

        def parse_item(self, response, **fields):
            yield BookItem(**fields)

puis passez ces arguments nommés en ligne de commande ::

    scrapy parse --pipelines -c parse_item --cbkwargs '{"title": "Test", "price": 10}' "https://books.toscrape.com/"

Indiquez n'importe quelle URL que votre spider sait gérer ; elle est
téléchargée même si le callback l'ignore.


Pièges courants
===============

Le pipeline ne s'exécute pas
----------------------------

Un composant de pipeline ne s'exécute que si sa classe figure dans le paramètre
:setting:`ITEM_PIPELINES`, normalement dans le fichier :file:`settings.py` de
votre projet (voir :ref:`activating-item-pipeline`). L'ajouter au spider ou
ailleurs n'a aucun effet.

Pour vérifier que Scrapy a bien chargé votre pipeline, cherchez une ligne comme
celle-ci vers le début du journal du crawl ::

    [scrapy.middleware] INFO: Enabled item pipelines:
    ['myproject.pipelines.PricePipeline']

Si votre pipeline est absent de cette liste, vérifiez que son chemin d'import
correspond à l'entrée de :setting:`ITEM_PIPELINES`, et que le paramètre n'est
pas écrasé, par exemple par :attr:`~scrapy.Spider.custom_settings` ou par une
nouvelle définition de :setting:`ITEM_PIPELINES` dans :file:`settings.py`.

L'item n'est pas retourné
-------------------------

:meth:`process_item` doit renvoyer l'item (ou lever
:exc:`~scrapy.exceptions.DropItem`). Une erreur fréquente consiste à modifier
l'item mais à oublier de le renvoyer :

.. code-block:: python

    def process_item(self, item):
        ItemAdapter(item)["price"] *= 1.15
        # Bug: returns None, so the next component gets None instead of the item.

Renvoyez l'item afin que le composant suivant, et le reste de Scrapy, puissent
continuer à le traiter :

.. code-block:: python

    def process_item(self, item):
        ItemAdapter(item)["price"] *= 1.15
        return item
