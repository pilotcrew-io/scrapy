.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/feed-exports.rst`.

.. _topics-feed-exports:

=================
Exports de feeds
=================

L'une des fonctionnalités les plus fréquemment nécessaires lorsqu'on implémente
des scrapers est de stocker correctement les données scrapées et, très souvent,
cela signifie générer un « fichier d'export » avec les données scrapées
(couramment appelé un « feed d'export ») destiné à être consommé par d'autres
systèmes.

Scrapy fournit cette fonctionnalité nativement grâce aux exports de feeds
(Feed Exports), qui vous permettent de générer des feeds avec les items
scrapés, en utilisant plusieurs formats de sérialisation et backends de
stockage.

Cette page fournit une documentation détaillée pour toutes les fonctionnalités
d'export de feeds. Si vous cherchez un guide pas à pas, consultez les
`guides d'export de Zyte`_.

.. _guides d'export de Zyte: https://docs.zyte.com/web-scraping/guides/export/index.html#exporting-scraped-data

.. _topics-feed-format:

Formats de sérialisation
=========================

Pour sérialiser les données scrapées, les exports de feeds utilisent les
:ref:`exporteurs d'items <topics-exporters>`. Ces formats sont pris en charge
nativement :

-   :ref:`topics-feed-format-json`
-   :ref:`topics-feed-format-jsonlines`
-   :ref:`topics-feed-format-csv`
-   :ref:`topics-feed-format-xml`

Mais vous pouvez aussi étendre les formats pris en charge grâce au paramètre
:setting:`FEED_EXPORTERS`.

.. _topics-feed-format-json:

JSON
----

-   Valeur pour la clé ``format`` dans le paramètre :setting:`FEEDS` :
    ``json``

-   Exporteur utilisé : :class:`~scrapy.exporters.JsonItemExporter`

-   Consultez :ref:`cet avertissement <json-with-large-data>` si vous utilisez
    du JSON avec de gros feeds.

.. _topics-feed-format-jsonlines:

JSON lines
----------

-   Valeur pour la clé ``format`` dans le paramètre :setting:`FEEDS` :
    ``jsonlines``
-   Exporteur utilisé : :class:`~scrapy.exporters.JsonLinesItemExporter`

.. _topics-feed-format-csv:

CSV
---

-   Valeur pour la clé ``format`` dans le paramètre :setting:`FEEDS` :
    ``csv``

-   Exporteur utilisé : :class:`~scrapy.exporters.CsvItemExporter`

-   Pour préciser les colonnes à exporter, leur ordre et leurs noms de
    colonnes, utilisez :setting:`FEED_EXPORT_FIELDS`. D'autres exporteurs de
    feeds peuvent aussi utiliser cette option, mais elle est particulièrement
    importante pour le CSV car, contrairement à beaucoup d'autres formats
    d'export, le CSV utilise un en-tête fixe.

.. _topics-feed-format-xml:

XML
---

-   Valeur pour la clé ``format`` dans le paramètre :setting:`FEEDS` :
    ``xml``
-   Exporteur utilisé : :class:`~scrapy.exporters.XmlItemExporter`

.. _topics-feed-format-pickle:

Pickle
------

-   Valeur pour la clé ``format`` dans le paramètre :setting:`FEEDS` :
    ``pickle``
-   Exporteur utilisé : :class:`~scrapy.exporters.PickleItemExporter`

.. _topics-feed-format-marshal:

Marshal
-------

-   Valeur pour la clé ``format`` dans le paramètre :setting:`FEEDS` :
    ``marshal``
-   Exporteur utilisé : :class:`~scrapy.exporters.MarshalItemExporter`

.. _topics-feed-storage:

Stockages
=========

Lorsque vous utilisez les exports de feeds, vous définissez où stocker le feed
en utilisant une ou plusieurs URIs_ (via le paramètre :setting:`FEEDS`). Les
exports de feeds prennent en charge plusieurs types de backends de stockage,
définis par le schéma de l'URI.

Les backends de stockage pris en charge nativement sont :

