.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/request-response.rst`.

.. _topics-request-response:

====================
Requêtes et réponses
====================

.. module:: scrapy.http

Scrapy utilise des objets :class:`~scrapy.Request` et :class:`Response` pour
explorer (crawler) les sites web.

En général, les objets :class:`~scrapy.Request` sont générés dans les spiders
et circulent à travers le système jusqu'à atteindre le downloader. Celui-ci
exécute la requête et renvoie un objet :class:`Response`, qui revient vers le
spider qui a émis la requête.

Les classes :class:`~scrapy.Request` et :class:`Response` ont toutes deux des
sous-classes qui ajoutent des fonctionnalités dont les classes de base n'ont
pas besoin. Elles sont décrites plus bas dans
:ref:`topics-request-response-ref-request-subclasses` et
:ref:`topics-request-response-ref-response-subclasses`.


Objets Request
==============

.. autoclass:: scrapy.Request

    :param url: l'URL de cette requête

        Si l'URL n'est pas valide, une exception :exc:`ValueError` est levée.
    :type url: str

    :param callback: définit :attr:`callback`, vaut ``None`` par défaut.
    :type callback: Callable[Concatenate[Response, ...], Any] | None

    :param method: la méthode HTTP de cette requête. Vaut ``'GET'`` par défaut.
    :type method: str

    :param meta: les valeurs initiales de l'attribut :attr:`.Request.meta`. Si
       elle est fournie, la dict passée dans ce paramètre fait l'objet d'une
       copie superficielle (shallow copy).
    :type meta: dict

    :param body: le corps de la requête. Si une chaîne de caractères est
      passée, elle est encodée en bytes avec l'``encoding`` passé (qui vaut
      ``utf-8`` par défaut). Si ``body`` n'est pas fourni, un objet bytes vide
      est stocké. Quel que soit le type de cet argument, la valeur finalement
      stockée est un objet bytes (jamais une chaîne ni ``None``).
    :type body: bytes or str

    :param headers: les en-têtes de cette requête. Les valeurs de la dict
       peuvent être des chaînes (pour les en-têtes à valeur unique) ou des
       listes (pour les en-têtes à valeurs multiples). Si ``None`` est passé
       comme valeur, l'en-tête HTTP n'est pas envoyé du tout.

       .. caution:: Les cookies définis via l'en-tête ``Cookie`` ne sont pas pris
           en compte par le :ref:`middleware de cookies <cookies>`. Si vous
           avez besoin de définir des cookies pour une requête, utilisez
           l'argument ``cookies``.

    :type headers: dict

    :param cookies: les cookies de la requête, sous la forme d'une dict de noms
        et de valeurs de cookies, ou d'une liste de dicts contenant chacune un
        cookie. Voir :ref:`cookies`.
    :type cookies: dict or list

    :param encoding: l'encodage de cette requête (vaut ``'utf-8'`` par défaut).
       Cet encodage est utilisé pour appliquer le percent-encoding à l'URL et
       pour convertir le corps en bytes (s'il est fourni sous forme de chaîne).

       Pour désactiver le percent-encoding de l'URL pour une requête, utilisez
       la clé de meta de requête :reqmeta:`verbatim_url`.
    :type encoding: str

    :param priority: définit :attr:`priority`, vaut ``0`` par défaut.
    :type priority: int

    :param dont_filter: définit :attr:`dont_filter`, vaut ``False`` par défaut.
    :type dont_filter: bool

    :param errback: définit :attr:`errback`, vaut ``None`` par défaut.
    :type errback: Callable[[Failure], Any] | None

    :param flags:  Drapeaux (flags) attachés à la requête ; ils peuvent servir à la journalisation ou à des usages similaires.
    :type flags: list

    :param cb_kwargs: Une dict de données arbitraires qui seront passées comme arguments nommés au callback de la Request.
    :type cb_kwargs: dict

    .. attribute:: Request.url

        Une chaîne de caractères contenant l'URL de cette requête.

        Gardez à l'esprit que cet attribut contient l'URL échappée : elle peut
        donc différer de l'URL passée à la méthode ``__init__()``.

        Si :reqmeta:`verbatim_url` vaut ``True``, l'URL est conservée telle
        qu'elle a été passée à ``__init__()``.

        Cet attribut est en lecture seule. Pour changer l'URL d'une Request,
        utilisez :meth:`replace`.

    .. attribute:: Request.method

        Une chaîne représentant la méthode HTTP de la requête. Elle est
        garantie d'être en majuscules. Exemples : ``"GET"``, ``"POST"``,
        ``"PUT"``, etc.

    .. attribute:: Request.headers

        Un objet de type dictionnaire (:class:`scrapy.http.headers.Headers`) qui
        contient les en-têtes de la requête.

    .. attribute:: Request.body

        Le corps de la requête, sous forme de bytes.

        Cet attribut est en lecture seule. Pour changer le corps d'une Request,
        utilisez :meth:`replace`.

    .. autoattribute:: callback

    .. autoattribute:: errback

    .. autoattribute:: priority

    .. attribute:: Request.cb_kwargs

        Un dictionnaire qui contient des métadonnées arbitraires pour cette
        requête. Son contenu sera passé au callback de la Request sous forme
        d'arguments nommés. Elle est vide pour les nouvelles Requests, ce qui
        signifie que, par défaut, les callbacks reçoivent seulement un objet
        :class:`~scrapy.http.Response` comme argument.

        Cette dict fait l'objet d'une :doc:`copie superficielle
        <library/copy>` lorsque la requête est clonée avec les méthodes
        ``copy()`` ou ``replace()``. Elle est aussi accessible, dans votre
        spider, via l'attribut ``response.cb_kwargs``.

        En cas d'échec du traitement de la requête, cette dict est accessible
        via ``failure.request.cb_kwargs`` dans l'errback de la requête. Pour
        plus d'informations, voir :ref:`errback-cb_kwargs`.

        .. note:: Lorsque :setting:`JOBDIR` est défini, les requêtes sont
            sérialisées sur le disque avec :mod:`pickle` (voir
            :ref:`request-serialization`). Par conséquent, le callback reçoit
            une copie profonde (deep copy) de tout objet stocké dans
            ``cb_kwargs`` : modifier un tel objet dans le callback n'affecte
            donc pas l'original. Dans ce cas, évitez de vous appuyer sur un
            état mutable partagé transmis via ``cb_kwargs``.

    .. attribute:: Request.meta
       :value: {}

        Un dictionnaire de métadonnées arbitraires pour la requête.

        Vous pouvez étendre les métadonnées de la requête comme bon vous
        semble.

        Les métadonnées de la requête sont aussi accessibles via l'attribut
        :attr:`~scrapy.http.Response.meta` d'une réponse.

        Pour passer vos propres données d'un callback de spider à un autre,
        utilisez plutôt :attr:`cb_kwargs`, voir :ref:`callback-data`. Toutefois,
        les métadonnées de requête peuvent être le bon choix dans certains
        scénarios, par exemple pour conserver des données de débogage à travers
        toutes les requêtes suivantes (p. ex. l'URL d'origine). Pour copier
        automatiquement certaines clés de métadonnées dans les requêtes
        suivantes, pensez à utiliser le paramètre :setting:`STICKY_META_KEYS`.

        Un usage courant des métadonnées de requête est de définir des
        paramètres propres à une requête pour les composants de Scrapy
        (extensions, middlewares, etc.). Par exemple, si vous définissez
        :reqmeta:`dont_retry` à ``True``,
        :class:`~scrapy.downloadermiddlewares.retry.RetryMiddleware` ne
        réessaiera jamais cette requête, même si elle échoue. Voir
        :ref:`topics-request-meta`.

        Vous pouvez aussi utiliser les métadonnées de requête dans vos propres
        composants Scrapy, par exemple pour conserver des informations d'état
        propres à votre composant. Ainsi,
        :class:`~scrapy.downloadermiddlewares.retry.RetryMiddleware` utilise la
        clé de métadonnées :reqmeta:`retry_times` pour suivre le nombre de fois
        qu'une requête a déjà été réessayée.

        Copier toutes les métadonnées d'une requête précédente dans une
        nouvelle requête de suivi, dans un callback de spider, est une mauvaise
        pratique : les métadonnées d'une requête peuvent inclure des
        métadonnées définies par des composants Scrapy qui ne sont pas faites
        pour être copiées dans d'autres requêtes. Par exemple, copier la clé de
        métadonnées :reqmeta:`retry_times` dans les requêtes suivantes peut
        réduire le nombre de tentatives autorisées pour ces requêtes.

        Vous ne devriez copier toutes les métadonnées d'une requête vers une
        autre que si la nouvelle requête est destinée à remplacer l'ancienne,
        comme c'est souvent le cas lorsqu'on renvoie une requête depuis une
        méthode d'un :ref:`middleware de downloader
        <topics-downloader-middleware>`.

        Notez aussi que les méthodes de requête :meth:`copy` et :meth:`replace`
        font une :doc:`copie superficielle <library/copy>` des métadonnées de
        la requête.

        .. seealso:: :class:`~scrapy.spidermiddlewares.metacopy.MetaCopyDetectionMiddleware`
            pour un middleware intégré qui signale ce problème à l'exécution.

    .. autoattribute:: dont_filter

    .. autoattribute:: Request.attributes

    .. method:: Request.copy()

       Renvoie une nouvelle Request qui est une copie de cette Request. Voir
       aussi : :ref:`callback-data`.

    .. method:: Request.replace([url, method, headers, body, cookies, meta, flags, encoding, priority, dont_filter, callback, errback, cb_kwargs, cls])

       Renvoie un objet Request avec les mêmes membres, sauf ceux pour lesquels
       de nouvelles valeurs sont données via les arguments nommés spécifiés.
       Les attributs :attr:`~scrapy.Request.cb_kwargs` et
       :attr:`~scrapy.Request.meta` font l'objet d'une copie superficielle par
       défaut (sauf si de nouvelles valeurs sont données en arguments). Voir
       aussi :ref:`callback-data`.

    .. automethod:: from_curl

    .. automethod:: to_curl

    .. automethod:: to_dict


.. _form:

Créer des requêtes qui soumettent des formulaires HTML
------------------------------------------------------

Utilisez :doc:`form2request <form2request:index>` pour construire les données
d'une requête à partir d'un élément HTML ``<form>`` et les convertir en
:class:`~scrapy.Request`.

Installez-le avec pip :

.. code-block:: bash

    pip install form2request

Sélectionnez le formulaire voulu avec CSS ou XPath, puis construisez et
convertissez les données de la requête :

.. code-block:: python

    from form2request import form2request


    def parse(self, response):
        form = response.css("form#search")
        request_data = form2request(form, data={"q": "scrapy"})
        yield request_data.to_scrapy(callback=self.parse_results)

Utilisez ``data`` pour remplacer la valeur de certains champs. Pour retirer un
champ de la requête obtenue, donnez-lui la valeur ``None``.

Par défaut, form2request simule un clic sur le premier bouton de soumission.
Pour soumettre sans cliquer sur aucun bouton, passez ``click=False``. Pour
cliquer sur un bouton de soumission précis, passez son élément :

.. code-block:: python

    def parse(self, response):
        form = response.css("form#checkout")
        submit = form.css('button[name="pay"]')
        request_data = form2request(form, click=submit)

.. _topics-request-response-ref-request-userlogin:

Utiliser form2request pour simuler une connexion d'utilisateur
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Il est courant que les sites web fournissent des champs de formulaire
pré-remplis via des éléments ``<input type="hidden">``, comme des données liées
à la session ou des jetons d'authentification (pour les pages de connexion).
Construisez la requête à partir du formulaire et ne remplacez que les
identifiants :

.. code-block:: python

    import scrapy
    from form2request import form2request


    class LoginSpider(scrapy.Spider):
        name = "example.com"
        start_urls = ["http://www.example.com/users/login.php"]

        def parse(self, response):
            form = response.css("form")
            request_data = form2request(
                form,
                data={"username": "john", "password": "secret"},
            )
            yield request_data.to_scrapy(callback=self.after_login)

        def after_login(self, response): ...


Autres fonctions liées aux requêtes
-----------------------------------

.. autofunction:: scrapy.http.request.NO_CALLBACK

.. autofunction:: scrapy.utils.request.request_from_dict

.. autofunction:: scrapy.utils.httpobj.urlparse_cached


.. _request-fingerprints:

Empreintes de requêtes (request fingerprints)
---------------------------------------------

Certains aspects du scraping, comme le filtrage des requêtes en double (voir
:setting:`DUPEFILTER_CLASS`) ou la mise en cache des réponses (voir
:setting:`HTTPCACHE_POLICY`), demandent de pouvoir générer un identifiant court
et unique à partir d'un objet :class:`~scrapy.Request` : une empreinte de
requête (request fingerprint).

Vous n'avez souvent pas besoin de vous soucier des empreintes de requêtes :
l'outil d'empreinte par défaut fonctionne pour la plupart des projets.

Cependant, il n'existe pas de méthode universelle pour générer un identifiant
unique à partir d'une requête, car différentes situations demandent de comparer
les requêtes de différentes façons. Par exemple, vous pouvez parfois avoir
besoin de comparer les URL sans tenir compte de la casse, d'inclure les
fragments d'URL, d'exclure certains paramètres de la chaîne de requête de
l'URL, d'inclure tout ou partie des en-têtes, etc.

Pour changer la façon dont les empreintes sont construites pour vos requêtes,
utilisez le paramètre :setting:`REQUEST_FINGERPRINTER_CLASS`.

.. setting:: REQUEST_FINGERPRINTER_CLASS

REQUEST_FINGERPRINTER_CLASS
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Valeur par défaut : :class:`scrapy.utils.request.RequestFingerprinter`

Une :ref:`classe d'outil d'empreinte de requêtes
<custom-request-fingerprinter>` ou son chemin d'import.

.. autoclass:: scrapy.utils.request.RequestFingerprinter

.. _custom-request-fingerprinter:

Écrire votre propre outil d'empreinte de requêtes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Un outil d'empreinte de requêtes (request fingerprinter) est un
:ref:`composant <topics-components>` qui doit implémenter la méthode suivante :

.. currentmodule:: None

.. method:: fingerprint(self, request: scrapy.Request)

   Renvoie un objet :class:`bytes` qui identifie de façon unique *request*.

   Voir aussi :ref:`request-fingerprint-restrictions`.

.. currentmodule:: scrapy.http

La méthode :meth:`fingerprint` de l'outil d'empreinte par défaut,
:class:`scrapy.utils.request.RequestFingerprinter`, utilise
:func:`scrapy.utils.request.fingerprint` avec ses paramètres par défaut. Pour
certains cas d'usage courants, vous pouvez aussi utiliser
:func:`scrapy.utils.request.fingerprint` dans l'implémentation de votre
méthode :meth:`fingerprint` :

.. autofunction:: scrapy.utils.request.fingerprint

Par défaut, le calcul de l'empreinte canonicalise l'URL de la requête. Si
:reqmeta:`verbatim_url` vaut ``True``, le calcul de l'empreinte ne canonicalise
pas l'URL, et le paramètre ``keep_fragments`` est ignoré (il est de fait
vrai).

Par exemple, pour prendre en compte la valeur d'un en-tête de requête nommé
``X-ID`` :

.. code-block:: python

    # my_project/settings.py
    REQUEST_FINGERPRINTER_CLASS = "my_project.utils.RequestFingerprinter"

    # my_project/utils.py
    from scrapy.utils.request import fingerprint


    class RequestFingerprinter:
        def fingerprint(self, request):
            return fingerprint(request, include_headers=["X-ID"])

Pour dédoublonner les paramètres répétés de la chaîne de requête, comme ceux
que certains sites ajoutent à chaque redirection et qui peuvent autrement
provoquer des boucles de redirection, construisez vous-même l'URL dédoublonnée
et déléguez le reste à :func:`scrapy.utils.request.fingerprint` :

.. code-block:: python

    # my_project/settings.py
    REQUEST_FINGERPRINTER_CLASS = "my_project.utils.RequestFingerprinter"

    # my_project/utils.py
    from urllib.parse import parse_qsl, urlencode, urlsplit, urlunsplit
    from weakref import WeakKeyDictionary

    from scrapy.utils.request import fingerprint


    class RequestFingerprinter:
        cache = WeakKeyDictionary()

        def fingerprint(self, request):
            if request not in self.cache:
                parts = urlsplit(request.url)
                query = urlencode(list(set(parse_qsl(parts.query))))
                deduped_url = urlunsplit(parts._replace(query=query))
                deduped_request = request.replace(url=deduped_url)
                self.cache[request] = fingerprint(deduped_request)
            return self.cache[request]

Vous pouvez aussi écrire votre propre logique de calcul d'empreinte à partir de
zéro.

Cependant, si vous n'utilisez pas :func:`scrapy.utils.request.fingerprint`,
veillez à utiliser un :class:`~weakref.WeakKeyDictionary` pour mettre en cache
les empreintes de requêtes :

-   La mise en cache économise du CPU en garantissant que les empreintes ne
    sont calculées qu'une seule fois par requête, et non une fois par composant
    Scrapy ayant besoin de l'empreinte d'une requête.

-   Utiliser un :class:`~weakref.WeakKeyDictionary` économise de la mémoire en
    garantissant que les objets requête ne restent pas en mémoire pour
    toujours simplement parce que vous en gardez des références dans votre
    dictionnaire de cache.

Par exemple, pour ne prendre en compte que l'URL d'une requête, sans
canonicalisation préalable de l'URL et sans tenir compte de la méthode ni du
corps de la requête :

.. code-block:: python

    from hashlib import sha1
    from weakref import WeakKeyDictionary

    from scrapy.utils.python import to_bytes


    class RequestFingerprinter:
        cache = WeakKeyDictionary()

        def fingerprint(self, request):
            if request not in self.cache:
                fp = sha1()
                fp.update(to_bytes(request.url))
                self.cache[request] = fp.digest()
            return self.cache[request]

Si vous avez besoin de pouvoir remplacer le calcul d'empreinte pour des
requêtes arbitraires depuis les callbacks de votre spider, vous pouvez
implémenter un outil d'empreinte qui lit les empreintes dans
:attr:`request.meta <scrapy.Request.meta>` lorsqu'elles sont disponibles, puis
se rabat sur :func:`scrapy.utils.request.fingerprint`. Par exemple :

.. code-block:: python

    from scrapy.utils.request import fingerprint


    class RequestFingerprinter:
        def fingerprint(self, request):
            if "fingerprint" in request.meta:
                return request.meta["fingerprint"]
            return fingerprint(request)

Si vous avez besoin de reproduire le même algorithme de calcul d'empreinte que
Scrapy 2.6, utilisez l'outil d'empreinte suivant :

.. code-block:: python

    from hashlib import sha1
    from weakref import WeakKeyDictionary

    from scrapy.utils.python import to_bytes
    from w3lib.url import canonicalize_url


    class RequestFingerprinter:
        cache = WeakKeyDictionary()

        def fingerprint(self, request):
            if request not in self.cache:
                fp = sha1()
                fp.update(to_bytes(request.method))
                fp.update(to_bytes(canonicalize_url(request.url)))
                fp.update(request.body or b"")
                self.cache[request] = fp.digest()
            return self.cache[request]


.. _request-fingerprint-restrictions:

Restrictions sur les empreintes de requêtes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Les composants Scrapy qui utilisent les empreintes de requêtes peuvent imposer
des restrictions supplémentaires sur le format des empreintes que génère votre
:ref:`outil d'empreinte de requêtes <custom-request-fingerprinter>`.

