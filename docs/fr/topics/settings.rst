.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/settings.rst`.

.. _topics-settings:

==========
Paramètres
==========

Les paramètres (settings) de Scrapy vous permettent de personnaliser le
comportement de tous les composants de Scrapy, y compris le cœur du framework,
les extensions, les pipelines et les spiders eux-mêmes.

L'infrastructure des paramètres fournit un espace de noms global de
correspondances clé-valeur, dans lequel le code peut aller chercher ses valeurs
de configuration. Les paramètres peuvent être renseignés par différents
mécanismes, décrits ci-dessous.

Les paramètres sont aussi le mécanisme qui permet de sélectionner le projet
Scrapy actuellement actif (au cas où vous en auriez plusieurs).

Pour la liste des paramètres intégrés disponibles, voir :
:ref:`topics-settings-ref`.

.. _topics-settings-module-envvar:

Désigner les paramètres
=======================

Lorsque vous utilisez Scrapy, vous devez lui indiquer quels paramètres vous
utilisez. Pour cela, servez-vous d'une variable d'environnement,
``SCRAPY_SETTINGS_MODULE``.

La valeur de ``SCRAPY_SETTINGS_MODULE`` doit être écrite avec la syntaxe des
chemins Python, par exemple ``myproject.settings``. Notez que le module de
paramètres doit se trouver dans le :ref:`chemin de recherche des imports
<tut-searchpath>` de Python.

.. _populating-settings:

Renseigner les paramètres
=========================

Les paramètres peuvent être renseignés par différents mécanismes, chacun ayant
un ordre de priorité différent :

 1. :ref:`Paramètres en ligne de commande <cli-settings>` (priorité la plus haute)
 2. :ref:`Paramètres du spider <spider-settings>`
 3. :ref:`Paramètres du projet <project-settings>`
 4. :ref:`Paramètres des add-ons <addon-settings>`
 5. :ref:`Paramètres par défaut propres à une commande <cmd-default-settings>`
 6. :ref:`Paramètres par défaut globaux <default-settings>` (priorité la plus basse)

.. _cli-settings:

1. Paramètres en ligne de commande
----------------------------------

Les paramètres définis en ligne de commande ont la priorité la plus haute : ils
écrasent tous les autres paramètres.

Vous pouvez remplacer explicitement un ou plusieurs paramètres avec l'option de
ligne de commande ``-s`` (ou ``--set``).

.. highlight:: sh

Exemple ::

    scrapy crawl myspider -s LOG_LEVEL=INFO -s LOG_FILE=scrapy.log

.. _spider-settings:

2. Paramètres du spider
-----------------------

Les :ref:`spiders <topics-spiders>` peuvent définir leurs propres paramètres,
qui auront priorité sur ceux du projet et les remplaceront.

.. note:: Les :ref:`paramètres pré-crawler <pre-crawler-settings>` ne peuvent
    pas être définis par spider, et les :ref:`paramètres du reactor
    <reactor-settings>` ainsi que les :ref:`paramètres de journalisation
    <logging-settings>` sont soumis à des restrictions lorsque l'on
    :ref:`exécute plusieurs spiders dans le même processus
    <run-multiple-spiders>`.

Une première manière de le faire consiste à renseigner leur attribut
:attr:`~scrapy.Spider.custom_settings` :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"

        custom_settings = {
            "SOME_SETTING": "some value",
        }

Il est souvent préférable d'implémenter plutôt
:meth:`~scrapy.Spider.update_settings`, et les paramètres définis à cet endroit
devraient utiliser explicitement la priorité ``"spider"`` :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"

        @classmethod
        def update_settings(cls, settings):
            super().update_settings(settings)
            settings.set("SOME_SETTING", "some value", priority="spider")

Il est aussi possible de modifier les paramètres dans la méthode
:meth:`~scrapy.Spider.from_crawler`, par exemple en fonction des
:ref:`arguments du spider <spiderargs>` ou d'une autre logique :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"

        @classmethod
        def from_crawler(cls, crawler, *args, **kwargs):
            spider = super().from_crawler(crawler, *args, **kwargs)
            if "some_argument" in kwargs:
                spider.settings.set(
                    "SOME_SETTING", kwargs["some_argument"], priority="spider"
                )
            return spider

.. _project-settings:

3. Paramètres du projet
-----------------------

Les projets Scrapy contiennent un module de paramètres, généralement un fichier
nommé ``settings.py``, dans lequel vous devriez renseigner la plupart des
paramètres qui s'appliquent à tous vos spiders.

:func:`scrapy.utils.project.get_project_settings` renvoie ces paramètres, par
exemple pour les transmettre à :class:`~scrapy.crawler.AsyncCrawlerProcess`
lorsque l'on :ref:`exécute Scrapy depuis un script <run-from-script>`.

.. seealso:: :ref:`topics-settings-module-envvar`

.. _addon-settings:

4. Paramètres des add-ons
-------------------------

Les :ref:`add-ons <topics-addons>` peuvent modifier les paramètres. Ils
devraient le faire avec la priorité ``"addon"`` chaque fois que c'est possible.

.. _cmd-default-settings:

5. Paramètres par défaut propres à une commande
-----------------------------------------------

Chaque :ref:`commande Scrapy <topics-commands>` peut avoir ses propres
paramètres par défaut, qui remplacent les :ref:`paramètres par défaut globaux
<default-settings>`.

Ces paramètres par défaut propres à une commande sont spécifiés dans l'attribut
``default_settings`` de chaque classe de commande.

.. _default-settings:

6. Paramètres par défaut globaux
--------------------------------

Le module ``scrapy.settings.default_settings`` définit les valeurs par défaut
globales de certains :ref:`paramètres intégrés <topics-settings-ref>`.

.. note:: :command:`startproject` génère un fichier ``settings.py`` qui fixe
    certains paramètres à des valeurs différentes.

    La documentation de référence des paramètres indique la valeur par défaut
    lorsqu'il en existe une. Si :command:`startproject` fixe une valeur, cette
    valeur est documentée comme valeur par défaut, et la valeur issue de
    ``scrapy.settings.default_settings`` est documentée comme valeur de
    repli (« fallback »).


Compatibilité avec pickle
=========================

Les valeurs des paramètres doivent être :ref:`sérialisables avec pickle
<pickle-picklable>`.

Chemins d'import et classes
===========================

Lorsqu'un paramètre référence un objet appelable (callable) que Scrapy doit
importer, comme une classe ou une fonction, il existe deux manières différentes
de spécifier cet objet :

-   Sous la forme d'une chaîne de caractères contenant le chemin d'import de cet
    objet

-   Sous la forme de l'objet lui-même

Par exemple :

.. skip: next
.. code-block:: python

   from mybot.pipelines.validate import ValidateMyItem

   ITEM_PIPELINES = {
       # passing the classname...
       ValidateMyItem: 300,
       # ...equals passing the class path
       "mybot.pipelines.validate.ValidateMyItem": 300,
   }

.. note:: Passer des objets non appelables n'est pas pris en charge.


Comment accéder aux paramètres
==============================

.. highlight:: python

Dans un spider, les paramètres sont disponibles via ``self.settings`` :

.. code-block:: python

    class MySpider(scrapy.Spider):
        name = "myspider"
        start_urls = ["http://example.com"]

        def parse(self, response):
            print(f"Existing settings: {self.settings.attributes.keys()}")

.. note::
    L'attribut ``settings`` est défini dans la classe Spider de base après
    l'initialisation du spider. Si vous voulez utiliser les paramètres avant
    l'initialisation (par exemple dans la méthode ``__init__()`` de votre
    spider), vous devrez redéfinir la méthode
    :meth:`~scrapy.Spider.from_crawler`.

Les :ref:`composants <topics-components>` peuvent eux aussi :ref:`accéder aux
paramètres <component-settings>`.

L'objet ``settings`` peut s'utiliser comme un :class:`dict` (par exemple
``settings["LOG_ENABLED"]``). Cependant, pour prendre en charge les valeurs de
paramètres qui ne sont pas des chaînes de caractères, et qui peuvent être
transmises depuis la ligne de commande sous forme de chaînes, il est
recommandé d'utiliser l'une des méthodes fournies par l'API
:class:`~scrapy.settings.Settings`.


.. _component-priority-dictionaries:

Dictionnaires de priorités des composants
=========================================

Un **dictionnaire de priorités de composants** est un :class:`dict` dont les
clés sont des :ref:`composants <topics-components>` et les valeurs sont des
priorités de composants. Par exemple :

.. skip: next
.. code-block:: python

    {
        "path.to.ComponentA": None,
        ComponentB: 100,
    }

Un composant peut être spécifié soit comme un objet classe, soit par son chemin
d'import.

Une clé qui ne peut pas être résolue en un composant, comme le chemin d'import
d'un composant qui n'existe plus, lève une exception, même si sa priorité est
:data:`None`.

.. versionchanged:: VERSION
   Les clés non résolubles étaient auparavant ignorées silencieusement dans
   certains cas.

.. warning:: Les dictionnaires de priorités de composants sont des objets
    :class:`dict` ordinaires. Veillez à ne pas définir plusieurs fois le même
    composant, par exemple avec des chaînes de chemin d'import différentes, ou
    en définissant à la fois un chemin d'import et un objet :class:`type`.

Une priorité peut être un :class:`int` ou :data:`None`.

Un composant de priorité 1 passe *avant* un composant de priorité 2. Ce que
signifie « passer avant » dépend toutefois du paramètre concerné. Par exemple,
dans le paramètre :setting:`DOWNLOADER_MIDDLEWARES`, les composants ont leur
méthode
:meth:`~scrapy.downloadermiddlewares.DownloaderMiddleware.process_request`
exécutée avant celle des composants suivants, mais ont leur méthode
:meth:`~scrapy.downloadermiddlewares.DownloaderMiddleware.process_response`
exécutée après celle des composants suivants.

Un composant de priorité :data:`None` est désactivé.

Certains dictionnaires de priorités de composants sont fusionnés avec une
valeur intégrée. Par exemple, :setting:`DOWNLOADER_MIDDLEWARES` est fusionné
avec :setting:`DOWNLOADER_MIDDLEWARES_BASE`. C'est là que :data:`None` devient
pratique : il vous permet de désactiver, dans le paramètre ordinaire, un
composant issu du paramètre de base :

.. code-block:: python

    DOWNLOADER_MIDDLEWARES = {
        "scrapy.downloadermiddlewares.offsite.OffsiteMiddleware": None,
    }


Paramètres spéciaux
===================

Les paramètres suivants fonctionnent de façon légèrement différente de tous les
autres.

.. _pre-crawler-settings:

Paramètres pré-crawler
----------------------

Les **paramètres pré-crawler** sont les paramètres utilisés avant la création
de l'objet :class:`~scrapy.crawler.Crawler`.

Ces paramètres ne peuvent pas être :ref:`définis depuis un spider
<spider-settings>`.

Ces paramètres sont :

-   :setting:`ADDONS`
-   :setting:`COMMANDS_MODULE`
-   :setting:`FORCE_CRAWLER_PROCESS`
-   :setting:`SPIDER_LOADER_CLASS` et les paramètres utilisés par la classe de
    chargement de spiders correspondante, par exemple :setting:`SPIDER_MODULES`
    et :setting:`SPIDER_LOADER_WARN_ONLY` pour la classe de chargement de
    spiders par défaut.
-   :setting:`TWISTED_REACTOR_ENABLED`

:setting:`ADDONS` est un cas particulier : il peut être défini depuis un
spider, mais la méthode ``update_pre_crawler_settings()`` des :ref:`add-ons
<topics-addons>` activés de cette façon n'est pas appelée.

:setting:`TWISTED_REACTOR` joue aussi le rôle de paramètre pré-crawler lorsque
l'on exécute une :ref:`commande qui a besoin d'un CrawlerProcess
<topics-commands-crawlerprocess>`, car sa valeur au niveau du projet détermine
la classe de processus de crawler utilisée.

.. _reactor-settings:

Paramètres du reactor
---------------------

Les **paramètres du reactor** sont les paramètres liés au :doc:`reactor Twisted
<twisted:core/howto/reactor-basics>`.

.. versionchanged:: VERSION
   :setting:`TWISTED_DNS_RESOLVER`, les paramètres du résolveur et
   :setting:`REACTOR_THREADPOOL_MAXSIZE` sont désormais lus depuis le premier
   spider, au lieu d'être lus depuis les paramètres du projet et ignorés dans
   les :ref:`paramètres par spider <spider-settings>`.

Comme un seul reactor peut être utilisé par processus, ces paramètres ne
peuvent pas avoir une valeur différente par spider lorsque l'on :ref:`exécute
plusieurs spiders dans le même processus <run-multiple-spiders>`.

Ces paramètres sont utilisés lors de l'installation du reactor :

-   :setting:`ASYNCIO_EVENT_LOOP` (impossible à définir par spider lorsque l'on
    utilise :class:`~scrapy.crawler.AsyncCrawlerProcess`, voir plus bas)

-   :setting:`TWISTED_REACTOR` (ignoré lorsque l'on utilise
    :class:`~scrapy.crawler.AsyncCrawlerProcess`, voir plus bas)

Ils peuvent être :ref:`définis depuis un spider <spider-settings>`, mais seules
les valeurs du premier spider exécuté sont utilisées, puisque c'est à ce
moment-là que le reactor est installé. Si un spider ultérieur demande un
reactor différent ou une boucle d'événements différente, une exception est
levée. Avec :class:`~scrapy.crawler.CrawlerRunner` et
:class:`~scrapy.crawler.AsyncCrawlerRunner`, le reactor doit être installé au
préalable ; ces paramètres servent donc uniquement à vérifier que le reactor et
la boucle d'événements installés correspondent bien à leurs valeurs.

Ces paramètres sont appliqués au démarrage du reactor :

-   :setting:`TWISTED_DNS_RESOLVER` et les paramètres utilisés par le composant
    correspondant, par exemple :setting:`DNSCACHE_ENABLED`,
    :setting:`DNSCACHE_SIZE` et :setting:`DNS_TIMEOUT` pour le composant par
    défaut.

-   :setting:`REACTOR_THREADPOOL_MAXSIZE`

Ils peuvent aussi être :ref:`définis depuis un spider <spider-settings>`, mais
seules les valeurs du premier spider exécuté sont utilisées ; si un spider
ultérieur définit une valeur différente, un avertissement est émis. Ils sont
totalement ignorés lorsque l'on utilise :class:`~scrapy.crawler.CrawlerRunner`
ou :class:`~scrapy.crawler.AsyncCrawlerRunner`, qui ne démarrent pas le
reactor.

Il existe une restriction supplémentaire pour :setting:`TWISTED_REACTOR` et
:setting:`ASYNCIO_EVENT_LOOP` lorsque l'on utilise
:class:`~scrapy.crawler.AsyncCrawlerProcess` : lorsque cette classe est
instanciée, elle installe
:class:`~twisted.internet.asyncioreactor.AsyncioSelectorReactor`, en ignorant
la valeur de :setting:`TWISTED_REACTOR` et en utilisant la valeur de
:setting:`ASYNCIO_EVENT_LOOP` qui a été transmise à
:meth:`AsyncCrawlerProcess.__init__()
<scrapy.crawler.AsyncCrawlerProcess.__init__>`. Si une valeur différente pour
:setting:`TWISTED_REACTOR` ou :setting:`ASYNCIO_EVENT_LOOP` est fournie plus
tard, par exemple dans les :ref:`paramètres par spider <spider-settings>`, une
exception sera levée.

Tous ces paramètres, à l'exception de :setting:`ASYNCIO_EVENT_LOOP`, ne sont
utilisés que lorsque le reactor Twisted est utilisé, c'est-à-dire lorsque
:setting:`TWISTED_REACTOR_ENABLED` vaut ``True``.

.. _logging-settings:

Paramètres de journalisation
----------------------------

Les **paramètres de journalisation** sont les paramètres qui configurent le
gestionnaire de journalisation racine global installé par
:func:`~scrapy.utils.log.configure_logging`.

Ces paramètres peuvent être définis depuis un spider. Cependant, comme un seul
gestionnaire de journalisation racine est actif par processus, ces paramètres
ne peuvent pas avoir une valeur différente par spider lorsque l'on
:ref:`exécute plusieurs spiders dans le même processus
<run-multiple-spiders>`.

Ces paramètres sont :

-   :setting:`LOG_COLOR`
-   :setting:`LOG_DATEFORMAT`
-   :setting:`LOG_ENABLED`
-   :setting:`LOG_ENCODING`
-   :setting:`LOG_FILE`
-   :setting:`LOG_FILE_APPEND`
-   :setting:`LOG_FORMAT`
-   :setting:`LOG_INSTALL_ROOT_HANDLER`
-   :setting:`LOG_LEVEL`
-   :setting:`LOG_SHORT_NAMES`
-   :setting:`LOG_STDOUT`

.. _topics-settings-ref:

Référence des paramètres intégrés
=================================

Voici la liste de tous les paramètres Scrapy disponibles, par ordre
alphabétique, avec leurs valeurs par défaut et le périmètre (scope) dans lequel
ils s'appliquent.

Le périmètre, lorsqu'il est indiqué, précise où le paramètre est utilisé, s'il
est lié à un composant particulier. Dans ce cas, le module de ce composant est
indiqué, typiquement une extension, un middleware ou un pipeline. Cela signifie
aussi que le composant doit être activé pour que le paramètre ait un effet.

.. setting:: ADDONS

ADDONS
------

Valeur par défaut : ``{}``

Un dict contenant les chemins des add-ons activés dans votre projet et leurs
priorités. Pour plus d'informations, voir :ref:`topics-addons`.

.. note:: Il s'agit d'un :ref:`paramètre pré-crawler <pre-crawler-settings>`,
    avec une réserve décrite dans cette section.

.. setting:: ASYNCIO_EVENT_LOOP

ASYNCIO_EVENT_LOOP
------------------

Valeur par défaut : ``None``

Chemin d'import d'une classe de boucle d'événements ``asyncio`` donnée.

Si le reactor asyncio est activé (voir :setting:`TWISTED_REACTOR`) ou lorsque
l'on :ref:`exécute Scrapy sans reactor <asyncio-without-reactor>`, ce paramètre
peut servir à spécifier la boucle d'événements asyncio à utiliser avec lui.
Donnez à ce paramètre le chemin d'import de la classe de boucle d'événements
asyncio souhaitée. Si le paramètre vaut ``None``, la boucle d'événements
asyncio par défaut sera utilisée.

Si vous installez vous-même le reactor asyncio avec la fonction
:func:`~scrapy.utils.reactor.install_reactor`, vous pouvez utiliser le
paramètre ``event_loop_path`` pour indiquer le chemin d'import de la classe de
boucle d'événements à utiliser.

Notez que la classe de boucle d'événements doit hériter de
:class:`asyncio.AbstractEventLoop`.

.. caution:: Sachez que, lorsque l'on utilise une boucle d'événements non
    définie par défaut (soit via :setting:`ASYNCIO_EVENT_LOOP`, soit installée
    avec :func:`~scrapy.utils.reactor.install_reactor`), Scrapy appellera
    :func:`asyncio.set_event_loop`, ce qui définira la boucle d'événements
    indiquée comme boucle courante pour le thread courant du système
    d'exploitation.

.. note:: Il s'agit d'un :ref:`paramètre du reactor <reactor-settings>`.

.. setting:: AWS_ACCESS_KEY_ID

AWS_ACCESS_KEY_ID
-----------------

Valeur par défaut : ``None``

La clé d'accès AWS utilisée par le code qui a besoin d'accéder à `Amazon Web
services`_, comme le :ref:`backend de stockage de feeds S3
<topics-feed-storage-s3>`.

.. setting:: AWS_ENDPOINT_URL

AWS_ENDPOINT_URL
----------------

Valeur par défaut : ``None``

URL du point d'accès (endpoint) utilisée pour un stockage de type S3, par
exemple Minio ou s3.scality.

.. setting:: AWS_MAX_POOL_CONNECTIONS

AWS_MAX_POOL_CONNECTIONS
------------------------

.. versionadded:: 2.18.0

Valeur par défaut : ``None``

Nombre maximal de connexions que les clients AWS, comme ceux du :ref:`backend
de stockage de feeds S3 <topics-feed-storage-s3>` et du :ref:`backend de
stockage S3 du pipeline de médias <media-pipelines-s3>`, conservent dans leur
pool de connexions.

Si la valeur est ``None``, celle de :setting:`REACTOR_THREADPOOL_MAXSIZE` est
utilisée.

Des valeurs inférieures au nombre d'appels AWS parallèles ne limitent pas ces
appels, mais leurs connexions sont alors fermées au lieu d'être réutilisées, ce
qui nuit aux performances, et des avertissements ``Connection pool is full,
discarding connection`` sont journalisés.

.. setting:: AWS_REGION_NAME

AWS_REGION_NAME
---------------

Valeur par défaut : ``None``

Le nom de la région associée au client AWS.

.. setting:: AWS_SECRET_ACCESS_KEY

AWS_SECRET_ACCESS_KEY
---------------------

Valeur par défaut : ``None``

La clé secrète AWS utilisée par le code qui a besoin d'accéder à `Amazon Web
services`_, comme le :ref:`backend de stockage de feeds S3
<topics-feed-storage-s3>`.

.. setting:: AWS_SESSION_TOKEN

AWS_SESSION_TOKEN
-----------------

Valeur par défaut : ``None``

Le jeton de sécurité AWS utilisé par le code qui a besoin d'accéder à `Amazon
Web services`_, comme le :ref:`backend de stockage de feeds S3
<topics-feed-storage-s3>`, lorsque l'on utilise des `identifiants de sécurité
temporaires`_.

.. _identifiants de sécurité temporaires: https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html

.. setting:: AWS_USE_SSL

AWS_USE_SSL
-----------

Valeur par défaut : ``None``

Utilisez cette option si vous voulez désactiver la connexion SSL pour la
communication avec S3 ou un stockage de type S3. Par défaut, SSL sera utilisé.

.. setting:: AWS_VERIFY

AWS_VERIFY
----------

Valeur par défaut : ``None``

Vérifie la connexion SSL entre Scrapy et S3 ou un stockage de type S3. Par
défaut, la vérification SSL aura lieu.

.. setting:: BOT_NAME

BOT_NAME
--------

Valeur par défaut : ``<project name>`` (:ref:`repli <default-settings>` :
``'scrapybot'``)

Le nom du bot implémenté par ce projet Scrapy (également appelé nom du projet).
Ce nom sera aussi utilisé pour la journalisation.

Il est automatiquement renseigné avec le nom de votre projet lorsque vous créez
votre projet avec la commande :command:`startproject`.

.. setting:: CONCURRENT_ITEMS

CONCURRENT_ITEMS
----------------

Valeur par défaut : ``100``

Nombre maximal d'items concurrents (par réponse) traités en parallèle dans les
:ref:`pipelines d'items <topics-item-pipeline>`.