-   :ref:`topics-feed-storage-fs`
-   :ref:`feed-storage-ftp`
-   :ref:`feed-storage-ftps`
-   :ref:`topics-feed-storage-s3` (nécessite l'extra :ref:`s3 <extras>`)
-   :ref:`topics-feed-storage-gcs` (nécessite l'extra :ref:`gcs <extras>`)
-   :ref:`topics-feed-storage-stdout`

Certains backends de stockage peuvent être indisponibles si les
:ref:`extras <extras>` nécessaires ne sont pas installés. Par exemple, le
backend S3 nécessite l'extra :ref:`s3 <extras>`.

.. _topics-feed-uri-params:

Paramètres de l'URI de stockage
================================

L'URI de stockage peut aussi contenir des paramètres qui sont remplacés au
moment de la création du feed. Ces paramètres sont :

-   ``%(time)s`` - remplacé par un horodatage au moment de la création du feed
-   ``%(name)s`` - remplacé par le nom du spider

Tout autre paramètre nommé est remplacé par l'attribut du spider portant le
même nom. Par exemple, ``%(site_id)s`` serait remplacé par l'attribut
``spider.site_id`` au moment où le feed est créé.

Voici quelques exemples pour illustrer :

-   Stocker en FTP avec un répertoire par spider :

    -   ``ftp://user:password@ftp.example.com/scraping/feeds/%(name)s/%(time)s.json``

-   Stocker en S3 avec un répertoire par spider :

    -   ``s3://mybucket/scraping/feeds/%(name)s/%(time)s.json``

.. note:: Les :ref:`arguments du spider <spiderargs>` deviennent des attributs
          du spider, ils peuvent donc eux aussi être utilisés comme paramètres
          de l'URI de stockage.

.. note:: Seuls les paramètres ``%(...)s`` sont remplacés. Tout autre
          caractère pourcentage est conservé tel quel, ce qui fait que les
          URIs encodées en pourcentage (par exemple ``%20`` pour un espace, ou
          des identifiants FTP encodés en pourcentage) et les clés
          :class:`pathlib.Path` contenant des paramètres ``%(...)s``
          fonctionnent toutes deux comme attendu.


.. _topics-feed-storage-backends:

Backends de stockage
=====================

.. _topics-feed-storage-fs:

Système de fichiers local
--------------------------

Les feeds sont stockés dans le système de fichiers local.

-   Schéma d'URI : ``file``
-   Exemple d'URI : ``file:///tmp/export.csv``
-   Bibliothèques externes nécessaires : aucune

Notez que pour le stockage sur système de fichiers local, vous pouvez omettre
le schéma si vous indiquez un chemin (par exemple ``/tmp/export.csv``). Vous
pouvez aussi, comme alternative, utiliser un objet :class:`pathlib.Path`.

.. _topics-feed-storage-ftp:
.. _feed-storage-ftp:

FTP
---

Les feeds sont stockés sur un serveur FTP.

-   Schéma d'URI : ``ftp``
-   Exemple d'URI : ``ftp://user:pass@ftp.example.com/path/to/export.csv``
-   Bibliothèques externes nécessaires : aucune

FTP envoie les identifiants et les données en clair. Utilisez plutôt
:ref:`feed-storage-ftps` lorsque c'est possible.

FTP prend en charge deux modes de connexion différents : `actif ou passif
<https://stackoverflow.com/a/1699163>`_. Scrapy utilise par défaut le mode de
connexion passif. Pour utiliser le mode de connexion actif à la place, réglez
le paramètre :setting:`FEED_STORAGE_FTP_ACTIVE` à ``True``.

La valeur par défaut de la clé ``overwrite`` dans :setting:`FEEDS` pour ce
backend de stockage est : ``True``.

.. caution:: La valeur ``True`` pour ``overwrite`` vous fera perdre la version
     précédente de vos données.

Ce backend de stockage utilise la :ref:`livraison différée de fichiers
<delayed-file-delivery>`.


.. _feed-storage-ftps:

FTPS
----

Les feeds sont stockés sur un serveur FTP, via une connexion TLS, avec
vérification du certificat du serveur.

.. versionadded:: 2.18.0

-   Schéma d'URI : ``ftps``
-   Exemple d'URI : ``ftps://user:pass@ftp.example.com/path/to/export.csv``
-   Bibliothèques externes nécessaires : aucune

Voir :ref:`feed-storage-ftp` pour les modes de connexion, la valeur par
défaut de ``overwrite`` et la livraison de fichiers.

.. note:: Pour SFTP, un protocole distinct construit sur SSH, utilisez
          `scrapy-feedexporter-sftp
          <https://github.com/scrapy-plugins/scrapy-feedexporter-sftp>`_.


.. _topics-feed-storage-s3:

S3
--

Les feeds sont stockés sur `Amazon S3`_.

-   Schéma d'URI : ``s3``

-   Exemples d'URI :

    -   ``s3://mybucket/path/to/export.csv``

    -   ``s3://aws_key:aws_secret@mybucket/path/to/export.csv``

-   Extras nécessaires : :ref:`s3 <extras>`

Les identifiants AWS peuvent être passés en tant qu'utilisateur/mot de passe
dans l'URI, ou ils peuvent être passés via les paramètres suivants :

-   :setting:`AWS_ACCESS_KEY_ID`
-   :setting:`AWS_SECRET_ACCESS_KEY`
-   :setting:`AWS_SESSION_TOKEN` (nécessaire uniquement pour les
    `identifiants de sécurité temporaires`_)

.. _identifiants de sécurité temporaires: https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html

Vous pouvez aussi définir une ACL personnalisée, un endpoint personnalisé, un
nom de région et une taille de pool de connexions pour les feeds exportés en
utilisant ces paramètres :

-   :setting:`FEED_STORAGE_S3_ACL`
-   :setting:`AWS_ENDPOINT_URL`
-   :setting:`AWS_REGION_NAME`
-   :setting:`AWS_MAX_POOL_CONNECTIONS`

La valeur par défaut de la clé ``overwrite`` dans :setting:`FEEDS` pour ce
backend de stockage est : ``True``.

.. caution:: La valeur ``True`` pour ``overwrite`` vous fera perdre la version
     précédente de vos données.

Ce backend de stockage utilise la :ref:`livraison différée de fichiers
<delayed-file-delivery>`.


.. _topics-feed-storage-gcs:

Google Cloud Storage (GCS)
----------------------------

Les feeds sont stockés sur `Google Cloud Storage`_.

-   Schéma d'URI : ``gs``

-   Exemples d'URI :

    -   ``gs://mybucket/path/to/export.csv``

-   Extras nécessaires : :ref:`gcs <extras>`

Pour plus d'informations sur l'authentification, référez-vous à la
`documentation Google Cloud <https://docs.cloud.google.com/docs/authentication>`_.

Vous pouvez définir un *Project ID* et une *Access Control List (ACL)* via les
paramètres suivants :

-   :setting:`FEED_STORAGE_GCS_ACL`
-   :setting:`GCS_PROJECT_ID`

La valeur par défaut de la clé ``overwrite`` dans :setting:`FEEDS` pour ce
backend de stockage est : ``True``.

.. caution:: La valeur ``True`` pour ``overwrite`` vous fera perdre la version
     précédente de vos données.

L'ajout en fin de fichier (``overwrite: False``) transforme le feed en un
`objet composite`_, qui possède une somme de contrôle CRC32C mais pas de hash
MD5.

.. versionadded:: VERSION
   Prise en charge de l'ajout en fin de fichier.

Ce backend de stockage utilise la :ref:`livraison différée de fichiers
<delayed-file-delivery>`.



.. _topics-feed-storage-stdout:

Sortie standard
----------------

Les feeds sont écrits sur la sortie standard du processus Scrapy.

-   Schéma d'URI : ``stdout``
-   Exemple d'URI : ``stdout:``
-   Bibliothèques externes nécessaires : aucune


.. _delayed-file-delivery:

Livraison différée de fichiers
--------------------------------

Comme indiqué ci-dessus, certains des backends de stockage décrits utilisent
une livraison différée de fichiers.

Ces backends de stockage n'envoient pas les items vers l'URI du feed au fur et
à mesure qu'ils sont scrapés. Scrapy écrit plutôt les items dans un fichier
local temporaire, et ce n'est qu'une fois que tout le contenu du fichier a été
écrit (c'est-à-dire à la fin du crawl) que ce fichier est envoyé vers l'URI du
feed.

Si vous voulez que la livraison des items commence plus tôt en utilisant l'un
de ces backends de stockage, utilisez :setting:`FEED_EXPORT_BATCH_ITEM_COUNT`
pour répartir les items de sortie sur plusieurs fichiers, avec le nombre
maximal d'items par fichier indiqué. Ainsi, dès qu'un fichier atteint le
nombre maximal d'items, ce fichier est livré à l'URI du feed, ce qui permet à
la livraison des items de commencer bien avant la fin du crawl.


.. _item-filter:

Filtrage des items
====================

Vous pouvez filtrer les items que vous souhaitez autoriser pour un feed
particulier en utilisant l'option ``item_classes`` dans les
:ref:`options de feed <feed-options>`. Seuls les items des types indiqués
seront ajoutés au feed.

L'option ``item_classes`` est implémentée par la classe
:class:`~scrapy.extensions.feedexport.ItemFilter`, qui est la valeur par
défaut de l'option de feed ``item_filter`` (:ref:`voir <feed-options>`).

Vous pouvez créer votre propre classe de filtrage personnalisée en
implémentant la méthode ``accepts`` de
:class:`~scrapy.extensions.feedexport.ItemFilter`, qui prend
``feed_options`` en argument.

Par exemple :

.. code-block:: python

    class MyCustomFilter:
        def __init__(self, feed_options):
            self.feed_options = feed_options

        def accepts(self, item):
            if "field1" in item and item["field1"] == "expected_data":
                return True
            return False


Vous pouvez assigner votre classe de filtrage personnalisée à l'option de feed
``item_filter`` (:ref:`voir <feed-options>`). Consultez :setting:`FEEDS` pour
des exemples.

ItemFilter
----------

.. autoclass:: scrapy.extensions.feedexport.ItemFilter
   :members:


.. _post-processing:

Post-traitement
================

Scrapy propose une option permettant d'activer des plugins pour post-traiter
les feeds avant qu'ils ne soient exportés vers les stockages de feeds. En plus
d'utiliser les :ref:`plugins intégrés <builtin-plugins>`, vous pouvez créer
vos propres :ref:`plugins <custom-plugins>`.

Ces plugins peuvent être activés via l'option ``postprocessing`` d'un feed.
Cette option doit recevoir une liste de plugins de post-traitement, dans
l'ordre dans lequel vous voulez que le feed soit traité. Ces plugins peuvent
être déclarés soit sous forme de chaîne d'import, soit avec la classe importée
du plugin. Des paramètres peuvent être passés aux plugins via les options de
feed. Consultez les :ref:`options de feed <feed-options>` pour des exemples.

.. _builtin-plugins:

Plugins intégrés
-----------------

.. autoclass:: scrapy.extensions.postprocessing.GzipPlugin

.. autoclass:: scrapy.extensions.postprocessing.LZMAPlugin

.. autoclass:: scrapy.extensions.postprocessing.Bz2Plugin

.. _custom-plugins:

Plugins personnalisés
-----------------------

Chaque plugin est une classe qui doit implémenter les méthodes suivantes :

.. method:: __init__(self, file, feed_options)

    Initialise le plugin.

    :param file: objet de type fichier ayant au moins les méthodes `write`,
        `tell` et `close` implémentées

    :param feed_options: options :ref:`propres au feed <feed-options>`
    :type feed_options: :class:`dict`

.. method:: write(self, data)

   Traite et écrit `data` (:class:`bytes` ou :class:`memoryview`) dans le
   fichier cible du plugin. Doit retourner le nombre d'octets écrits.

.. method:: close(self)

    Nettoie le plugin.

    Par exemple, vous pourriez vouloir fermer un wrapper de fichier que vous
    auriez utilisé pour compresser les données écrites dans le fichier reçu
    dans la méthode ``__init__``.

    .. warning:: Ne fermez pas le fichier depuis la méthode ``__init__``.

Pour passer un paramètre à votre plugin, utilisez les :ref:`options de feed
<feed-options>`. Vous pouvez ensuite accéder à ces paramètres depuis la
méthode ``__init__`` de votre plugin.


Paramètres
==========

Voici les paramètres utilisés pour configurer les exports de feeds :

-   :setting:`FEEDS` (obligatoire)
-   :setting:`FEED_EXPORT_ENCODING`
-   :setting:`FEED_STORE_EMPTY`
-   :setting:`FEED_EXPORT_FIELDS`
-   :setting:`FEED_EXPORT_INDENT`
-   :setting:`FEED_STORAGES`
-   :setting:`FEED_STORAGE_FTP_ACTIVE`
-   :setting:`FEED_STORAGE_S3_ACL`
-   :setting:`FEED_EXPORTERS`
-   :setting:`FEED_EXPORT_BATCH_ITEM_COUNT`

.. setting:: FEEDS

FEEDS
-----

Valeur par défaut : ``{}``

Un dictionnaire dans lequel chaque clé est une URI de feed (ou un objet
:class:`pathlib.Path`) et chaque valeur est un dictionnaire imbriqué
contenant les paramètres de configuration pour ce feed en particulier.

Ce paramètre est nécessaire pour activer la fonctionnalité d'export de feeds.

Consultez :ref:`topics-feed-storage-backends` pour les schémas d'URI pris en
charge.

Par exemple :

.. skip: next

.. code-block:: python

    {
        "items.json": {
            "format": "json",
            "encoding": "utf8",
            "store_empty": False,
            "item_classes": [MyItemClass1, "myproject.items.MyItemClass2"],
            "fields": None,
            "indent": 4,
            "item_export_kwargs": {
                "export_empty_fields": True,
            },
        },
        "/home/user/documents/items.xml": {
            "format": "xml",
            "fields": ["name", "price"],
            "item_filter": MyCustomFilter1,
            "encoding": "latin1",
            "indent": 8,
        },
        pathlib.Path("items.csv.gz"): {
            "format": "csv",
            "fields": ["price", "name"],
            "item_filter": "myproject.filters.MyCustomFilter2",
            "postprocessing": [MyPlugin1, "scrapy.extensions.postprocessing.GzipPlugin"],
            "gzip_compresslevel": 5,
        },
    }

.. _feed-options:

Voici la liste des clés acceptées et du paramètre utilisé comme valeur de
repli si cette clé n'est pas fournie pour une définition de feed donnée :

-   ``format`` : le :ref:`format de sérialisation <topics-feed-format>`.

    S'il n'est pas défini, il est déduit de l'extension du fichier de l'URI du
    feed, par exemple ``json`` pour une URI se terminant par :file:`.json`.
    Il est obligatoire si cette déduction est impossible.

-   ``batch_item_count`` : se replie sur
    :setting:`FEED_EXPORT_BATCH_ITEM_COUNT`.

-   ``encoding`` : se replie sur :setting:`FEED_EXPORT_ENCODING`.

-   ``fields`` : se replie sur :setting:`FEED_EXPORT_FIELDS`.

-   ``item_classes`` : liste de :ref:`classes d'items <topics-items>` à
    exporter.

    Si elle n'est pas définie ou vide, tous les items sont exportés.

-   ``item_filter`` : une :ref:`classe de filtrage <item-filter>` pour
    filtrer les items à exporter.

    :class:`~scrapy.extensions.feedexport.ItemFilter` est utilisée par
    défaut.

-   ``indent`` : se replie sur :setting:`FEED_EXPORT_INDENT`.

-   ``item_export_kwargs`` : :class:`dict` d'arguments nommés pour la
    :ref:`classe d'exporteur d'items <topics-exporters>` correspondante.

-   ``overwrite`` : indique s'il faut écraser le fichier s'il existe déjà
    (``True``) ou ajouter à son contenu (``False``).

    La valeur par défaut dépend du :ref:`backend de stockage
    <topics-feed-storage-backends>` :

    -   :ref:`topics-feed-storage-fs` : ``False``

    -   :ref:`feed-storage-ftp` et :ref:`feed-storage-ftps` : ``True``

        .. note:: Certains serveurs FTP peuvent ne pas prendre en charge
                  l'ajout en fin de fichier (la commande FTP ``APPE``).

    -   :ref:`topics-feed-storage-s3` : ``True`` (l'ajout en fin de fichier
        n'est pas pris en charge)

    -   :ref:`topics-feed-storage-gcs` : ``True``

    -   :ref:`topics-feed-storage-stdout` : ``False`` (l'écrasement n'est pas
        pris en charge)

-   ``store_empty`` : se replie sur :setting:`FEED_STORE_EMPTY`.

-   ``uri_params`` : se replie sur :setting:`FEED_URI_PARAMS`.

-   ``postprocessing`` : liste de :ref:`plugins <post-processing>` à utiliser
    pour le post-traitement.

    Les plugins seront utilisés dans l'ordre de la liste fournie.

.. setting:: FEED_EXPORT_ENCODING

FEED_EXPORT_ENCODING
---------------------

Valeur par défaut : ``"utf-8"`` (:ref:`repli <default-settings>` :
``None``)

L'encodage à utiliser pour le feed.

S'il est réglé à ``None``, UTF-8 est utilisé pour tout sauf pour la sortie
JSON, qui utilise pour des raisons historiques un encodage numérique sûr
(séquences ``\uXXXX``).

Utilisez ``"utf-8"`` si vous voulez de l'UTF-8 pour le JSON également.

.. setting:: FEED_EXPORT_FIELDS

FEED_EXPORT_FIELDS
--------------------

Valeur par défaut : ``None``

Utilisez le paramètre ``FEED_EXPORT_FIELDS`` pour définir les champs à
exporter, leur ordre et leurs noms de sortie. Consultez
:attr:`BaseItemExporter.fields_to_export
<scrapy.exporters.BaseItemExporter.fields_to_export>` pour plus
d'informations.

.. setting:: FEED_EXPORT_INDENT

FEED_EXPORT_INDENT
--------------------

Valeur par défaut : ``0``

Nombre d'espaces utilisés pour indenter la sortie à chaque niveau. Si
``FEED_EXPORT_INDENT`` est un entier positif ou nul, les éléments de tableau
et les membres d'objet seront affichés avec une belle mise en forme, avec ce
niveau d'indentation. Un niveau d'indentation de ``0`` (la valeur par
défaut), ou négatif, mettra chaque item sur une nouvelle ligne. ``None``
sélectionne la représentation la plus compacte.

Actuellement implémenté seulement par
:class:`~scrapy.exporters.JsonItemExporter` et
:class:`~scrapy.exporters.XmlItemExporter`, c'est-à-dire lorsque vous exportez
vers ``.json`` ou ``.xml``.

.. setting:: FEED_STORE_EMPTY

FEED_STORE_EMPTY
------------------

Valeur par défaut : ``True``

Indique s'il faut exporter les feeds vides (c'est-à-dire les feeds sans
items). Si ``False``, et qu'il n'y a aucun item à exporter, aucun nouveau
fichier n'est créé et les fichiers existants ne sont pas modifiés, même si
l'option de feed :ref:`overwrite <feed-options>` est activée.

.. setting:: FEED_STORAGES

FEED_STORAGES
--------------

Valeur par défaut : ``{}``

Un dictionnaire contenant des backends de stockage de feeds supplémentaires
pris en charge par votre projet. Les clés sont des schémas d'URI et les
valeurs sont des chemins vers des classes de stockage.

.. setting:: FEED_STORAGE_FTP_ACTIVE

FEED_STORAGE_FTP_ACTIVE
-------------------------

Valeur par défaut : ``False``

Indique s'il faut utiliser le mode de connexion actif lors de l'export de
feeds vers un serveur FTP (``True``), ou utiliser plutôt le mode de connexion
passif (``False``, valeur par défaut).

Pour plus d'informations sur les modes de connexion FTP, consultez
`Quelle est la différence entre le FTP actif et le FTP passif ?
<https://stackoverflow.com/a/1699163>`_.

.. setting:: FEED_STORAGE_S3_ACL

FEED_STORAGE_S3_ACL
---------------------

Valeur par défaut : ``''`` (chaîne vide)

Une chaîne contenant une ACL personnalisée pour les feeds exportés vers
Amazon S3 par votre projet.

Pour une liste complète des valeurs disponibles, consultez la section
`Canned ACL`_ de la documentation Amazon S3.

.. setting:: FEED_STORAGES_BASE

FEED_STORAGES_BASE
--------------------

Valeur par défaut :

.. code-block:: python

    {
        "": "scrapy.extensions.feedexport.FileFeedStorage",
        "file": "scrapy.extensions.feedexport.FileFeedStorage",
        "stdout": "scrapy.extensions.feedexport.StdoutFeedStorage",
        "s3": "scrapy.extensions.feedexport.S3FeedStorage",
        "gs": "scrapy.extensions.feedexport.GCSFeedStorage",
        "ftp": "scrapy.extensions.feedexport.FTPFeedStorage",
        "ftps": "scrapy.extensions.feedexport.FTPFeedStorage",
    }

Un dictionnaire contenant les backends de stockage de feeds intégrés pris en
charge par Scrapy. Vous pouvez désactiver n'importe lequel de ces backends en
assignant ``None`` à leur schéma d'URI dans :setting:`FEED_STORAGES`. Par
exemple, pour désactiver le backend de stockage FTP intégré (sans le
remplacer), placez ceci dans votre ``settings.py`` :

.. code-block:: python

    FEED_STORAGES = {
        "ftp": None,
    }

.. setting:: FEED_EXPORTERS

FEED_EXPORTERS
----------------

Valeur par défaut : ``{}``

Un dictionnaire contenant des exporteurs supplémentaires pris en charge par
votre projet. Les clés sont des formats de sérialisation et les valeurs sont
des chemins vers des classes d'\ :ref:`exporteur d'items <topics-exporters>`.

.. setting:: FEED_EXPORTERS_BASE

FEED_EXPORTERS_BASE
---------------------
Valeur par défaut :

.. code-block:: python

    {
        "json": "scrapy.exporters.JsonItemExporter",
        "jsonlines": "scrapy.exporters.JsonLinesItemExporter",
        "jsonl": "scrapy.exporters.JsonLinesItemExporter",
        "jl": "scrapy.exporters.JsonLinesItemExporter",
        "csv": "scrapy.exporters.CsvItemExporter",
        "xml": "scrapy.exporters.XmlItemExporter",
        "marshal": "scrapy.exporters.MarshalItemExporter",
        "pickle": "scrapy.exporters.PickleItemExporter",
    }

Un dictionnaire contenant les exporteurs de feeds intégrés pris en charge par
Scrapy. Vous pouvez désactiver n'importe lequel de ces exporteurs en
assignant ``None`` à leur format de sérialisation dans
:setting:`FEED_EXPORTERS`. Par exemple, pour désactiver l'exporteur CSV
intégré (sans le remplacer), placez ceci dans votre ``settings.py`` :

.. code-block:: python

    FEED_EXPORTERS = {
        "csv": None,
    }


.. setting:: FEED_EXPORT_BATCH_ITEM_COUNT

FEED_EXPORT_BATCH_ITEM_COUNT
-------------------------------

Valeur par défaut : ``0``

Si on lui assigne un nombre entier supérieur à ``0``, Scrapy génère
plusieurs fichiers de sortie, en stockant jusqu'au nombre d'items indiqué
dans chaque fichier de sortie.

Lors de la génération de plusieurs fichiers de sortie, vous devez utiliser au
moins l'un des espaces réservés suivants dans l'URI du feed pour indiquer
comment les différents noms de fichiers de sortie sont générés :

* ``%(batch_time)s`` - remplacé par un horodatage au moment de la création du
  feed (par exemple ``2020-03-28T14-45-08.237134``)

* ``%(batch_id)d`` - remplacé par le numéro de séquence du lot, à partir de 1.

  Utilisez le :ref:`formatage de chaînes façon printf
  <python:old-string-formatting>` pour modifier le format du nombre. Par
  exemple, pour faire de l'ID de lot un nombre à 5 chiffres en introduisant
  des zéros de tête si nécessaire, utilisez ``%(batch_id)05d`` (par exemple,
  ``3`` devient ``00003``, ``123`` devient ``00123``).

Par exemple, si vos paramètres incluent :

.. code-block:: python

    FEED_EXPORT_BATCH_ITEM_COUNT = 100

Et que votre ligne de commande :command:`crawl` est ::

    scrapy crawl spidername -o "dirname/%(batch_id)d-filename%(batch_time)s.json"

La ligne de commande ci-dessus peut générer une arborescence de répertoires
comme ceci ::

    ->projectname
    -->dirname
    --->1-filename2020-03-28T14-45-08.237134.json
    --->2-filename2020-03-28T14-45-09.148903.json
    --->3-filename2020-03-28T14-45-10.046092.json

Où le premier et le deuxième fichier contiennent exactement 100 items. Le
dernier contient 100 items ou moins.


.. setting:: FEED_URI_PARAMS

FEED_URI_PARAMS
-----------------

Valeur par défaut : ``None``

Une chaîne contenant le chemin d'import d'une fonction qui définit les
paramètres à appliquer, via le :ref:`formatage de chaînes façon printf
<python:old-string-formatting>`, à l'URI du feed.

La signature de la fonction doit être la suivante :

.. function:: scrapy.extensions.feedexport.uri_params(params, spider)

   Retourne un :class:`dict` de paires clé-valeur à appliquer à l'URI du feed
   via le :ref:`formatage de chaînes façon printf
   <python:old-string-formatting>`.

   :param params: paires clé-valeur par défaut

        Plus précisément :

        -   ``batch_id`` : ID du lot de fichiers. Voir
            :setting:`FEED_EXPORT_BATCH_ITEM_COUNT`.

            Si :setting:`FEED_EXPORT_BATCH_ITEM_COUNT` vaut ``0``,
            ``batch_id`` vaut toujours ``1``.

        -   ``batch_time`` : date et heure UTC, au format ISO, avec les
            caractères ``:`` remplacés par ``-``.

            Voir :setting:`FEED_EXPORT_BATCH_ITEM_COUNT`.

        -   ``time`` : ``batch_time``, avec les microsecondes mises à ``0``.
   :type params: dict

   :param spider: spider source des items du feed
   :type spider: scrapy.Spider

   .. caution:: La fonction doit retourner un nouveau dictionnaire plutôt que
                de modifier ``params`` sur place.

Par exemple, pour inclure le :attr:`nom <scrapy.Spider.name>` du spider
source dans l'URI du feed :

#.  Définissez la fonction suivante quelque part dans votre projet :

    .. code-block:: python

        # myproject/utils.py
        def uri_params(params, spider):
            return {**params, "spider_name": spider.name}

#.  Faites pointer :setting:`FEED_URI_PARAMS` vers cette fonction dans vos
    paramètres :

    .. code-block:: python

        # myproject/settings.py
        FEED_URI_PARAMS = "myproject.utils.uri_params"

#.  Utilisez ``%(spider_name)s`` dans l'URI de votre feed ::

        scrapy crawl <spider_name> -o "%(spider_name)s.jsonl"


.. _URIs: https://en.wikipedia.org/wiki/Uniform_Resource_Identifier
.. _Amazon S3: https://aws.amazon.com/s3/
.. _Canned ACL: https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html#canned-acl
.. _objet composite: https://docs.cloud.google.com/storage/docs/composite-objects
.. _Google Cloud Storage: https://cloud.google.com/storage/