Les composants Scrapy intégrés suivants ont de telles restrictions :

-   :class:`scrapy.extensions.httpcache.FilesystemCacheStorage` (valeur par
    défaut de :setting:`HTTPCACHE_STORAGE`)

    Les empreintes de requêtes doivent faire au moins 1 octet.

    Les limites de longueur des chemins et des noms de fichiers du système de
    fichiers de :setting:`HTTPCACHE_DIR` s'appliquent aussi. À l'intérieur de
    :setting:`HTTPCACHE_DIR`, la structure de répertoires suivante est créée :

    -   :attr:`.Spider.name`

        -   le premier octet d'une empreinte de requête, en hexadécimal

            -   l'empreinte en hexadécimal

                -   des noms de fichiers de 16 caractères maximum

    Par exemple, si une empreinte de requête est composée de 20 octets (valeur
    par défaut), que :setting:`HTTPCACHE_DIR` vaut
    ``'/home/user/project/.scrapy/httpcache'`` et que le nom de votre spider
    est ``'my_spider'``, votre système de fichiers doit prendre en charge un
    chemin de fichier comme celui-ci ::

        /home/user/project/.scrapy/httpcache/my_spider/01/0123456789abcdef0123456789abcdef01234567/response_headers

-   :class:`scrapy.extensions.httpcache.DbmCacheStorage`

    L'implémentation DBM sous-jacente doit prendre en charge des clés dont la
    longueur vaut deux fois le nombre d'octets d'une empreinte de requête, plus
    5. Par exemple, si une empreinte de requête est composée de 20 octets
    (valeur par défaut), des clés de 45 caractères doivent être prises en
    charge.