.. setting:: CONCURRENT_REQUESTS

CONCURRENT_REQUESTS
-------------------

Valeur par défaut : ``16``

Le nombre maximal de requêtes concurrentes (c'est-à-dire simultanées) qui
seront effectuées par le downloader de Scrapy. Utilisez ``0`` pour ne fixer
aucune limite.

.. setting:: CONCURRENT_REQUESTS_PER_DOMAIN

CONCURRENT_REQUESTS_PER_DOMAIN
------------------------------

Valeur par défaut : ``1`` (:ref:`repli <default-settings>` : ``8``)

Le nombre maximal de requêtes concurrentes (c'est-à-dire simultanées) qui
seront effectuées vers un même domaine.

Voir aussi : :ref:`topics-autothrottle` et son option
:setting:`AUTOTHROTTLE_TARGET_CONCURRENCY`.

Il est possible de modifier ce paramètre par domaine en utilisant
:setting:`DOWNLOAD_SLOTS`.

.. setting:: DEFAULT_DROPITEM_LOG_LEVEL

DEFAULT_DROPITEM_LOG_LEVEL
--------------------------

Valeur par défaut : ``"WARNING"``

:ref:`Niveau de journalisation <levels>` par défaut des messages relatifs aux
items abandonnés.

Lorsqu'un item est abandonné en levant :exc:`scrapy.exceptions.DropItem` depuis
la méthode :func:`process_item` d'un :ref:`pipeline d'items
<topics-item-pipeline>`, un message est journalisé, et par défaut son niveau
de journalisation est celui configuré dans ce paramètre.

Vous pouvez indiquer ce niveau de journalisation sous la forme d'un entier (par
exemple ``20``), d'une constante de niveau de journalisation (par exemple
``logging.INFO``) ou d'une chaîne contenant le nom d'une constante de niveau de
journalisation (par exemple ``"INFO"``).

Lorsque vous écrivez un pipeline d'items, vous pouvez imposer un niveau de
journalisation différent en renseignant
:attr:`scrapy.exceptions.DropItem.log_level` dans votre exception
:exc:`scrapy.exceptions.DropItem`. Par exemple :

.. code-block:: python

   from scrapy.exceptions import DropItem


   class MyPipeline:
       def process_item(self, item):
           if not item.get("price"):
               raise DropItem("Missing price data", log_level="INFO")
           return item

.. setting:: DEFAULT_ITEM_CLASS

DEFAULT_ITEM_CLASS
------------------

Valeur par défaut : ``'scrapy.item.Item'``

La classe par défaut qui sera utilisée pour instancier les items dans le
:ref:`shell Scrapy <topics-shell>`.

.. setting:: DEFAULT_REQUEST_HEADERS

DEFAULT_REQUEST_HEADERS
-----------------------

Valeur par défaut :

.. code-block:: python

    {
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
        "Accept-Language": "en",
    }

Les en-têtes par défaut utilisés pour les requêtes HTTP de Scrapy. Ils sont
renseignés dans
:class:`~scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware`.

.. caution:: Les cookies définis via l'en-tête ``Cookie`` ne sont pas pris en
    compte par le :ref:`middleware de cookies <cookies>`. Si vous avez besoin
    de définir des cookies pour une requête, utilisez le paramètre
    :class:`Request.cookies <scrapy.Request>`.

.. caution:: Un en-tête ``Referer`` défini ici n'atteint que les requêtes pour
    lesquelles :class:`~scrapy.spidermiddlewares.referer.RefererMiddleware` n'en
    définit pas, comme les requêtes de départ (start requests). Pour l'envoyer
    sur chaque requête, donnez à :setting:`REFERRER_POLICY` la valeur
    ``"no-referrer"``.

.. setting:: DEPTH_LIMIT

DEPTH_LIMIT
-----------

Valeur par défaut : ``0``

Périmètre : ``scrapy.spidermiddlewares.depth.DepthMiddleware``

La profondeur maximale à laquelle le crawl est autorisé pour n'importe quel
site. Si la valeur est zéro, aucune limite n'est imposée.

.. setting:: DEPTH_PRIORITY

DEPTH_PRIORITY
--------------

Valeur par défaut : ``0``

Périmètre : ``scrapy.spidermiddlewares.depth.DepthMiddleware``

Un entier utilisé pour ajuster la :attr:`~scrapy.Request.priority` d'une
:class:`~scrapy.Request` en fonction de sa profondeur.

La priorité d'une requête est ajustée ainsi :

.. skip: next
.. code-block:: python

    request.priority = request.priority - (depth * DEPTH_PRIORITY)

Lorsque la profondeur augmente, des valeurs positives de ``DEPTH_PRIORITY``
diminuent la priorité de la requête (BFO, parcours en largeur d'abord), tandis
que des valeurs négatives l'augmentent (DFO, parcours en profondeur d'abord).
Voir aussi :ref:`faq-bfo-dfo`.

.. note::

    Ce paramètre ajuste la priorité **dans le sens opposé** à celui des autres
    paramètres de priorité, :setting:`REDIRECT_PRIORITY_ADJUST` et
    :setting:`RETRY_PRIORITY_ADJUST`.

.. setting:: DEPTH_STATS_VERBOSE

DEPTH_STATS_VERBOSE
-------------------

Valeur par défaut : ``False``

Périmètre : ``scrapy.spidermiddlewares.depth.DepthMiddleware``

Indique s'il faut collecter des statistiques de profondeur détaillées. Si ce
paramètre est activé, le nombre de requêtes pour chaque profondeur est collecté
dans les statistiques (stats).

.. setting:: DNSCACHE_ENABLED

DNSCACHE_ENABLED
----------------

Valeur par défaut : ``True``

Indique s'il faut activer le cache DNS en mémoire.

.. note::
    Ce paramètre n'est utilisé que par
    :class:`~scrapy.resolver.CachingThreadedResolver` et
    :class:`~scrapy.resolver.CachingHostnameResolver`. Il n'a aucun effet
    lorsque :setting:`TWISTED_REACTOR_ENABLED` vaut ``False``, et peut aussi
    n'avoir aucun effet lorsque :setting:`TWISTED_DNS_RESOLVER` est réglé sur
    un autre résolveur.

.. note:: Il s'agit d'un :ref:`paramètre du reactor <reactor-settings>`.

.. setting:: DNSCACHE_SIZE

DNSCACHE_SIZE
-------------

Valeur par défaut : ``10000``

Taille du cache DNS en mémoire, voir :setting:`DNSCACHE_ENABLED`.

.. note:: Il s'agit d'un :ref:`paramètre du reactor <reactor-settings>`.

.. setting:: DNS_TIMEOUT

DNS_TIMEOUT
-----------

Valeur par défaut : ``60``

Délai d'expiration (timeout) du traitement des requêtes DNS, en secondes. Les
nombres à virgule flottante sont acceptés.

Le délai commence à courir lorsque la requête est placée dans la file du pool
de threads du reactor Twisted, et non lorsqu'elle est envoyée. Si ce pool de
threads est saturé, des requêtes peuvent expirer avant d'avoir été envoyées ;
dans ce cas, augmenter :setting:`REACTOR_THREADPOOL_MAXSIZE` aide davantage
qu'augmenter ce paramètre.

.. note::
    Ce paramètre n'est utilisé que par
    :class:`~scrapy.resolver.CachingThreadedResolver`. Il n'a aucun effet
    lorsque :setting:`TWISTED_REACTOR_ENABLED` vaut ``False``, et peut aussi
    n'avoir aucun effet lorsque :setting:`TWISTED_DNS_RESOLVER` est réglé sur
    un autre résolveur.

.. note:: Il s'agit d'un :ref:`paramètre du reactor <reactor-settings>`.

.. setting:: DOWNLOADER

DOWNLOADER
----------

Valeur par défaut : ``'scrapy.core.downloader.Downloader'``

Le downloader à utiliser pour le crawl.

.. setting:: DOWNLOADER_CLIENT_TLS_CIPHERS

DOWNLOADER_CLIENT_TLS_CIPHERS
-----------------------------

Valeur par défaut : ``'DEFAULT'``

Utilisez ce paramètre pour personnaliser les chiffrements (ciphers) TLS/SSL
utilisés par le gestionnaire de téléchargement (download handler) HTTPS.

Le paramètre doit contenir une chaîne au `format de liste de chiffrements
OpenSSL`_ ; ces chiffrements seront utilisés comme chiffrements côté client.
Modifier ce paramètre peut être nécessaire pour accéder à certains sites web
HTTPS : par exemple, vous pouvez avoir besoin de ``'DEFAULT:!DH'`` pour un site
dont les paramètres DH sont faibles, ou d'activer un chiffrement précis qui
n'est pas inclus dans ``DEFAULT`` si un site l'exige.

Donnez à ce paramètre la valeur ``None`` pour utiliser les chiffrements par
défaut de l'implémentation TLS sous-jacente.

.. versionchanged:: 2.17.0
   Ajout de la possibilité de lui donner la valeur ``None``.

.. _format de liste de chiffrements OpenSSL: https://docs.openssl.org/master/man1/openssl-ciphers/#cipher-list-format

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers.

.. seealso:: :ref:`security-tls-protocols-ciphers`

.. setting:: DOWNLOAD_TLS_MAX_VERSION

DOWNLOAD_TLS_MAX_VERSION
------------------------

.. versionadded:: 2.17.0

Valeur par défaut : ``None``

Utilisez ce paramètre pour modifier la version maximale du protocole TLS que
Scrapy est autorisé à utiliser.

Ce paramètre doit valoir soit ``None``, auquel cas il n'influence pas le choix
de la version, soit l'une de ces valeurs de type chaîne :

- ``'TLSv1.0'``
- ``'TLSv1.1'``
- ``'TLSv1.2'``
- ``'TLSv1.3'``

La plage de versions TLS autorisées que Scrapy annonce lorsqu'il établit des
connexions TLS dépendra des valeurs par défaut de l'implémentation TLS et des
valeurs de :setting:`DOWNLOAD_TLS_MIN_VERSION` et de
:setting:`DOWNLOAD_TLS_MAX_VERSION`. Il est possible de réactiver des versions
prises en charge par l'implémentation TLS mais désactivées par défaut, en
ajustant ces paramètres, mais il est impossible d'activer des versions non
prises en charge, comme toutes les versions inférieures à 1.2 dans de nombreux
environnements modernes.

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers. De plus,
    l'ensemble des versions TLS prises en charge dépend de l'implémentation TLS
    utilisée par le handler.

.. seealso:: :ref:`security-tls-protocols-ciphers`

.. setting:: DOWNLOAD_TLS_MIN_VERSION

DOWNLOAD_TLS_MIN_VERSION
------------------------

.. versionadded:: 2.17.0

Valeur par défaut : ``None``

Utilisez ce paramètre pour modifier la version minimale du protocole TLS que
Scrapy est autorisé à utiliser.

Voir :setting:`DOWNLOAD_TLS_MAX_VERSION` pour les détails et les limites.

.. seealso:: :ref:`security-tls-protocols-ciphers`

.. setting:: DOWNLOADER_CLIENT_TLS_VERBOSE_LOGGING

DOWNLOADER_CLIENT_TLS_VERBOSE_LOGGING
-------------------------------------

Valeur par défaut : ``False``

Lui donner la valeur ``True`` active des messages de niveau DEBUG sur les
paramètres de la connexion TLS après l'établissement des connexions HTTPS. Le
type d'informations journalisées dépend de l'implémentation du download handler
et des versions des bibliothèques liées à TLS.

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers.

.. setting:: DOWNLOADER_MIDDLEWARES

DOWNLOADER_MIDDLEWARES
----------------------

Valeur par défaut : ``{}``

Un dict contenant les middlewares de downloader activés dans votre projet, et
leurs ordres. Pour plus d'informations, voir
:ref:`topics-downloader-middleware-setting`.

.. setting:: DOWNLOADER_MIDDLEWARES_BASE

DOWNLOADER_MIDDLEWARES_BASE
---------------------------

Valeur par défaut :

.. code-block:: python

    {
        "scrapy.downloadermiddlewares.offsite.OffsiteMiddleware": 50,
        "scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware": 100,
        "scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware": 300,
        "scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware": 350,
        "scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware": 400,
        "scrapy.downloadermiddlewares.useragent.UserAgentMiddleware": 500,
        "scrapy.downloadermiddlewares.retry.RetryMiddleware": 550,
        "scrapy.downloadermiddlewares.jsonvalidation.JsonValidationMiddleware": 560,
        "scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware": 580,
        "scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware": 590,
        "scrapy.downloadermiddlewares.redirect.RedirectMiddleware": 600,
        "scrapy.downloadermiddlewares.cookies.CookiesMiddleware": 700,
        "scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware": 750,
        "scrapy.downloadermiddlewares.stats.DownloaderStats": 850,
        "scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware": 900,
    }

Un dict contenant les middlewares de downloader activés par défaut dans Scrapy.
Les ordres bas sont plus proches de l'engine, les ordres élevés sont plus
proches du downloader. Vous ne devriez jamais modifier ce paramètre dans votre
projet ; modifiez plutôt :setting:`DOWNLOADER_MIDDLEWARES`. Pour plus
d'informations, voir :ref:`topics-downloader-middleware-setting`.

.. setting:: DOWNLOADER_MIDDLEWARE_RESPONSE_EXCEPTIONS

DOWNLOADER_MIDDLEWARE_RESPONSE_EXCEPTIONS
-----------------------------------------

.. versionadded:: VERSION

Valeur par défaut : ``False``

Indique si une exception levée par la méthode
:meth:`~scrapy.downloadermiddlewares.DownloaderMiddleware.process_response`
d'un middleware de downloader est transmise à la méthode
:meth:`~scrapy.downloadermiddlewares.DownloaderMiddleware.process_exception`
des middlewares de downloader qui n'ont pas encore traité la réponse.

L'activer permet à
:class:`~scrapy.downloadermiddlewares.retry.RetryMiddleware` de réessayer ces
exceptions, par exemple pour une réponse qui ne peut pas être décompressée.

Avant de l'activer, vérifiez que les méthodes ``process_exception`` de vos
middlewares de downloader traitent ces exceptions comme prévu. Elles reçoivent
aussi les exceptions :exc:`~scrapy.exceptions.IgnoreRequest` que des
middlewares lèvent pour abandonner une réponse ; une méthode qui renvoie une
requête pour chaque exception reçue transforme donc un tel abandon en nouvelle
requête.

``True`` deviendra la seule valeur prise en charge dans une future version de
Scrapy.

.. setting:: DOWNLOADER_STATS

DOWNLOADER_STATS
----------------

Valeur par défaut : ``True``

Indique s'il faut activer la collecte de statistiques du downloader.

.. setting:: DOWNLOAD_DELAY

DOWNLOAD_DELAY
--------------

Valeur par défaut : ``1`` (:ref:`repli <default-settings>` : ``0``)

Nombre minimal de secondes à attendre entre 2 requêtes consécutives vers un
même domaine.

Utilisez :setting:`DOWNLOAD_DELAY` pour limiter la vitesse de votre crawl, afin
de ne pas solliciter les serveurs trop fortement.

Les nombres décimaux sont acceptés. Par exemple, pour envoyer au maximum 4
requêtes toutes les 10 secondes :

.. code-block:: python

    DOWNLOAD_DELAY = 2.5

Ce paramètre est aussi affecté par le paramètre :setting:`DOWNLOAD_DELAY_JITTER`,
qui rend les délais aléatoires à ±50 % par défaut.

Notez que :setting:`DOWNLOAD_DELAY` peut faire descendre la concurrence
effective par domaine en dessous de :setting:`CONCURRENT_REQUESTS_PER_DOMAIN`.
Si le temps de réponse d'un domaine est inférieur à :setting:`DOWNLOAD_DELAY`,
la concurrence effective pour ce domaine est de 1. Lorsque vous testez des
configurations de limitation de débit, il est généralement judicieux de
commencer par diminuer :setting:`CONCURRENT_REQUESTS_PER_DOMAIN`, et de
n'augmenter :setting:`DOWNLOAD_DELAY` qu'une fois que
:setting:`CONCURRENT_REQUESTS_PER_DOMAIN` vaut 1 mais qu'une limitation plus
forte est souhaitée.

.. _spider-download_delay-attribute:

Il est possible de modifier ce paramètre par domaine en utilisant
:setting:`DOWNLOAD_SLOTS`.

.. setting:: DOWNLOAD_DELAY_JITTER

DOWNLOAD_DELAY_JITTER
---------------------

.. versionadded:: 2.19.0

Valeur par défaut : ``0.5``

Amplitude de la variation aléatoire appliquée à :setting:`DOWNLOAD_DELAY` ; par
exemple, ``0.2`` répartit les délais entre 80 % et 120 % de
:setting:`DOWNLOAD_DELAY`. ``0`` désactive l'aléatoire.

Rendre les délais aléatoires rend le temps entre les requêtes moins uniforme,
ce qui donne un schéma de crawl plus naturel.

Il est possible de modifier ce paramètre par domaine en utilisant
:setting:`DOWNLOAD_SLOTS`.

.. setting:: DOWNLOAD_BIND_ADDRESS

DOWNLOAD_BIND_ADDRESS
---------------------

Valeur par défaut : ``None``

L'adresse locale sortante par défaut pour les connexions des download handlers.

Ce paramètre peut être :

- une adresse d'hôte sous forme de chaîne (par exemple ``"127.0.0.2"``), auquel
  cas le port local est choisi automatiquement, ou

- un tuple ``(host, port)`` (par exemple ``("127.0.0.2", 50000)``) pour se lier
  à la fois à une interface locale précise et à un port local précis.

Par exemple :

.. code-block:: python

    # Bind to this local address
    DOWNLOAD_BIND_ADDRESS = "127.0.0.2"

.. code-block:: python

    # Bind to this local address and local port
    DOWNLOAD_BIND_ADDRESS = ("127.0.0.2", 5000)

S'il est défini, les download handlers HTTP intégrés utilisent cette valeur par
défaut. Définissez la clé de meta de requête :reqmeta:`bindaddress` pour la
remplacer pour une requête donnée.

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers. Spécifier le
    port n'est pas pris en charge par
    :class:`~scrapy.core.downloader.handlers._httpx.HttpxDownloadHandler`.

.. setting:: DOWNLOAD_HANDLERS

DOWNLOAD_HANDLERS
-----------------

Valeur par défaut : ``{}``

Un dict contenant les :ref:`download handlers <topics-download-handlers>`
activés dans votre projet.

Voir :setting:`DOWNLOAD_HANDLERS_BASE` pour un exemple de format.

.. seealso:: :ref:`security-unencrypted-protocols` et
    :ref:`security-local-resources`

.. setting:: DOWNLOAD_HANDLERS_BASE

DOWNLOAD_HANDLERS_BASE
----------------------

Valeur par défaut :

.. code-block:: python

    {
        "data": "scrapy.core.downloader.handlers.datauri.DataURIDownloadHandler",
        "file": "scrapy.core.downloader.handlers.file.FileDownloadHandler",
        "http": "scrapy.core.downloader.handlers.http11.HTTP11DownloadHandler",
        "https": "scrapy.core.downloader.handlers.http11.HTTP11DownloadHandler",
        "s3": "scrapy.core.downloader.handlers.s3.S3DownloadHandler",
        "ftp": "scrapy.core.downloader.handlers.ftp.FTPDownloadHandler",
    }

(lorsque :setting:`TWISTED_REACTOR_ENABLED` vaut ``True``)

.. code-block:: python

    {
        "data": "scrapy.core.downloader.handlers.datauri.DataURIDownloadHandler",
        "file": "scrapy.core.downloader.handlers.file.FileDownloadHandler",
        "http": "scrapy.core.downloader.handlers._aiohttp.AiohttpDownloadHandler",
        "https": "scrapy.core.downloader.handlers._aiohttp.AiohttpDownloadHandler",
        "s3": "scrapy.core.downloader.handlers.s3.S3DownloadHandler",
        "ftp": None,
    }

(lorsque :setting:`TWISTED_REACTOR_ENABLED` vaut ``False``)

Un dict contenant les :ref:`download handlers <topics-download-handlers>`
activés par défaut dans Scrapy. Vous ne devriez jamais modifier ce paramètre
dans votre projet ; modifiez plutôt :setting:`DOWNLOAD_HANDLERS`.

Vous pouvez désactiver n'importe lequel de ces download handlers en affectant
``None`` à leur schéma d'URI dans :setting:`DOWNLOAD_HANDLERS`. Par exemple,
pour désactiver le handler FTP intégré (sans remplacement), placez ceci dans
votre ``settings.py`` :

.. code-block:: python

    DOWNLOAD_HANDLERS = {
        "ftp": None,
    }

.. seealso:: :ref:`security-unencrypted-protocols` et
    :ref:`security-local-resources`


.. reqmeta:: download_slot
.. setting:: DOWNLOAD_SLOTS

DOWNLOAD_SLOTS
--------------

Valeur par défaut : ``{}``

Permet de définir des paramètres de concurrence et de délai par slot (domaine) :

    .. code-block:: python

        DOWNLOAD_SLOTS = {
            "quotes.toscrape.com": {"concurrency": 1, "delay": 2, "jitter": 0},
            "books.toscrape.com": {"delay": 3, "jitter": 0.2},
        }

.. note::

    Pour les autres slots du downloader, les valeurs par défaut des paramètres
    seront utilisées :

    -   :setting:`DOWNLOAD_DELAY` : ``delay``
    -   :setting:`CONCURRENT_REQUESTS_PER_DOMAIN` : ``concurrency``
    -   :setting:`DOWNLOAD_DELAY_JITTER` : ``jitter``

Les requêtes sont affectées à un slot en fonction du domaine de leur URL. Pour
affecter plutôt une requête à un slot précis, définissez le nom du slot comme
clé ``download_slot`` de :attr:`Request.meta <scrapy.Request.meta>`. Une fois
qu'une requête est affectée à un slot, cette clé contient le nom du slot.

Comme cette clé est conservée lors des redirections, une requête redirigée
reste dans le slot de la requête dont elle provient, même si elle pointe vers
un domaine différent.


.. setting:: DOWNLOAD_TIMEOUT

DOWNLOAD_TIMEOUT
----------------

Valeur par défaut : ``180``

Le temps (en secondes) que le downloader attendra avant d'abandonner sur
expiration du délai (timeout).

.. note::

    Ce délai peut être défini par requête grâce à la clé
    :reqmeta:`download_timeout` de :attr:`.Request.meta`.

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers.

.. setting:: DOWNLOAD_MAXSIZE
.. reqmeta:: download_maxsize

DOWNLOAD_MAXSIZE
----------------

Valeur par défaut : ``1073741824`` (1 Gio)

La taille maximale autorisée (en octets) du corps d'une réponse. Les réponses
plus volumineuses sont interrompues et ignorées.

Cela s'applique à la fois avant et après compression. Si la décompression du
corps d'une réponse devait dépasser cette limite, la décompression est
interrompue et la réponse est ignorée.

Utilisez ``0`` pour désactiver cette limite.

.. note::

    Cette limite peut être définie par requête grâce à la clé
    :reqmeta:`download_maxsize` de :attr:`.Request.meta`.

.. note::

    La vérification des réponses avant leur décompression doit être
    implémentée dans le :ref:`download handler <topics-download-handlers>` ;
    il n'est donc pas garanti qu'elle soit prise en charge par tous les
    handlers tiers.

.. setting:: DOWNLOAD_WARNSIZE
.. reqmeta:: download_warnsize

DOWNLOAD_WARNSIZE
-----------------

Valeur par défaut : ``33554432`` (32 Mio)

Si la taille d'une réponse dépasse cette valeur, avant ou après compression, un
avertissement sera journalisé.

Utilisez ``0`` pour désactiver cette limite.

.. note::

    Cette limite peut être définie par requête grâce à la clé
    :reqmeta:`download_warnsize` de :attr:`.Request.meta`.

.. note::

    La vérification des réponses avant leur décompression doit être
    implémentée dans le :ref:`download handler <topics-download-handlers>` ;
    il n'est donc pas garanti qu'elle soit prise en charge par tous les
    handlers tiers.

.. setting:: DOWNLOAD_FAIL_ON_DATALOSS

DOWNLOAD_FAIL_ON_DATALOSS
-------------------------

Valeur par défaut : ``True``

Indique s'il faut ou non échouer sur les réponses corrompues, c'est-à-dire
lorsque l'en-tête ``Content-Length`` déclaré ne correspond pas au contenu
envoyé par le serveur, ou qu'une réponse en morceaux (chunked) n'a pas été
terminée correctement. Si la valeur est ``True``, ces réponses lèvent une
exception :exc:`~scrapy.exceptions.ResponseDataLossError`. Si la valeur est
``False``, ces réponses sont transmises telles quelles et le drapeau
``dataloss`` est ajouté à la réponse, c'est-à-dire que
``'dataloss' in response.flags`` vaut ``True``.

Facultativement, cela peut être défini requête par requête en donnant la valeur
``False`` à la clé Request.meta :reqmeta:`download_fail_on_dataloss`.

.. note::

  Une réponse corrompue, ou erreur de perte de données, peut survenir dans
  plusieurs circonstances, d'une mauvaise configuration du serveur à des
  erreurs réseau, en passant par une corruption de données. C'est à
  l'utilisateur de décider s'il est pertinent de traiter des réponses
  corrompues, sachant qu'elles peuvent contenir un contenu partiel ou
  incomplet. Si :setting:`RETRY_ENABLED` vaut ``True`` et que ce paramètre vaut
  ``True``, l'échec :exc:`~scrapy.exceptions.ResponseDataLossError` sera
  réessayé comme d'habitude.

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers.

.. warning::

    Ce paramètre est ignoré par le
    :ref:`download handler <topics-download-handlers>`
    :class:`~scrapy.core.downloader.handlers.http2.H2DownloadHandler`. En cas
    d'erreur de perte de données, la connexion HTTP/2 correspondante peut être
    corrompue, ce qui affecte les autres requêtes qui utilisent la même
    connexion ; par conséquent, un échec ``ResponseFailed([InvalidBodyLengthError])``
    est toujours levé pour chaque requête qui utilisait cette connexion.

.. setting:: DOWNLOAD_VERIFY_CERTIFICATES

DOWNLOAD_VERIFY_CERTIFICATES
----------------------------

Valeur par défaut : ``False``

Indique si les download handlers HTTPS doivent vérifier le certificat TLS du
serveur lors d'une requête, et abandonner la requête si la vérification échoue.

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers. Le
    comportement exact d'un handler (par exemple, si les problèmes de
    certificat sont journalisés lorsque ce paramètre vaut ``False``) dépend de
    son implémentation.

.. seealso:: :ref:`security-certificate-verification`

.. setting:: DUPEFILTER_CLASS

DUPEFILTER_CLASS
----------------

Valeur par défaut : ``'scrapy.dupefilters.RFPDupeFilter'``

La classe utilisée pour détecter et filtrer les requêtes en double.

La classe par défaut, :class:`~scrapy.dupefilters.RFPDupeFilter`, filtre en
fonction du paramètre :setting:`REQUEST_FINGERPRINTER_CLASS`.

Pour changer la façon dont les doublons sont détectés, vous pouvez faire
pointer :setting:`DUPEFILTER_CLASS` vers une sous-classe personnalisée de
:class:`~scrapy.dupefilters.RFPDupeFilter` qui redéfinit sa méthode
``__init__`` pour utiliser une :ref:`autre classe d'empreinte de requête
<custom-request-fingerprinter>`. Par exemple :

.. code-block:: python

    from scrapy.dupefilters import RFPDupeFilter
    from scrapy.utils.request import fingerprint


    class CustomRequestFingerprinter:
        def fingerprint(self, request):
            return fingerprint(request, include_headers=["X-ID"])


    class CustomDupeFilter(RFPDupeFilter):

        def __init__(self, path=None, debug=False, *, fingerprinter=None):
            super().__init__(
                path=path, debug=debug, fingerprinter=CustomRequestFingerprinter()
            )

Pour désactiver le filtrage des requêtes en double, donnez à
:setting:`DUPEFILTER_CLASS` la valeur ``'scrapy.dupefilters.BaseDupeFilter'``.
Notez que ne pas filtrer les requêtes en double peut provoquer des boucles de
crawl. Il est généralement préférable de donner la valeur ``True`` au paramètre
``dont_filter`` de la méthode ``__init__`` d'un objet :class:`~scrapy.Request`
précis qui ne doit pas être filtré.

Une classe affectée à :setting:`DUPEFILTER_CLASS` doit implémenter l'interface
suivante :

.. code-block:: python

    class MyDupeFilter:

        @classmethod
        def from_crawler(cls, crawler):
            """Returns an instance of this duplicate request filtering class
            based on the current Crawler instance."""
            return cls()

        def request_seen(self, request):
            """Returns ``True`` if *request* is a duplicate of another request
            seen in a previous call to :meth:`request_seen`, or ``False``
            otherwise."""
            return False

        def open(self):
            """Called before the spider opens. It may return a deferred."""
            pass

        def close(self, reason):
            """Called before the spider closes. It may return a deferred."""
            pass

        def log(self, request, spider):
            """Logs that a request has been filtered out.

            It is called right after a call to :meth:`request_seen` that
            returns ``True``.

            If :meth:`request_seen` always returns ``False``, such as in the
            case of :class:`~scrapy.dupefilters.BaseDupeFilter`, this method
            may be omitted.
            """
            pass

.. autoclass:: scrapy.dupefilters.BaseDupeFilter

.. autoclass:: scrapy.dupefilters.RFPDupeFilter


.. setting:: DUPEFILTER_DEBUG

DUPEFILTER_DEBUG
----------------

Valeur par défaut : ``False``

Par défaut, ``RFPDupeFilter`` ne journalise que la première requête en double.
Donner à :setting:`DUPEFILTER_DEBUG` la valeur ``True`` lui fera journaliser
toutes les requêtes en double.

.. setting:: EDITOR

EDITOR
------

Valeur par défaut : ``vi`` (sur les systèmes Unix) ou l'éditeur IDLE (sur
Windows)

L'éditeur à utiliser pour modifier des spiders avec la commande
:command:`edit`. De plus, si la variable d'environnement ``EDITOR`` est
définie, la commande :command:`edit` la préférera au paramètre par défaut.

.. setting:: EXTENSIONS

EXTENSIONS
----------

Valeur par défaut : ``{}``

:ref:`Dictionnaire de priorités de composants
<component-priority-dictionaries>` des extensions activées. Voir
:ref:`topics-extensions`.

.. setting:: EXTENSIONS_BASE

EXTENSIONS_BASE
---------------

Valeur par défaut :

.. code-block:: python

    {
        "scrapy.extensions.corestats.CoreStats": 0,
        "scrapy.extensions.logcount.LogCount": 0,
        "scrapy.extensions.telnet.TelnetConsole": 0,
        "scrapy.extensions.memusage.MemoryUsage": 0,
        "scrapy.extensions.memdebug.MemoryDebugger": 0,
        "scrapy.extensions.closespider.CloseSpider": 0,
        "scrapy.extensions.feedexport.FeedExporter": 0,
        "scrapy.extensions.logstats.LogStats": 0,
        "scrapy.extensions.spiderstate.SpiderState": 0,
        "scrapy.extensions.throttle.AutoThrottle": 0,
        "scrapy.extensions.remote_control.RemoteControl": 0,
    }

Un dict contenant les extensions disponibles par défaut dans Scrapy, et leurs
ordres. Ce paramètre contient toutes les extensions intégrées stables. Gardez à
l'esprit que certaines d'entre elles doivent être activées par un paramètre.

Pour plus d'informations, voir le :ref:`guide utilisateur des extensions
<topics-extensions>` et la :ref:`liste des extensions disponibles
<topics-extensions-ref>`.

.. setting:: FEED_TEMPDIR

FEED_TEMPDIR
------------

Valeur par défaut : ``None``

Le répertoire temporaire des feeds (Feed Temp dir) vous permet de définir un
dossier personnalisé où enregistrer les fichiers temporaires du crawler avant
leur envoi avec le :ref:`stockage de feeds FTP <feed-storage-ftp>` et
:ref:`Amazon S3 <topics-feed-storage-s3>`.

.. setting:: FEED_STORAGE_GCS_ACL

FEED_STORAGE_GCS_ACL
--------------------

Valeur par défaut : ``""``

La liste de contrôle d'accès (ACL, Access Control List) utilisée lors du
stockage d'items sur :ref:`Google Cloud Storage <topics-feed-storage-gcs>`.
Pour plus d'informations sur la manière de renseigner cette valeur, reportez-vous
à la colonne *JSON API* dans la `documentation Google Cloud <https://docs.cloud.google.com/storage/docs/access-control/lists>`_.

.. setting:: FORCE_CRAWLER_PROCESS

FORCE_CRAWLER_PROCESS
---------------------

Valeur par défaut : ``False``

Si la valeur est ``False``, les :ref:`commandes Scrapy qui ont besoin d'un
CrawlerProcess <topics-commands-crawlerprocess>`, lorsque
:setting:`TWISTED_REACTOR_ENABLED` vaut ``True``, choisiront entre
:class:`scrapy.crawler.AsyncCrawlerProcess` et
:class:`scrapy.crawler.CrawlerProcess` en fonction de la valeur du paramètre
:setting:`TWISTED_REACTOR`, mais en ignorant sa valeur dans les :ref:`paramètres
par spider <spider-settings>`.

Si la valeur est ``True``, ces commandes utiliseront toujours
:class:`~scrapy.crawler.CrawlerProcess` lorsque
:setting:`TWISTED_REACTOR_ENABLED` vaut ``True``.

Lorsque :setting:`TWISTED_REACTOR_ENABLED` vaut ``False``,
:class:`~scrapy.crawler.AsyncCrawlerProcess` sera utilisé dans tous les cas.

Donnez-lui la valeur ``True`` si vous voulez donner à :setting:`TWISTED_REACTOR`
une valeur non définie par défaut dans les :ref:`paramètres par spider
<spider-settings>`.

.. note:: Il s'agit d'un :ref:`paramètre pré-crawler <pre-crawler-settings>`.

.. reqmeta:: ftp_passive
.. setting:: FTP_PASSIVE_MODE

FTP_PASSIVE_MODE
----------------

Valeur par défaut : ``True``

Indique s'il faut ou non utiliser le mode passif lors de l'initiation des
transferts FTP, sauf s'il existe une clé ``"ftp_passive"`` dans la meta de
``Request``.

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers.

.. reqmeta:: ftp_password
.. setting:: FTP_PASSWORD

FTP_PASSWORD
------------

Valeur par défaut : ``"guest"``

Le mot de passe à utiliser pour les connexions FTP lorsqu'il n'y a pas de clé
``"ftp_password"`` dans la meta de ``Request``.

.. note::
    Pour paraphraser la `RFC 1635`_, bien qu'il soit courant d'utiliser soit le
    mot de passe « guest », soit son adresse e-mail pour un FTP anonyme,
    certains serveurs FTP demandent explicitement l'adresse e-mail de
    l'utilisateur et n'autorisent pas la connexion avec le mot de passe
    « guest ».

.. _RFC 1635: https://datatracker.ietf.org/doc/html/rfc1635

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers.

.. reqmeta:: ftp_user
.. setting:: FTP_USER

FTP_USER
--------

Valeur par défaut : ``"anonymous"``

Le nom d'utilisateur à utiliser pour les connexions FTP lorsqu'il n'y a pas de
clé ``"ftp_user"`` dans la meta de ``Request``.

.. note::

    La prise en charge de ce paramètre doit être implémentée dans le
    :ref:`download handler <topics-download-handlers>` ; il n'est donc pas
    garanti qu'il soit pris en charge par tous les handlers tiers.

.. setting:: GCS_PROJECT_ID

GCS_PROJECT_ID
--------------

Valeur par défaut : ``None``

L'ID de projet qui sera utilisé lors du stockage de données sur `Google Cloud
Storage`_.

.. setting:: HTTP2_MAX_FRAME_SIZE

HTTP2_MAX_FRAME_SIZE
--------------------

.. versionadded:: 2.18.0

Valeur par défaut : ``16384``

`Taille de trame`_ maximale, en octets, que les serveurs peuvent envoyer, entre
``16384`` et ``16777215``. Les connexions vers des serveurs qui envoient une
trame plus grande échouent.

Augmentez-la pour les serveurs qui envoient des trames plus grandes quelle que
soit cette valeur. Notez que :setting:`DOWNLOAD_MAXSIZE` et
:setting:`DOWNLOAD_WARNSIZE` sont vérifiés une fois par trame reçue ; une valeur
plus élevée permet donc à une réponse de les dépasser davantage avant d'être
interceptée.

:class:`~scrapy.core.downloader.handlers._httpx.HttpxDownloadHandler` ignore ce
paramètre, car ``httpx`` ne permet pas de configurer la taille des trames.

.. _Taille de trame: https://datatracker.ietf.org/doc/html/rfc7540#section-4.2

.. setting:: ITEM_PIPELINES

ITEM_PIPELINES
--------------

Valeur par défaut : ``{}``

Un dict contenant les pipelines d'items à utiliser, et leurs ordres. Les
valeurs d'ordre sont arbitraires, mais il est d'usage de les définir dans la
plage 0-1000. Les ordres les plus bas sont traités avant les ordres les plus
hauts.

Exemple :

.. code-block:: python

   ITEM_PIPELINES = {
       "mybot.pipelines.validate.ValidateMyItem": 300,
       "mybot.pipelines.validate.StoreMyItem": 800,
   }

.. setting:: ITEM_PIPELINES_BASE

ITEM_PIPELINES_BASE
-------------------

Valeur par défaut : ``{}``

Un dict contenant les pipelines activés par défaut dans Scrapy. Vous ne devriez
jamais modifier ce paramètre dans votre projet ; modifiez plutôt
:setting:`ITEM_PIPELINES`.

.. setting:: ITEM_PROCESSOR

ITEM_PROCESSOR
--------------

Valeur par défaut : ``"scrapy.pipelines.ItemPipelineManager"``

Le :ref:`composant <topics-components>` qui construit le :ref:`pipeline d'items
<topics-item-pipeline>` à partir de :setting:`ITEM_PIPELINES` et y fait passer
les items extraits. Il doit implémenter
:class:`~scrapy.pipelines.ItemProcessorProtocol`.

.. autoclass:: scrapy.pipelines.ItemProcessorProtocol
    :members:


.. setting:: JOBDIR

JOBDIR
------

Valeur par défaut : ``None``

Une chaîne indiquant le répertoire où stocker l'état d'un crawl lorsque l'on
:ref:`met en pause et reprend des crawls <topics-jobs>`.


.. setting:: LOG_COLOR

LOG_COLOR
---------

.. versionadded:: 2.19.0

Valeur par défaut : ``True``

Indique s'il faut colorer la sortie des journaux selon le niveau de
journalisation lorsque l'on journalise vers un terminal. Nécessite l'extra
``color`` :

.. code-block:: bash

    pip install scrapy[color]

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_ENABLED

LOG_ENABLED
-----------

Valeur par défaut : ``True``

Indique s'il faut activer la journalisation.

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_ENCODING

LOG_ENCODING
------------

Valeur par défaut : ``'utf-8'``

L'encodage à utiliser pour la journalisation.

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_FILE

LOG_FILE
--------

Valeur par défaut : ``None``

Nom de fichier à utiliser pour la sortie de la journalisation. Si la valeur est
``None``, l'erreur standard sera utilisée.

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_FILE_APPEND

LOG_FILE_APPEND
---------------

Valeur par défaut : ``True``

Si la valeur est ``False``, le fichier de journal indiqué par
:setting:`LOG_FILE` sera écrasé (ce qui supprime la sortie des exécutions
précédentes, le cas échéant).

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_FORMAT

LOG_FORMAT
----------

Valeur par défaut : ``'%(asctime)s [%(name)s] %(levelname)s: %(message)s'``

Chaîne de formatage des messages de journal. Reportez-vous à la
:ref:`documentation de la journalisation Python <logrecord-attributes>` pour la
liste complète des espaces réservés (placeholders) disponibles.

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_DATEFORMAT

LOG_DATEFORMAT
--------------

Valeur par défaut : ``'%Y-%m-%d %H:%M:%S'``

Chaîne de formatage de la date et de l'heure, qui remplace l'espace réservé
``%(asctime)s`` dans :setting:`LOG_FORMAT`. Reportez-vous à la
:ref:`documentation datetime de Python <strftime-strptime-behavior>` pour la
liste complète des directives disponibles.

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_FORMATTER

LOG_FORMATTER
-------------

Valeur par défaut : :class:`scrapy.logformatter.LogFormatter`

La classe à utiliser pour :ref:`formater les messages de journal
<custom-log-formats>` des différentes actions.

.. setting:: LOG_INSTALL_ROOT_HANDLER

LOG_INSTALL_ROOT_HANDLER
------------------------

.. versionadded:: 2.19.0

Valeur par défaut : ``True``

Indique s'il faut installer un gestionnaire pour le logger racine, configuré
selon les autres :ref:`paramètres de journalisation <logging-settings>`.
Donnez-lui la valeur ``False`` pour gérer vous-même le logger racine, par
exemple depuis une commande personnalisée ou depuis un module
:file:`settings.py` exécuté avant que Scrapy ne configure la journalisation.

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_LEVEL

LOG_LEVEL
---------

Valeur par défaut : ``'DEBUG'``

Niveau minimal à journaliser. Les niveaux disponibles sont : CRITICAL, ERROR,
WARNING, INFO, DEBUG. Pour plus d'informations, voir :ref:`topics-logging`.

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_STDOUT

LOG_STDOUT
----------

Valeur par défaut : ``False``

Si la valeur est ``True``, toute la sortie standard (et d'erreur) de votre
processus sera redirigée vers le journal. Par exemple, si vous faites
``print('hello')``, cela apparaîtra dans le journal de Scrapy.

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_SHORT_NAMES

LOG_SHORT_NAMES
---------------

Valeur par défaut : ``False``

Si la valeur est ``True``, les journaux ne contiendront que le chemin racine.
Si elle est ``False``, ils affichent le composant responsable de la sortie du
journal.

.. note:: Il s'agit d'un :ref:`paramètre de journalisation <logging-settings>`.

.. setting:: LOG_VERSIONS

LOG_VERSIONS
------------

Valeur par défaut : ``["lxml", "libxml2", "cssselect", "parsel", "w3lib", "Twisted", "Python", "pyOpenSSL", "cryptography", "Platform"]``

Journalise les versions installées des éléments indiqués.

Un élément peut être n'importe quel paquet Python installé.

Les éléments spéciaux suivants sont également pris en charge :

-   ``libxml2``

-   ``Platform`` (:func:`platform.platform`)

-   ``Python``

-   ``pyOpenSSL``

.. setting:: LOGSTATS_INTERVAL

LOGSTATS_INTERVAL
-----------------

Valeur par défaut : ``60.0``

L'intervalle (en secondes) entre chaque affichage des statistiques dans le
journal par :class:`~scrapy.extensions.logstats.LogStats`.

.. setting:: MEMDEBUG_ENABLED

MEMDEBUG_ENABLED
----------------

Valeur par défaut : ``False``

Indique s'il faut activer le débogage de la mémoire.

.. setting:: MEMUSAGE_ENABLED

MEMUSAGE_ENABLED
----------------

Valeur par défaut : ``True``

Périmètre : ``scrapy.extensions.memusage.MemoryUsage``

Indique s'il faut activer l'extension d'utilisation de la mémoire. Cette
extension suit le pic de mémoire utilisé par le processus (elle l'écrit dans
les statistiques). Elle peut aussi, en option, arrêter le processus Scrapy
lorsqu'il dépasse une limite de mémoire (voir :setting:`MEMUSAGE_LIMIT_MB`).

Voir :ref:`topics-extensions-ref-memusage`.

.. setting:: MEMUSAGE_LIMIT_MB

MEMUSAGE_LIMIT_MB
-----------------

Valeur par défaut : ``0``

Périmètre : ``scrapy.extensions.memusage.MemoryUsage``

La quantité maximale de mémoire autorisée (en mégaoctets) avant d'arrêter
Scrapy (si :setting:`MEMUSAGE_ENABLED` vaut ``True``). Si la valeur est zéro,
aucune vérification ne sera effectuée.

Voir :ref:`topics-extensions-ref-memusage`.

.. setting:: MEMUSAGE_CHECK_INTERVAL_SECONDS

MEMUSAGE_CHECK_INTERVAL_SECONDS
-------------------------------

Valeur par défaut : ``60.0``

Périmètre : ``scrapy.extensions.memusage.MemoryUsage``

L':ref:`extension d'utilisation de la mémoire <topics-extensions-ref-memusage>`
vérifie l'utilisation actuelle de la mémoire, par rapport aux limites fixées
par :setting:`MEMUSAGE_LIMIT_MB` et :setting:`MEMUSAGE_WARNING_MB`, à
intervalles de temps fixes.

Ce paramètre fixe la durée de ces intervalles, en secondes.

Voir :ref:`topics-extensions-ref-memusage`.

.. setting:: MEMUSAGE_WARNING_MB

MEMUSAGE_WARNING_MB
-------------------

Valeur par défaut : ``0``

Périmètre : ``scrapy.extensions.memusage.MemoryUsage``

La quantité maximale de mémoire autorisée (en mégaoctets) avant d'envoyer un
signal :signal:`memusage_warning_reached` (si :setting:`MEMUSAGE_ENABLED` vaut
``True``). Si la valeur est zéro, aucun signal ne sera envoyé.

Voir :ref:`topics-extensions-ref-memusage`.

.. setting:: NEWSPIDER_MODULE

NEWSPIDER_MODULE
----------------

Valeur par défaut : ``"<project name>.spiders"`` (:ref:`repli <default-settings>` : ``""``)

Module dans lequel créer les nouveaux spiders avec la commande
:command:`genspider`.

Exemple :

.. code-block:: python

    NEWSPIDER_MODULE = "mybot.spiders_dev"

.. setting:: REACTOR_THREADPOOL_MAXSIZE

REACTOR_THREADPOOL_MAXSIZE
--------------------------

Valeur par défaut : ``10``

La limite maximale de la taille du pool de threads du reactor Twisted. C'est un
pool de threads polyvalent commun, utilisé par divers composants de Scrapy : le
résolveur DNS à threads, BlockingFeedStorage, S3FilesStore, pour n'en citer que
quelques-uns. Augmentez cette valeur si vous rencontrez des problèmes de
manque de capacité pour les entrées/sorties bloquantes.

.. note:: Il s'agit d'un :ref:`paramètre du reactor <reactor-settings>`.

.. setting:: REDIRECT_PRIORITY_ADJUST

REDIRECT_PRIORITY_ADJUST
------------------------

Valeur par défaut : ``+2``

Périmètre : ``scrapy.downloadermiddlewares.redirect.RedirectMiddleware``

Ajuste la priorité d'une requête de redirection par rapport à la requête
d'origine :

- **un ajustement de priorité positif (valeur par défaut) signifie une priorité
  plus élevée.**
- un ajustement de priorité négatif signifie une priorité plus basse.

.. setting:: ROBOTSTXT_OBEY

ROBOTSTXT_OBEY
--------------

Valeur par défaut : ``True`` (:ref:`repli <default-settings>` : ``False``)

Si ce paramètre est activé, Scrapy respectera les règles de robots.txt. Pour
plus d'informations, voir :ref:`topics-dlmw-robots`.

.. note::

    Bien que la valeur par défaut soit ``False`` pour des raisons historiques,
    cette option est activée par défaut dans le fichier settings.py généré par
    la commande ``scrapy startproject``.

.. setting:: ROBOTSTXT_PARSER

ROBOTSTXT_PARSER
----------------

Valeur par défaut : ``'scrapy.robotstxt.ProtegoRobotParser'``

Le backend d'analyse (parser) à utiliser pour analyser les fichiers
``robots.txt``. Pour plus d'informations, voir :ref:`topics-dlmw-robots`.

.. setting:: ROBOTSTXT_USER_AGENT

ROBOTSTXT_USER_AGENT
--------------------

Valeur par défaut : ``None``

La chaîne user agent à utiliser pour la correspondance dans le fichier
robots.txt. Si la valeur est ``None``, l'en-tête User-Agent que vous envoyez
avec la requête, ou le paramètre :setting:`USER_AGENT` (dans cet ordre), sera
utilisé pour déterminer le user agent à utiliser dans le fichier robots.txt.

.. setting:: SCHEDULER

SCHEDULER
---------

Valeur par défaut : :class:`~scrapy.core.scheduler.Scheduler`

La classe de scheduler à utiliser pour le crawl. Voir :ref:`topics-scheduler`
pour les détails.

.. setting:: SCHEDULER_DEBUG

SCHEDULER_DEBUG
---------------

Valeur par défaut : ``False``

Lui donner la valeur ``True`` journalisera des informations de débogage sur le
scheduler de requêtes. Actuellement, cela journalise (une seule fois) le cas où
les requêtes ne peuvent pas être sérialisées sur le disque. La statistique
:stat:`scheduler/unserializable` compte le nombre de fois où cela se produit.

Exemple d'entrée dans les journaux ::

    1956-01-31 00:00:00+0800 [scrapy.core.scheduler] ERROR: Unable to serialize request:
    <GET http://example.com> - reason: cannot serialize <Request at 0x9a7c7ec>
    (type Request)> - no more unserializable requests will be logged
    (see 'scheduler/unserializable' stats counter)


.. setting:: SCHEDULER_DISK_QUEUE

SCHEDULER_DISK_QUEUE
--------------------

Valeur par défaut : ``'scrapy.squeues.PickleLifoDiskQueue'``

.. versionadded:: 2.19.0
   Les types de files ``SQLite``.

Type de file sur disque qui sera utilisée par le scheduler. Les autres types
disponibles sont ``scrapy.squeues.PickleFifoDiskQueue``,
``scrapy.squeues.MarshalFifoDiskQueue``,
``scrapy.squeues.MarshalLifoDiskQueue``,
``scrapy.squeues.PickleFifoSQLiteQueue``,
``scrapy.squeues.PickleLifoSQLiteQueue``,
``scrapy.squeues.MarshalFifoSQLiteQueue`` et
``scrapy.squeues.MarshalLifoSQLiteQueue``.

Les types ``SQLite`` stockent les requêtes dans une base de données SQLite, ce
qui rend les écritures plus lentes mais garde la file utilisable après un arrêt
brutal. Voir :ref:`security-job-state`.


.. setting:: SCHEDULER_MEMORY_QUEUE

SCHEDULER_MEMORY_QUEUE
----------------------

Valeur par défaut : ``'scrapy.squeues.LifoMemoryQueue'``

Type de file en mémoire utilisée par le scheduler. L'autre type disponible est :
``scrapy.squeues.FifoMemoryQueue``.


.. setting:: SCHEDULER_PRIORITY_QUEUE
.. _broad-crawls-scheduler-priority-queue:

SCHEDULER_PRIORITY_QUEUE
------------------------

Valeur par défaut : :class:`~scrapy.pqueues.DownloaderAwarePriorityQueue`

Type de file de priorité utilisée par le scheduler.

Un autre type disponible est :class:`~scrapy.pqueues.ScrapyPriorityQueue`.

:class:`~scrapy.pqueues.DownloaderAwarePriorityQueue` fonctionne mieux que
:class:`~scrapy.pqueues.ScrapyPriorityQueue` lorsque vous crawlez en parallèle
de nombreux domaines différents.


.. setting:: SCHEDULER_START_DISK_QUEUE

SCHEDULER_START_DISK_QUEUE
--------------------------

Valeur par défaut : ``'scrapy.squeues.PickleFifoDiskQueue'``

Type de file sur disque (voir :setting:`JOBDIR`) que le :ref:`scheduler
<topics-scheduler>` utilise pour les :ref:`requêtes de départ (start requests)
<start-requests>`.

Pour les choix disponibles, voir :setting:`SCHEDULER_DISK_QUEUE`.

.. queue-common-starts

Utilisez ``None`` ou ``""`` pour désactiver entièrement ces files séparées, et
faire plutôt partager aux requêtes de départ les mêmes files que les autres
requêtes.

.. note::

    Désactiver les files séparées des requêtes de départ rend l':ref:`ordre des
    requêtes de départ <start-request-order>` peu intuitif : les requêtes de
    départ seront envoyées dans l'ordre uniquement jusqu'à ce que
    :setting:`CONCURRENT_REQUESTS` soit atteint, puis les requêtes de départ
    restantes seront envoyées dans l'ordre inverse.

.. queue-common-ends


.. setting:: SCHEDULER_START_MEMORY_QUEUE

SCHEDULER_START_MEMORY_QUEUE
----------------------------

Valeur par défaut : ``'scrapy.squeues.FifoMemoryQueue'``

Type de file en mémoire que le :ref:`scheduler <topics-scheduler>` utilise pour
les :ref:`requêtes de départ (start requests) <start-requests>`.

Pour les choix disponibles, voir :setting:`SCHEDULER_MEMORY_QUEUE`.

.. include:: settings.rst
    :start-after: queue-common-starts
    :end-before: queue-common-ends


.. setting:: SCRAPER_SLOT_MAX_ACTIVE_SIZE

SCRAPER_SLOT_MAX_ACTIVE_SIZE
----------------------------

Valeur par défaut : ``5_000_000``

Limite souple (en octets) pour les données de réponse en cours de traitement.

Tant que la somme des tailles de toutes les réponses en cours de traitement est
supérieure à cette valeur, Scrapy ne traite pas de nouvelles requêtes.

.. setting:: SPIDER_CONTRACTS

SPIDER_CONTRACTS
----------------

Valeur par défaut : ``{}``

Un dict contenant les contrats de spider (spider contracts) activés dans votre
projet, utilisés pour tester les spiders. Pour plus d'informations, voir
:ref:`topics-contracts`.

.. setting:: SPIDER_CONTRACTS_BASE

SPIDER_CONTRACTS_BASE
---------------------

Valeur par défaut :

.. code-block:: python

    {
        "scrapy.contracts.default.UrlContract": 1,
        "scrapy.contracts.default.CallbackKeywordArgumentsContract": 1,
        "scrapy.contracts.default.MetadataContract": 1,
        "scrapy.contracts.default.ReturnsContract": 2,
        "scrapy.contracts.default.ScrapesContract": 3,
    }

Un dict contenant les contrats Scrapy activés par défaut dans Scrapy. Vous ne
devriez jamais modifier ce paramètre dans votre projet ; modifiez plutôt
:setting:`SPIDER_CONTRACTS`. Pour plus d'informations, voir
:ref:`topics-contracts`.

Vous pouvez désactiver n'importe lequel de ces contrats en affectant ``None`` à
leur chemin de classe dans :setting:`SPIDER_CONTRACTS`. Par exemple, pour
désactiver le ``ScrapesContract`` intégré, placez ceci dans votre
``settings.py`` :

.. code-block:: python

    SPIDER_CONTRACTS = {
        "scrapy.contracts.default.ScrapesContract": None,
    }

.. setting:: SPIDER_LOADER_CLASS

SPIDER_LOADER_CLASS
-------------------

Valeur par défaut : ``'scrapy.spiderloader.SpiderLoader'``

La classe qui sera utilisée pour charger les spiders, et qui doit implémenter
l':ref:`topics-api-spiderloader`.

.. note:: Il s'agit d'un :ref:`paramètre pré-crawler <pre-crawler-settings>`.

.. setting:: SPIDER_LOADER_WARN_ONLY

SPIDER_LOADER_WARN_ONLY
-----------------------

Valeur par défaut : ``False``

Par défaut, lorsque Scrapy essaie d'importer les classes de spiders depuis
:setting:`SPIDER_MODULES`, il échoue bruyamment si une exception
``ImportError`` ou ``SyntaxError`` se produit. Mais vous pouvez choisir de
réduire cette exception au silence et de la transformer en simple
avertissement en définissant ``SPIDER_LOADER_WARN_ONLY = True``.

.. note:: Il s'agit d'un :ref:`paramètre pré-crawler <pre-crawler-settings>`.

.. setting:: SPIDER_MIDDLEWARES

SPIDER_MIDDLEWARES
------------------

Valeur par défaut : ``{}``

Un dict contenant les middlewares de spider activés dans votre projet, et leurs
ordres. Pour plus d'informations, voir :ref:`topics-spider-middleware-setting`.

.. setting:: SPIDER_MIDDLEWARES_BASE

SPIDER_MIDDLEWARES_BASE
-----------------------

Valeur par défaut :

.. code-block:: python

    {
        "scrapy.spidermiddlewares.start.StartSpiderMiddleware": 25,
        "scrapy.spidermiddlewares.httperror.HttpErrorMiddleware": 50,
        "scrapy.spidermiddlewares.referer.RefererMiddleware": 700,
        "scrapy.spidermiddlewares.urllength.UrlLengthMiddleware": 800,
        "scrapy.spidermiddlewares.depth.DepthMiddleware": 900,
        "scrapy.spidermiddlewares.metacopy.MetaCopyDetectionMiddleware": 999,
        "scrapy.spidermiddlewares.stickymeta.StickyMetaParamsMiddleware": 1000,
    }

Un dict contenant les middlewares de spider activés par défaut dans Scrapy, et
leurs ordres. Les ordres bas sont plus proches de l'engine, les ordres élevés
sont plus proches du spider. Pour plus d'informations, voir
:ref:`topics-spider-middleware-setting`.

.. setting:: SPIDER_MODULES

SPIDER_MODULES
--------------

Valeur par défaut : ``["<project name>.spiders"]`` (:ref:`repli <default-settings>` : ``[]``)

Une liste de modules dans lesquels Scrapy cherchera des spiders.

Exemple :

.. code-block:: python

    SPIDER_MODULES = ["mybot.spiders_prod", "mybot.spiders_dev"]

.. note:: Il s'agit d'un :ref:`paramètre pré-crawler <pre-crawler-settings>`.

.. setting:: STATS_CLASS

STATS_CLASS
-----------

Valeur par défaut : ``'scrapy.statscollectors.MemoryStatsCollector'``

La classe à utiliser pour collecter les statistiques, qui doit implémenter
l':ref:`topics-api-stats`.

.. setting:: STATS_DUMP

STATS_DUMP
----------

Valeur par défaut : ``True``

Affiche (dans le journal de Scrapy) les :ref:`statistiques de Scrapy
<topics-stats>` une fois que le spider a terminé.

Pour plus d'informations, voir : :ref:`topics-stats`.

.. setting:: STICKY_META_KEYS

STICKY_META_KEYS
----------------

Valeur par défaut : ``[]`` (liste vide)

Les clés de :attr:`Request.meta <scrapy.http.Request.meta>` à copier
automatiquement d'une réponse vers les requêtes de suite produites (yielded)
par son callback, gérées par
:class:`~scrapy.spidermiddlewares.stickymeta.StickyMetaParamsMiddleware`.

Les clés de métadonnées déjà définies sur une requête de suite ne sont pas
écrasées.

Par exemple, le spider suivant :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"

        async def start(self):
            start_url = "https://toscrape.com/"
            yield scrapy.Request(start_url, meta={"start_url": start_url})

        def parse(self, response):
            for a in response.css("a"):
                yield response.follow(
                    a,
                    meta={"start_url": response.meta["start_url"]},
                )
            yield {
                "url": response.url,
                "start_url": response.meta["start_url"],
            }

peut être réécrit comme suit en utilisant le paramètre
:setting:`STICKY_META_KEYS` :

.. code-block:: python

    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"
        custom_settings = {
            "STICKY_META_KEYS": ["start_url"],
        }

        async def start(self):
            start_url = "https://toscrape.com/"
            yield scrapy.Request(start_url, meta={"start_url": start_url})

        def parse(self, response):
            for a in response.css("a"):
                yield response.follow(a)
            yield {
                "url": response.url,
                "start_url": response.meta["start_url"],
            }

.. setting:: TELNETCONSOLE_ENABLED

TELNETCONSOLE_ENABLED
---------------------

Valeur par défaut : ``True`` (``False`` lorsque :setting:`TWISTED_REACTOR_ENABLED` vaut ``False``)

Un booléen qui indique si la :ref:`console telnet <topics-telnetconsole>` sera
activée (à condition que son extension soit elle aussi activée).

.. seealso:: :ref:`security-telnet`

.. setting:: TEMPLATES_DIR

TEMPLATES_DIR
-------------

Valeur par défaut : le répertoire ``templates`` à l'intérieur du module scrapy

Le répertoire où chercher les modèles (templates) lors de la création de
nouveaux projets avec la commande :command:`startproject` et de nouveaux
spiders avec la commande :command:`genspider`. Voir :ref:`spider-templates`.

Le nom du projet ne doit pas entrer en conflit avec le nom de fichiers ou de
répertoires personnalisés dans le sous-répertoire ``project``.

.. setting:: TWISTED_DNS_RESOLVER

TWISTED_DNS_RESOLVER
--------------------

Valeur par défaut : ``'scrapy.resolver.CachingThreadedResolver'``

La classe utilisée par Twisted pour résoudre les noms DNS. La classe par
défaut ``scrapy.resolver.CachingThreadedResolver`` permet de spécifier un délai
d'expiration pour les requêtes DNS via le paramètre :setting:`DNS_TIMEOUT`,
mais ne fonctionne qu'avec les adresses IPv4. Scrapy fournit un résolveur
alternatif, ``scrapy.resolver.CachingHostnameResolver``, qui prend en charge
les adresses IPv4/IPv6 mais ne tient pas compte du paramètre
:setting:`DNS_TIMEOUT`.

.. note::
    Ce paramètre n'a aucun effet lorsque :setting:`TWISTED_REACTOR_ENABLED`
    vaut ``False``.

.. note:: Il s'agit d'un :ref:`paramètre du reactor <reactor-settings>`.

.. setting:: TWISTED_REACTOR_ENABLED

TWISTED_REACTOR_ENABLED
-----------------------

Valeur par défaut : ``True``

Indique s'il faut installer et utiliser le reactor Twisted.

Si la valeur est ``True``, Scrapy utilisera le reactor Twisted et en installera
un selon la valeur du paramètre :setting:`TWISTED_REACTOR` lorsque c'est
approprié (par exemple lors d'une exécution via l':ref:`outil en ligne de
commande <topics-commands>`). C'est le mode traditionnel d'utilisation de
Scrapy.

Si la valeur est ``False``, Scrapy utilisera directement la boucle d'événements
asyncio et n'essaiera pas d'installer ni d'utiliser un reactor. Les
fonctionnalités qui nécessitent un reactor ne seront pas disponibles, mais les
API Twisted qui n'ont pas besoin d'un reactor, notamment
:class:`~twisted.internet.defer.Deferred` et
:class:`~twisted.python.failure.Failure`, resteront disponibles. En revanche,
les limitations liées aux reactors Twisted (comme l'impossibilité de démarrer
un reactor dans le même processus que celui où un reactor a déjà été démarré
puis arrêté) ne s'appliqueront pas. Ce mode est actuellement expérimental et
peut ne pas convenir à un usage en production. Il peut aussi ne pas être pris
en charge par du code tiers. Voir :ref:`asyncio-without-reactor` pour plus
d'informations sur ce mode.

.. note:: Il s'agit d'un :ref:`paramètre pré-crawler <pre-crawler-settings>`.

.. versionadded:: 2.15.0

.. setting:: TWISTED_REACTOR

TWISTED_REACTOR
---------------

Valeur par défaut : ``"twisted.internet.asyncioreactor.AsyncioSelectorReactor"``

Chemin d'import d'un :mod:`~twisted.internet.reactor` donné.

.. note::
    Ce paramètre n'a aucun effet lorsque :setting:`TWISTED_REACTOR_ENABLED`
    vaut ``False``.

Scrapy installera ce reactor si aucun autre reactor n'est encore installé,
comme lorsque le programme en ligne de commande ``scrapy`` est invoqué ou
lorsque l'on utilise la classe :class:`~scrapy.crawler.AsyncCrawlerProcess` ou
la classe :class:`~scrapy.crawler.CrawlerProcess`.

Si vous utilisez la classe :class:`~scrapy.crawler.AsyncCrawlerRunner` ou la
classe :class:`~scrapy.crawler.CrawlerRunner`, vous devez aussi installer
manuellement le bon reactor. Vous pouvez le faire avec
:func:`~scrapy.utils.reactor.install_reactor` :

.. autofunction:: scrapy.utils.reactor.install_reactor

Si un reactor est déjà installé, :func:`~scrapy.utils.reactor.install_reactor`
n'a aucun effet.

:class:`~scrapy.crawler.AsyncCrawlerRunner` et les autres classes similaires
lèvent une exception si le reactor installé ne correspond pas au paramètre
:setting:`TWISTED_REACTOR` ; par conséquent, avoir des imports de
:mod:`~twisted.internet.reactor` au niveau supérieur des fichiers du projet et
des bibliothèques tierces importées fera lever une exception à Scrapy lorsqu'il
vérifie quel reactor est installé.

Pour utiliser le reactor installé par Scrapy :

.. skip: next
.. code-block:: python

    import scrapy
    from twisted.internet import reactor


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        def __init__(self, *args, **kwargs):
            self.timeout = int(kwargs.pop("timeout", "60"))
            super().__init__(*args, **kwargs)

        async def start(self):
            reactor.callLater(self.timeout, self.stop)

            urls = ["https://quotes.toscrape.com/page/1"]
            for url in urls:
                yield scrapy.Request(url=url, callback=self.parse)

        def parse(self, response):
            for quote in response.css("div.quote"):
                yield {"text": quote.css("span.text::text").get()}

        def stop(self):
            self.crawler.engine.close_spider(self, "timeout")


ce code, qui lève une exception, devient :

.. code-block:: python

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        def __init__(self, *args, **kwargs):
            self.timeout = int(kwargs.pop("timeout", "60"))
            super().__init__(*args, **kwargs)

        async def start(self):
            from twisted.internet import reactor

            reactor.callLater(self.timeout, self.stop)

            urls = ["https://quotes.toscrape.com/page/1"]
            for url in urls:
                yield scrapy.Request(url=url, callback=self.parse)

        def parse(self, response):
            for quote in response.css("div.quote"):
                yield {"text": quote.css("span.text::text").get()}

        def stop(self):
            self.crawler.engine.close_spider(self, "timeout")


Si ce paramètre vaut ``None``, Scrapy utilisera le reactor existant si un
reactor est déjà installé, ou installera le reactor par défaut défini par
Twisted pour la plateforme courante.

.. versionchanged:: 2.13
   La valeur par défaut est passée de ``None`` à
   ``"twisted.internet.asyncioreactor.AsyncioSelectorReactor"``.

Pour des informations supplémentaires, voir :doc:`core/howto/choosing-reactor`.

.. note:: Il s'agit d'un :ref:`paramètre du reactor <reactor-settings>`.

.. setting:: URLLENGTH_LIMIT

URLLENGTH_LIMIT
---------------

Valeur par défaut : ``2083``

Périmètre : ``scrapy.spidermiddlewares.urllength``

La longueur maximale d'URL autorisée pour les URL crawlées.

Ce paramètre peut servir de condition d'arrêt dans le cas d'URL de longueur
toujours croissante, ce qui peut être causé par exemple par une erreur de
programmation, soit sur le serveur cible, soit dans votre code. Voir aussi
:setting:`REDIRECT_MAX_TIMES` et :setting:`DEPTH_LIMIT`.

Utilisez ``0`` pour autoriser des URL de n'importe quelle longueur.

La valeur par défaut est copiée de la `longueur maximale d'URL de Microsoft
Internet Explorer`_, même si ce paramètre existe pour des raisons différentes.

.. _longueur maximale d'URL de Microsoft Internet Explorer: https://web.archive.org/web/20250206050143/https://support.microsoft.com/en-us/topic/maximum-url-length-is-2-083-characters-in-internet-explorer-174e7c8a-6666-f4e0-6fd6-908b53c12246

.. setting:: USER_AGENT

USER_AGENT
----------

Valeur par défaut : ``"Scrapy/VERSION (+https://scrapy.org)"``

Le User-Agent par défaut à utiliser lors du crawl, sauf s'il est remplacé. Ce
user agent est aussi utilisé par
:class:`~scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware` si le
paramètre :setting:`ROBOTSTXT_USER_AGENT` vaut ``None`` et qu'aucun en-tête
User-Agent de remplacement n'est spécifié pour la requête.

Donnez-lui une valeur qui vous identifie, incluant une URL ou une adresse
e-mail où les propriétaires de sites web peuvent vous joindre, par exemple
``"MyProject (+https://example.com/bot)"``, afin qu'ils puissent vous demander
d'ajuster votre crawler plutôt que de le bloquer.

.. setting:: WARN_ON_GENERATOR_RETURN_VALUE

WARN_ON_GENERATOR_RETURN_VALUE
------------------------------

Valeur par défaut : ``True``

Lorsque ce paramètre est activé, Scrapy émet un avertissement si des méthodes
de callback basées sur des générateurs (comme ``parse``) contiennent des
instructions ``return`` avec des valeurs autres que ``None``. Cela aide à
détecter d'éventuelles erreurs lors du développement de spiders.

Désactivez ce paramètre pour éviter les erreurs de syntaxe qui peuvent survenir
lorsque l'on modifie dynamiquement le code source d'une fonction génératrice
pendant l'exécution, pour éviter l'analyse AST des fonctions de callback, ou
pour améliorer les performances dans des environnements de développement à
rechargement automatique.

.. only:: html

    Paramètres documentés ailleurs :
    --------------------------------

    Les paramètres suivants sont documentés ailleurs ; consultez chaque cas
    particulier pour savoir comment les activer et les utiliser.

    .. settingslist::

.. _Amazon web services: https://aws.amazon.com/
.. _Google Cloud Storage: https://cloud.google.com/storage/