.. _callbacks:

Callbacks
=========

Un callback est une fonction que Scrapy appelle avec la :class:`Response` d'une
:class:`~scrapy.Request` une fois cette requête téléchargée, afin que vous
puissiez extraire des données de cette réponse et générer des requêtes
supplémentaires pour poursuivre le crawl :

.. code-block:: python

    from scrapy import Request, Spider


    class BookSpider(Spider):
        name = "books"

        async def start(self):
            yield Request("https://books.toscrape.com/", callback=self.parse_home)

        def parse_home(self, response):
            for url in response.css("h3 a::attr(href)").getall():
                yield Request(response.urljoin(url), callback=self.parse_book)

        def parse_book(self, response):
            yield {"title": response.css("h1::text").get()}

Les requêtes peuvent aussi définir un :ref:`errback <errbacks>`, que Scrapy
appelle à la place du callback lorsqu'une exception est levée pendant le
traitement de la requête ou de sa réponse, par exemple une erreur de connexion
ou, par défaut, une réponse hors de la plage 2xx.


.. _callback-assignment:

Assigner un callback à une requête
----------------------------------

Pour assigner un callback à une requête, utilisez le paramètre ``callback`` de
:class:`~scrapy.Request`, qui définit l'attribut :attr:`.Request.callback` :

.. code-block:: python

    from scrapy import Request


    def parse_home(response): ...


    request = Request("https://books.toscrape.com/", callback=parse_home)

Les requêtes sans callback, c'est-à-dire dont :attr:`~scrapy.Request.callback`
vaut ``None``, sont traitées par la méthode :meth:`~scrapy.Spider.parse` du
spider :

.. code-block:: python

    request = Request("https://books.toscrape.com/")  # Handled by parse()

Si une requête n'est jamais destinée à atteindre un callback de spider, par
exemple parce qu'un :ref:`composant <topics-components>` l'envoie et traite
lui-même sa réponse, assignez-lui plutôt la valeur spéciale
:func:`~scrapy.http.request.NO_CALLBACK`, afin que les :ref:`middlewares de
downloader <topics-downloader-middleware>` puissent distinguer ces requêtes.

Alors que :attr:`~scrapy.Request.callback` n'accepte que des callables, certaines
classes de spiders vous permettent aussi de définir un callback par son nom :
:attr:`CrawlSpider.rules <scrapy.spiders.CrawlSpider.rules>` et
:attr:`SitemapSpider.sitemap_rules <scrapy.spiders.SitemapSpider.sitemap_rules>`
acceptent toutes deux le nom d'une méthode du spider sous forme de chaîne.


.. _writing-callbacks:

Écrire un callback
------------------

N'importe quel callable peut être un callback, à condition de prendre la
réponse comme premier paramètre positionnel, et toute :ref:`donnée
supplémentaire de callback <callback-data>` comme paramètres nommés. Les
méthodes de spider sont le choix le plus courant, mais les fonctions simples,
les expressions lambda et les autres objets callables fonctionnent aussi.

.. note:: Si vous activez la :ref:`persistance des jobs <topics-jobs>` via le
    paramètre :setting:`JOBDIR`, les callbacks doivent être des méthodes du
    spider en cours d'exécution. Les requêtes avec tout autre callback ne
    peuvent pas être sérialisées : elles ne sont donc conservées qu'en mémoire
    et sont perdues lorsque vous mettez le crawl en pause. Voir
    :ref:`request-serialization`.

Un callback peut être :

-   Une fonction ordinaire :

    .. code-block:: python

        def parse(self, response):
            return {"url": response.url}

-   Une fonction génératrice :

    .. code-block:: python

        def parse(self, response):
            yield {"url": response.url}

-   Une fonction coroutine, c'est-à-dire définie avec ``async def`` :

    .. code-block:: python

        async def parse(self, response):
            return {"url": response.url}

-   Une fonction génératrice asynchrone :

    .. code-block:: python

        async def parse(self, response):
            yield {"url": response.url}

Les deux dernières permettent d'utiliser ``await``, ``async for`` et
``async with`` dans votre callback. Voir :ref:`topics-coroutines`.


.. _callback-output:

Sortie d'un callback
--------------------

Un callback peut renvoyer (return) ou produire (yield) l'une des valeurs
suivantes :

-   ``None``, ce qui ne fait rien.

    Les callbacks qui ne produisent aucune sortie, par exemple ceux qui se
    contentent de journaliser des informations sur la réponse, sont parfaitement
    valides. Les valeurs ``None`` au sein d'un itérable de sortie de callback
    sont également ignorées.

-   Un objet :class:`~scrapy.Request`, que Scrapy planifie, télécharge, puis
    envoie finalement à son propre callback.

-   Un :ref:`objet item <topics-items>`, que Scrapy envoie aux :ref:`item
    pipelines <topics-item-pipeline>`.

    Tout objet qui n'est ni ``None`` ni un objet :class:`~scrapy.Request` est
    traité comme un item.

-   Un itérable de n'importe quelles valeurs ci-dessus, par exemple une liste
    ou, plus couramment, un générateur.

    Les :term:`itérables asynchrones <asynchronous iterable>`, par exemple un
    :term:`générateur asynchrone <asynchronous generator>`, sont également pris
    en charge.

.. note:: Lorsqu'un callback *renvoie* (return) un objet, Scrapy itère sur cet
    objet s'il prend en charge l'itération, sauf pour les objets :class:`dict`,
    :class:`~scrapy.Item`, :class:`str` et :class:`bytes`, qui sont toujours
    traités comme des items uniques.

.. note:: Dans un callback générateur, une instruction ``return`` avec une
    valeur ne produit aucune sortie, car une telle valeur ne fait pas partie de
    ce que le générateur produit (yield). Scrapy journalise un avertissement
    lorsqu'il détecte un tel callback, voir
    :setting:`WARN_ON_GENERATOR_RETURN_VALUE`.

Avant que Scrapy n'agisse sur la sortie d'un callback, cette sortie passe par
la méthode
:meth:`~scrapy.spidermiddlewares.SpiderMiddleware.process_spider_output` de vos
:ref:`middlewares de spider <topics-spider-middleware>`, qui peuvent la modifier
ou en supprimer une partie.

Si un callback lève une exception, l':attr:`~scrapy.Request.errback` de la
requête n'est *pas* appelé. L'exception passe à la place par la méthode
:meth:`~scrapy.spidermiddlewares.SpiderMiddleware.process_spider_exception` de
vos middlewares de spider et, à moins que l'un d'eux ne la traite, Scrapy la
journalise et envoie le signal :signal:`spider_error`.


.. _callback-data:
.. _topics-request-response-ref-request-callback-arguments:

Passer des données supplémentaires aux fonctions callback
---------------------------------------------------------

Dans certains cas, vous pouvez vouloir passer des données à un callback en plus
de la réponse, par exemple des données extraites de la réponse qui a déclenché
la requête. L'exemple suivant montre comment y parvenir avec l'attribut
:attr:`.Request.cb_kwargs` :

.. code-block:: python

    from scrapy import Request


    def parse(self, response):
        request = Request(
            "http://www.example.com/index.html",
            callback=self.parse_page2,
            cb_kwargs=dict(main_url=response.url),
        )
        request.cb_kwargs["foo"] = "bar"  # add more arguments for the callback
        yield request


    def parse_page2(self, response, main_url, foo):
        yield dict(
            main_url=main_url,
            other_url=response.url,
            foo=foo,
        )

:attr:`.Request.cb_kwargs` est la méthode recommandée pour passer vos propres
données à un callback. N'utilisez :attr:`.Request.meta` que pour des données
destinées aux :ref:`composants <topics-components>`, comme les middlewares et
les extensions.

.. _errbacks:
.. _topics-request-response-ref-errbacks:

Errbacks
========

L'errback d'une requête est une fonction qui sera appelée lorsqu'une exception
est levée pendant son traitement.

Il reçoit un :exc:`~twisted.python.failure.Failure` comme premier paramètre et
peut servir à suivre les délais d'expiration lors de l'établissement d'une
connexion, les erreurs DNS, etc.

Scrapy définit l'attribut ``request`` de cet objet
:exc:`~twisted.python.failure.Failure` avec l'objet :class:`~scrapy.Request`
en cours de traitement.

Si un errback lève une exception, Scrapy la journalise et envoie le signal
:signal:`spider_error`, sauf si l'exception est celle que l'errback a reçue :
Scrapy la journalise alors comme une erreur de téléchargement.

Voici un exemple de spider qui journalise toutes les erreurs et intercepte
certaines erreurs spécifiques si besoin :

.. code-block:: python

    from scrapy import Request, Spider
    from scrapy.spidermiddlewares.httperror import HttpError
    from twisted.internet.error import DNSLookupError
    from twisted.internet.error import TimeoutError, TCPTimedOutError


    class ErrbackSpider(Spider):
        name = "errback_example"
        start_urls = [
            "http://www.httpbin.org/",  # HTTP 200 expected
            "http://www.httpbin.org/status/404",  # Not found error
            "http://www.httpbin.org/status/500",  # server issue
            "http://www.httpbin.org:12345/",  # non-responding host, timeout expected
            "https://example.invalid/",  # DNS error expected
        ]

        async def start(self):
            for u in self.start_urls:
                yield Request(
                    u,
                    callback=self.parse_httpbin,
                    errback=self.errback_httpbin,
                    dont_filter=True,
                )

        def parse_httpbin(self, response):
            self.logger.info(f"Got successful response from {response.url}")
            # do something useful here...

        def errback_httpbin(self, failure):
            # log all failures
            self.logger.error(repr(failure))

            # in case you want to do something special for some errors,
            # you may need the failure's type:

            if failure.check(HttpError):
                # these exceptions come from HttpError spider middleware
                # you can get the non-200 response
                response = failure.value.response
                self.logger.error("HttpError on %s", response.url)

            elif failure.check(DNSLookupError):
                # this is the original request
                request = failure.request
                self.logger.error("DNSLookupError on %s", request.url)

            elif failure.check(TimeoutError, TCPTimedOutError):
                request = failure.request
                self.logger.error("TimeoutError on %s", request.url)


.. _errback-cb_kwargs:

Accéder aux données supplémentaires dans les fonctions errback
--------------------------------------------------------------

En cas d'échec du traitement de la requête, vous pouvez vouloir accéder aux
arguments des fonctions callback afin de poursuivre le traitement dans l'errback
en fonction de ces arguments. L'exemple suivant montre comment y parvenir avec
``Failure.request.cb_kwargs`` :

.. code-block:: python

    from scrapy import Request


    def parse(self, response):
        request = Request(
            "http://www.example.com/index.html",
            callback=self.parse_page2,
            errback=self.errback_page2,
            cb_kwargs=dict(main_url=response.url),
        )
        yield request


    def parse_page2(self, response, main_url):
        pass


    def errback_page2(self, failure):
        yield dict(
            main_url=failure.request.cb_kwargs["main_url"],
        )


.. _topics-request-meta:

Clés spéciales de Request.meta
==============================

L'attribut :attr:`.Request.meta` peut contenir n'importe quelle donnée
arbitraire, mais certaines clés spéciales sont reconnues par Scrapy et ses
extensions intégrées.

Les voici :

* :reqmeta:`allow_offsite`
* :reqmeta:`autothrottle_dont_adjust_delay`
* :reqmeta:`bindaddress`
* :reqmeta:`cache_timestamp`
* :reqmeta:`cookiejar`
* :reqmeta:`depth`
* :reqmeta:`dont_cache`
* :reqmeta:`dont_merge_cookies`
* :reqmeta:`dont_obey_robotstxt`
* :reqmeta:`dont_redirect`
* :reqmeta:`dont_retry`
* :reqmeta:`download_fail_on_dataloss`
* :reqmeta:`download_latency`
* :reqmeta:`download_maxsize`
* :reqmeta:`download_slot`
* :reqmeta:`download_timeout`
* :reqmeta:`download_warnsize`
* :reqmeta:`ftp_local_filename`
* :reqmeta:`ftp_passive`
* :reqmeta:`ftp_password`
* :reqmeta:`ftp_user`
* :reqmeta:`give_up_log_level`
* :reqmeta:`handle_httpstatus_all`
* :reqmeta:`handle_httpstatus_list`
* :reqmeta:`http_auth_domain`
* :reqmeta:`http_pass`
* :reqmeta:`http_user`
* :reqmeta:`is_start_request`
* :reqmeta:`link_text`
* :reqmeta:`max_retry_times`
* :reqmeta:`priority_adjust`
* :reqmeta:`proxy`
* :reqmeta:`redirect_reasons`
* :reqmeta:`redirect_times`
* :reqmeta:`redirect_ttl`
* :reqmeta:`redirect_urls`
* :reqmeta:`referrer_policy`
* :reqmeta:`retry_times`
* :reqmeta:`rule`
* :reqmeta:`verbatim_url`

Les composants Scrapy utilisent aussi des clés de meta dont le nom commence par
un tiret bas (underscore), comme ``_auth_proxy``. Celles-ci sont internes et
peuvent changer ou disparaître à tout moment.

.. reqmeta:: bindaddress

bindaddress
-----------

L'adresse sortante locale par défaut pour les connexions des download handlers.

Cette valeur de meta peut être soit :

- une adresse d'hôte sous forme de chaîne (p. ex. ``"127.0.0.2"``), auquel cas
  le port local est choisi automatiquement, soit

- un tuple ``(host, port)`` (p. ex. ``("127.0.0.2", 50000)``) pour se lier à la
  fois à une interface locale précise et à un port local précis.

Par exemple :

.. code-block:: python

    Request(
        "https://example.org",
        meta={"bindaddress": "127.0.0.2"},
    )

.. code-block:: python

    Request(
        "https://example.org",
        meta={"bindaddress": ("127.0.0.2", 50000)},
    )

Si elle n'est pas définie, les download handlers HTTP intégrés utilisent la
valeur de :setting:`DOWNLOAD_BIND_ADDRESS` comme adresse de liaison par défaut.
Définissez la clé de meta de requête :reqmeta:`bindaddress` pour la remplacer
pour une requête précise.

Cette clé de meta n'est pas prise en charge par
:class:`~scrapy.core.downloader.handlers._httpx.HttpxDownloadHandler` ni par
:class:`~scrapy.core.downloader.handlers._aiohttp.AiohttpDownloadHandler`, mais
ils prennent bien en charge le paramètre :setting:`DOWNLOAD_BIND_ADDRESS`.

.. reqmeta:: download_timeout

download_timeout
----------------

Le temps (en secondes) que le downloader attend avant d'abandonner pour cause
de délai dépassé. Voir aussi : :setting:`DOWNLOAD_TIMEOUT`.

.. reqmeta:: download_latency

download_latency
----------------

Le temps écoulé pour récupérer la réponse depuis le début de la requête,
c'est-à-dire depuis l'envoi du message HTTP sur le réseau. Il couvre le temps
jusqu'à ce que Scrapy lise la réponse, que votre propre code peut retarder,
voir :ref:`optimize-blocking`. Cette clé de meta n'est disponible qu'une fois
la réponse téléchargée. Alors que la plupart des autres clés de meta servent à
contrôler le comportement de Scrapy, celle-ci est censée être en lecture seule.

.. reqmeta:: download_fail_on_dataloss

download_fail_on_dataloss
-------------------------

Indique s'il faut ou non échouer sur les réponses tronquées ou corrompues. Voir :
:setting:`DOWNLOAD_FAIL_ON_DATALOSS`.

.. reqmeta:: ftp_local_filename

ftp_local_filename
------------------

Chemin, sous forme de :class:`bytes`, du fichier dans lequel écrire le corps de
la réponse d'une requête ``ftp://``. S'il est défini, :attr:`Response.body
<scrapy.http.Response.body>` contient ce chemin au lieu du contenu du fichier.

.. reqmeta:: give_up_log_level

give_up_log_level
-----------------

.. versionadded:: 2.17.0

:ref:`Niveau de journalisation <levels>` utilisé pour le message journalisé
lorsqu'une requête dépasse son nombre de tentatives. Voir
:setting:`RETRY_GIVE_UP_LOG_LEVEL` pour plus de détails.

.. reqmeta:: http_auth_domain

http_auth_domain
----------------

.. versionadded:: 2.17.0

Remplace :setting:`HTTPAUTH_DOMAIN` pour cette requête.

.. reqmeta:: http_pass

http_pass
---------

.. versionadded:: 2.17.0

Remplace :setting:`HTTPAUTH_PASS` pour cette requête.

.. reqmeta:: http_user

http_user
---------

.. versionadded:: 2.17.0

Remplace :setting:`HTTPAUTH_USER` pour cette requête.

.. reqmeta:: max_retry_times

max_retry_times
---------------

Cette clé de meta sert à définir le nombre de tentatives par requête. Lorsqu'elle
est définie, la clé de meta :reqmeta:`max_retry_times` a priorité sur le
paramètre :setting:`RETRY_TIMES`.

.. reqmeta:: verbatim_url

verbatim_url
------------

.. versionadded:: 2.17.0

Définissez cette clé à ``True`` pour conserver l'URL de la requête telle
qu'elle a été passée à :class:`~scrapy.Request`, sans percent-encoding de
l'URL.

Lorsque cette clé est activée, :func:`~scrapy.utils.request.fingerprint` ne
canonicalise pas l'URL de la requête : des requêtes dont les URL ne diffèrent
que par des caractères qui seraient autrement canonicalisés obtiennent donc des
empreintes différentes.

Dans ce mode, le paramètre ``keep_fragments`` est ignoré (il est de fait
vrai).

.. _topics-stop-response-download:

Arrêter le téléchargement d'une réponse
=======================================

Lever une exception :exc:`~scrapy.exceptions.StopDownload` depuis un handler des
signaux :class:`~scrapy.signals.bytes_received` ou
:class:`~scrapy.signals.headers_received` arrête le téléchargement d'une
réponse donnée. Voir l'exemple suivant :

.. code-block:: python

    import scrapy


    class StopSpider(scrapy.Spider):
        name = "stop"
        start_urls = ["https://docs.scrapy.org/en/latest/"]

        @classmethod
        def from_crawler(cls, crawler):
            spider = super().from_crawler(crawler)
            crawler.signals.connect(
                spider.on_bytes_received, signal=scrapy.signals.bytes_received
            )
            return spider

        def parse(self, response):
            # 'last_chars' show that the full response was not downloaded
            yield {"len": len(response.text), "last_chars": response.text[-40:]}

        def on_bytes_received(self, data, request, spider):
            raise scrapy.exceptions.StopDownload(fail=False)

ce qui produit la sortie suivante ::

    2020-05-19 17:26:12 [scrapy.core.engine] INFO: Spider opened
    2020-05-19 17:26:12 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
    2020-05-19 17:26:13 [scrapy.core.downloader.handlers.http11] DEBUG: Download stopped for <GET https://docs.scrapy.org/en/latest/> from signal handler StopSpider.on_bytes_received
    2020-05-19 17:26:13 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://docs.scrapy.org/en/latest/> (referer: None) ['download_stopped']
    2020-05-19 17:26:13 [scrapy.core.scraper] DEBUG: Scraped from <200 https://docs.scrapy.org/en/latest/>
    {'len': 279, 'last_chars': 'dth, initial-scale=1.0">\n  \n  <title>Scr'}
    2020-05-19 17:26:13 [scrapy.core.engine] INFO: Closing spider (finished)

Par défaut, les réponses obtenues sont traitées par leurs errbacks
correspondants. Pour appeler leur callback à la place, comme dans cet exemple,
passez ``fail=False`` à l'exception :exc:`~scrapy.exceptions.StopDownload`.


.. _topics-request-response-ref-request-subclasses:

Sous-classes de Request
=======================

Voici la liste des sous-classes de :class:`~scrapy.Request` intégrées. Vous
pouvez aussi en créer une sous-classe pour implémenter vos propres
fonctionnalités.

FormRequest
-----------

.. autoclass:: scrapy.FormRequest

JsonRequest
-----------

La classe JsonRequest étend la classe de base :class:`~scrapy.Request` avec des
fonctionnalités pour gérer les requêtes JSON.

.. class:: JsonRequest(url, [... data, dumps_kwargs])

   La classe :class:`JsonRequest` ajoute deux nouveaux paramètres nommés à la
   méthode ``__init__()``. Les autres arguments sont les mêmes que pour la
   classe :class:`~scrapy.Request` et ne sont pas documentés ici.

   Utiliser :class:`JsonRequest` définit l'en-tête ``Content-Type`` à
   ``application/json`` et l'en-tête ``Accept`` à
   ``application/json, text/javascript, */*; q=0.01``

   :param data: tout objet sérialisable en JSON qui doit être encodé en JSON et
      assigné au corps de la requête. Si l'argument :attr:`~scrapy.Request.body`
      est fourni, ce paramètre est ignoré. Si l'argument
      :attr:`~scrapy.Request.body` n'est pas fourni et que l'argument ``data``
      l'est, :attr:`~scrapy.Request.method` est automatiquement défini à
      ``'POST'``.
   :type data: object

   :param dumps_kwargs: Paramètres qui seront passés à la méthode
       :func:`json.dumps` sous-jacente, utilisée pour sérialiser les données au
       format JSON.
   :type dumps_kwargs: dict

   .. autoattribute:: JsonRequest.attributes

Exemple d'utilisation de JsonRequest
------------------------------------

Envoyer une requête POST JSON avec une charge utile (payload) JSON :

.. skip: next
.. code-block:: python

   data = {
       "name1": "value1",
       "name2": "value2",
   }
   yield JsonRequest(url="http://www.example.com/post/action", data=data)


Objets Response
===============

.. autoclass:: Response

    :param url: l'URL de cette réponse
    :type url: str

    :param status: le statut HTTP de la réponse. Vaut ``200`` par défaut.
    :type status: int

    :param headers: les en-têtes de cette réponse. Les valeurs de la dict
       peuvent être des chaînes (pour les en-têtes à valeur unique) ou des
       listes (pour les en-têtes à valeurs multiples).
    :type headers: dict

    :param body: le corps de la réponse. Pour accéder au texte décodé sous forme
       de chaîne, utilisez ``response.text`` depuis une
       :ref:`sous-classe de Response <topics-request-response-ref-response-subclasses>`
       qui gère l'encodage, comme :class:`TextResponse`.
    :type body: bytes

    :param flags: une liste contenant les valeurs initiales de l'attribut
       :attr:`Response.flags`. Si elle est fournie, la liste fait l'objet d'une
       copie superficielle.
    :type flags: list

    :param request: la valeur initiale de l'attribut :attr:`Response.request`.
        Il représente la :class:`~scrapy.Request` qui a généré cette réponse.
    :type request: scrapy.Request

    :param certificate: un objet représentant le certificat SSL du serveur.
    :type certificate: typing.Any

    :param ip_address: L'adresse IP du serveur d'où provient la Response.
    :type ip_address: :class:`ipaddress.IPv4Address` or :class:`ipaddress.IPv6Address`

    :param protocol: Le protocole qui a été utilisé pour télécharger la réponse.
        Par exemple : "HTTP/1.0", "HTTP/1.1", "h2"
    :type protocol: :class:`str`

    .. attribute:: Response.url

        Une chaîne de caractères contenant l'URL de la réponse.

        Cet attribut est en lecture seule. Pour changer l'URL d'une Response,
        utilisez :meth:`replace`.

    .. attribute:: Response.status

        Un entier représentant le statut HTTP de la réponse. Exemples : ``200``,
        ``404``.

    .. attribute:: Response.headers

        Un objet de type dictionnaire (:class:`scrapy.http.headers.Headers`) qui
        contient les en-têtes de la réponse. Les valeurs sont accessibles avec
        :meth:`~scrapy.http.headers.Headers.get`, qui renvoie la dernière valeur
        d'en-tête portant le nom indiqué, ou avec
        :meth:`~scrapy.http.headers.Headers.getlist`, qui renvoie toutes les
        valeurs d'en-tête portant le nom indiqué. Par exemple, cet appel vous
        donnera tous les cookies des en-têtes :

        .. skip: next

        .. code-block:: python

            response.headers.getlist("Set-Cookie")

    .. attribute:: Response.body

        Le corps de la réponse, sous forme de bytes.

        Si vous voulez le corps sous forme de chaîne, utilisez
        :attr:`TextResponse.text` (disponible uniquement dans
        :class:`TextResponse` et ses sous-classes).

        Cet attribut est en lecture seule. Pour changer le corps d'une Response,
        utilisez :meth:`replace`.

    .. attribute:: Response.request

        L'objet :class:`~scrapy.Request` qui a généré cette réponse. Cet
        attribut est assigné dans l'engine de Scrapy, après que la réponse et la
        requête sont passées par tous les :ref:`middlewares de downloader
        <topics-downloader-middleware>`. Cela signifie en particulier que :

        - Les redirections HTTP créent une nouvelle requête à partir de la
          requête d'avant la redirection. Elle possède la majorité des mêmes
          métadonnées et attributs de la requête d'origine, et c'est elle qui
          est assignée à la réponse redirigée, au lieu de la propagation de la
          requête d'origine.

        - Response.request.url n'est pas toujours égal à Response.url

        - Cet attribut n'est disponible que dans le code du spider et dans les
          :ref:`middlewares de spider <topics-spider-middleware>`, mais pas dans
          les middlewares de downloader (bien que la Request y soit disponible
          par d'autres moyens) ni dans les handlers du signal
          :signal:`response_downloaded`.

    .. attribute:: Response.meta

        Un raccourci vers l'attribut :attr:`~scrapy.Request.meta` de l'objet
        :attr:`Response.request` (c'est-à-dire ``self.request.meta``).

        Contrairement à l'attribut :attr:`Response.request`, l'attribut
        :attr:`Response.meta` est propagé à travers les redirections et les
        nouvelles tentatives : vous obtenez donc le :attr:`.Request.meta`
        d'origine envoyé depuis votre spider.

        .. seealso:: attribut :attr:`.Request.meta`

    .. attribute:: Response.cb_kwargs

        Un raccourci vers l'attribut :attr:`~scrapy.Request.cb_kwargs` de l'objet
        :attr:`Response.request` (c'est-à-dire ``self.request.cb_kwargs``).

        Contrairement à l'attribut :attr:`Response.request`, l'attribut
        :attr:`Response.cb_kwargs` est propagé à travers les redirections et les
        nouvelles tentatives : vous obtenez donc le :attr:`.Request.cb_kwargs`
        d'origine envoyé depuis votre spider.

        .. seealso:: attribut :attr:`.Request.cb_kwargs`

    .. attribute:: Response.flags

        Une liste qui contient les drapeaux (flags) de cette réponse. Les flags
        sont des étiquettes utilisées pour marquer les Responses. Par exemple :
        ``'cached'``, ``'redirected'``', etc. Ils sont affichés dans la
        représentation en chaîne de la Response (méthode ``__str__()``), que
        l'engine utilise pour la journalisation.

    .. attribute:: Response.certificate

        Un objet représentant le certificat SSL du serveur. Son type et son
        contenu dépendent du download handler qui a produit la réponse.

        Renseigné uniquement pour les réponses ``https`` ; vaut ``None``
        sinon.

    .. attribute:: Response.ip_address

        L'adresse IP du serveur d'où provient la Response.

        Cet attribut n'est actuellement renseigné que par les download handlers
        HTTP, c'est-à-dire pour les réponses ``http(s)``. Pour les autres
        handlers, :attr:`ip_address` vaut toujours ``None``.

    .. attribute:: Response.protocol

        Le protocole qui a été utilisé pour télécharger la réponse.
        Par exemple : "HTTP/1.0", "HTTP/1.1"

        Cet attribut n'est actuellement renseigné que par les download handlers
        HTTP, c'est-à-dire pour les réponses ``http(s)``. Pour les autres
        handlers, :attr:`protocol` vaut toujours ``None``.

    .. autoattribute:: Response.attributes

    .. method:: Response.copy()

       Renvoie une nouvelle Response qui est une copie de cette Response.

    .. method:: Response.replace([url, status, headers, body, request, flags, certificate, ip_address, protocol, cls])

       Renvoie un objet Response avec les mêmes membres, sauf ceux pour lesquels
       de nouvelles valeurs sont données via les arguments nommés spécifiés.
       L'attribut :attr:`Response.meta` est copié par défaut.

    .. method:: Response.urljoin(url)

        Construit une URL absolue en combinant l':attr:`url` de la Response avec
        une éventuelle URL relative.

        C'est une enveloppe (wrapper) autour de :func:`~urllib.parse.urljoin` :
        c'est simplement un alias pour faire cet appel :

        .. skip: next

        .. code-block:: python

            urllib.parse.urljoin(response.url, url)

    .. automethod:: Response.follow

    .. automethod:: Response.follow_all

    .. automethod:: Response.to_dict

    .. automethod:: Response.from_dict


.. _topics-request-response-ref-response-subclasses:

Sous-classes de Response
========================

Voici la liste des sous-classes de Response intégrées disponibles. Vous pouvez
aussi créer une sous-classe de la classe Response pour implémenter vos propres
fonctionnalités.

Objets TextResponse
-------------------

.. class:: TextResponse(url, [encoding[, ...]])

    Les objets :class:`TextResponse` ajoutent des capacités d'encodage à la
    classe de base :class:`Response`, qui est conçue pour n'être utilisée que
    pour des données binaires, comme des images, des sons ou n'importe quel
    fichier multimédia.

    Les objets :class:`TextResponse` prennent en charge un nouvel argument de la
    méthode ``__init__()``, en plus de ceux des objets :class:`Response` de
    base. Le reste des fonctionnalités est le même que pour la classe
    :class:`Response` et n'est pas documenté ici.

    :param encoding: une chaîne qui contient l'encodage à utiliser pour cette
       réponse. Si vous créez un objet :class:`TextResponse` avec une chaîne
       comme corps, elle sera convertie en bytes avec cet encodage. Si
       *encoding* vaut ``None`` (valeur par défaut), l'encodage est recherché
       à la place dans les en-têtes et dans le corps de la réponse.
    :type encoding: str

    Les objets :class:`TextResponse` prennent en charge les attributs suivants,
    en plus de ceux des objets :class:`Response` standard :

    .. attribute:: TextResponse.text

       Le corps de la réponse, sous forme de chaîne.

       C'est la même chose que ``response.body.decode(response.encoding)``,
       mais le résultat est mis en cache après le premier appel : vous pouvez
       donc accéder plusieurs fois à ``response.text`` sans surcoût
       supplémentaire.

       .. note::

            ``str(response.body)`` n'est pas une façon correcte de convertir le
            corps de la réponse en chaîne :

            .. code-block:: pycon

                >>> str(b"body")
                "b'body'"


    .. attribute:: TextResponse.encoding

       Une chaîne avec l'encodage de cette réponse. L'encodage est déterminé en
       essayant les mécanismes suivants, dans l'ordre :

       1. l'encodage passé dans l'argument ``encoding`` de la méthode
          ``__init__()``

       2. l'encodage de la `marque d'ordre des octets (byte order mark)`_ au
          début du corps de la réponse

       3. l'encodage déclaré dans l'en-tête HTTP Content-Type. Si cet encodage
          n'est pas valide (c'est-à-dire inconnu), il est ignoré et le
          mécanisme de résolution suivant est essayé.

       4. l'encodage déclaré dans le corps de la réponse. La classe
          TextResponse ne fournit aucune fonctionnalité particulière pour cela.
          En revanche, les classes :class:`HtmlResponse` et
          :class:`XmlResponse` en fournissent.

       5. l'encodage déduit en examinant le corps de la réponse. C'est la
          méthode la plus fragile, mais aussi la dernière essayée.

       Cet ordre correspond à l'`algorithme de détection d'encodage (encoding
       sniffing algorithm)`_ du standard HTML, que suivent les navigateurs web.

       Pour déterminer l'encodage autrement, déterminez-le vous-même et passez-le
       via :meth:`Response.replace` depuis un :ref:`middleware de downloader
       <topics-downloader-middleware>`. Donnez à ce middleware un ordre compris
       entre ceux de
       :class:`~scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware`
       (580) et de
       :class:`~scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware`
       (590), afin qu'il reçoive un corps décompressé et qu'aucun autre composant
       ne lise le texte de la réponse avant lui.

       Par exemple, pour donner la priorité à une déclaration présente dans le
       corps de la réponse par rapport à l'en-tête Content-Type :

       .. code-block:: python

           from w3lib.encoding import html_body_declared_encoding, read_bom

           from scrapy.http import TextResponse


           class BodyEncodingMiddleware:
               def process_response(self, request, response, spider):
                   if not isinstance(response, TextResponse):
                       return response
                   if read_bom(response.body)[0]:
                       return response
                   encoding = html_body_declared_encoding(response.body)
                   return response.replace(encoding=encoding) if encoding else response

    .. attribute:: TextResponse.selector

        Une instance de :class:`~scrapy.Selector` qui utilise la réponse comme
        cible. Le sélecteur est instancié de façon paresseuse (lazy) au premier
        accès.

    .. autoattribute:: TextResponse.attributes

    Les objets :class:`TextResponse` prennent en charge les méthodes suivantes,
    en plus de celles des objets :class:`Response` standard :

    .. method:: TextResponse.jmespath(query)

        .. skip: start

        Un raccourci vers ``TextResponse.selector.jmespath(query)`` :

        .. code-block:: python

            response.jmespath("object.[*]")

    .. method:: TextResponse.xpath(query)

        Un raccourci vers ``TextResponse.selector.xpath(query)`` :

        .. code-block:: python

            response.xpath("//p")

    .. method:: TextResponse.css(query)

        Un raccourci vers ``TextResponse.selector.css(query)`` :

        .. code-block:: python

            response.css("p")

        .. skip: end

    .. automethod:: TextResponse.follow

    .. automethod:: TextResponse.follow_all

    .. automethod:: TextResponse.json()

    .. method:: TextResponse.urljoin(url)

        Construit une URL absolue en combinant l'URL de base de la Response avec
        une éventuelle URL relative. L'URL de base est extraite de la balise
        ``<base>``, ou simplement de :attr:`Response.url` s'il n'y a pas de
        telle balise.

.. _marque d'ordre des octets (byte order mark): https://en.wikipedia.org/wiki/Byte_order_mark
.. _algorithme de détection d'encodage (encoding sniffing algorithm): https://html.spec.whatwg.org/multipage/parsing.html#determining-the-character-encoding


Objets HtmlResponse
-------------------

.. class:: HtmlResponse(url[, ...])

    La classe :class:`HtmlResponse` est une sous-classe de
    :class:`TextResponse` qui ajoute la détection automatique de l'encodage en
    examinant l'attribut HTML `meta http-equiv`_. Voir
    :attr:`TextResponse.encoding`.

.. _meta http-equiv: https://www.w3schools.com/TAGS/att_meta_http_equiv.asp

Objets XmlResponse
------------------

.. class:: XmlResponse(url[, ...])

    La classe :class:`XmlResponse` est une sous-classe de :class:`TextResponse`
    qui ajoute la détection automatique de l'encodage en examinant la ligne de
    déclaration XML. Voir :attr:`TextResponse.encoding`.

.. _bug in lxml: https://bugs.launchpad.net/lxml/+bug/1665241

Objets JsonResponse
-------------------

.. class:: JsonResponse(url[, ...])

    La classe :class:`JsonResponse` est une sous-classe de
    :class:`TextResponse` utilisée lorsque la réponse a un `type MIME JSON
    <https://mimesniff.spec.whatwg.org/#json-mime-type>`_ dans son en-tête
    `Content-Type`.


Autres fonctions liées aux réponses
===================================

.. autofunction:: scrapy.utils.response.response_from_dict
