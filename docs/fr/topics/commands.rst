.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/commands.rst`.

.. highlight:: none

.. _topics-commands:

==========================
Outil en ligne de commande
==========================

Scrapy se pilote avec l'outil en ligne de commande ``scrapy``, que nous
appellerons ici « l'outil Scrapy » pour le distinguer de ses sous-commandes, que
nous appelons simplement « commandes » ou « commandes Scrapy ».

L'outil Scrapy propose plusieurs commandes pour des usages variés, et chacune
accepte un ensemble différent d'arguments et d'options.

(La commande ``scrapy deploy`` a été supprimée dans la version 1.0 au profit de
l'outil autonome ``scrapyd-deploy``. Voir `Déploiement de votre projet <Deploying your project_>`_.)

.. _topics-config-settings:

Paramètres de configuration
===========================

Scrapy recherche des paramètres de configuration dans des fichiers
``scrapy.cfg`` au format ini, situés à des emplacements standards :

1. ``/etc/scrapy.cfg`` ou ``c:\scrapy\scrapy.cfg`` (pour tout le système),
2. ``~/.config/scrapy.cfg`` (``$XDG_CONFIG_HOME``) et ``~/.scrapy.cfg`` (``$HOME``)
   pour les paramètres globaux (propres à l'utilisateur), et
3. ``scrapy.cfg`` à la racine d'un projet Scrapy (voir la section suivante).

Les paramètres de ces fichiers sont fusionnés selon l'ordre de priorité
indiqué : les valeurs définies par l'utilisateur ont une priorité plus haute
que les valeurs par défaut du système, et les paramètres du projet
l'emportent sur tous les autres, lorsqu'ils sont définis.

Scrapy comprend aussi un certain nombre de variables d'environnement, par
lesquelles il peut être configuré. Actuellement, ce sont :

* ``SCRAPY_SETTINGS_MODULE`` (voir :ref:`topics-settings-module-envvar`)
* ``SCRAPY_PROJECT`` (voir :ref:`topics-project-envvar`)
* ``SCRAPY_PYTHON_SHELL`` (voir :ref:`topics-shell`)

.. _topics-project-structure:

Structure par défaut des projets Scrapy
=======================================

Avant de plonger dans l'outil en ligne de commande et ses sous-commandes,
commençons par comprendre la structure des répertoires d'un projet Scrapy.

Même si elle peut être modifiée, tous les projets Scrapy ont par défaut la même
structure de fichiers, semblable à celle-ci::

   scrapy.cfg
   myproject/
       __init__.py
       items.py
       middlewares.py
       pipelines.py
       settings.py
       spiders/
           __init__.py
           spider1.py
           spider2.py
           ...

Le répertoire où se trouve le fichier ``scrapy.cfg`` est appelé *répertoire
racine du projet*. Ce fichier contient le nom du module Python qui définit les
paramètres du projet. En voici un exemple :

.. code-block:: ini

    [settings]
    default = myproject.settings

Par défaut, le nom du projet apparaît aussi à d'autres endroits.
:setting:`SPIDER_MODULES` et :setting:`NEWSPIDER_MODULE` font référence au
module du projet lui-même : ils doivent donc correspondre à son emplacement
réel. :setting:`BOT_NAME` vaut par défaut le même nom, mais ce n'est qu'un
identifiant. Quant au préfixe (le nom du projet en majuscule initiale) des noms
de classes dans :file:`middlewares.py` et :file:`pipelines.py`, ce n'est qu'une
convention de nommage : ni l'un ni l'autre ne doit obligatoirement correspondre
au nom du module.

.. _find-projects:

Trouver des projets
-------------------

Pour trouver les projets Scrapy dans une arborescence de répertoires, par
exemple depuis une extension d'éditeur, utilisez
:func:`~scrapy.utils.project.find_projects` :

.. autofunction:: scrapy.utils.project.find_projects

.. _topics-project-envvar:

Partager le répertoire racine entre plusieurs projets
=====================================================

Un répertoire racine de projet, celui qui contient le fichier ``scrapy.cfg``,
peut être partagé par plusieurs projets Scrapy, chacun ayant son propre module
de paramètres.

Dans ce cas, vous devez définir un ou plusieurs alias pour ces modules de
paramètres, dans la section ``[settings]`` de votre fichier ``scrapy.cfg`` :

.. code-block:: ini

    [settings]
    default = myproject1.settings
    project1 = myproject1.settings
    project2 = myproject2.settings

Par défaut, l'outil en ligne de commande ``scrapy`` utilise les paramètres
``default``. Utilisez la variable d'environnement ``SCRAPY_PROJECT`` pour
indiquer à ``scrapy`` un autre projet à utiliser::

    $ scrapy settings --get BOT_NAME
    Project 1 Bot
    $ export SCRAPY_PROJECT=project2
    $ scrapy settings --get BOT_NAME
    Project 2 Bot


Utiliser l'outil ``scrapy``
===========================

Vous pouvez commencer par lancer l'outil Scrapy sans aucun argument : il
affichera une aide d'utilisation et la liste des commandes disponibles::

    Scrapy X.Y - no active project

    Usage:
      scrapy <command> [options] [args]

    Available commands:
      fetch         Fetch a URL using the Scrapy downloader
      runspider     Run a spider from a Python file, no project required
    [...]

La première ligne affiche le projet actuellement actif si vous vous trouvez à
l'intérieur d'un projet Scrapy. Dans cet exemple, la commande a été lancée
depuis l'extérieur d'un projet. Lancée depuis l'intérieur d'un projet, elle
aurait affiché quelque chose comme ceci::

    Scrapy X.Y - project: myproject

    Usage:
      scrapy <command> [options] [args]

    [...]

Créer des projets
-----------------

La première chose que l'on fait généralement avec l'outil ``scrapy`` est de
créer son projet Scrapy::

    scrapy startproject myproject [project_dir]

Cela crée un projet Scrapy dans le répertoire ``project_dir``. Si
``project_dir`` n'est pas indiqué, il vaut ``myproject`` par défaut.

Ensuite, vous entrez dans le nouveau répertoire du projet::

    cd project_dir

Et vous êtes prêt à utiliser la commande ``scrapy`` pour gérer et contrôler
votre projet à partir de là.

Contrôler des projets
---------------------

Vous utilisez l'outil ``scrapy`` depuis l'intérieur de vos projets pour les
contrôler et les gérer.

Par exemple, pour créer un nouveau spider::

    scrapy genspider mydomain mydomain.com

Certaines commandes Scrapy (comme :command:`crawl`) doivent être lancées depuis
l'intérieur d'un projet Scrapy. Consultez la :ref:`référence des commandes
<topics-commands-ref>` ci-dessous pour savoir quelles commandes doivent être
lancées à l'intérieur d'un projet et lesquelles n'en ont pas besoin.

Gardez aussi à l'esprit que certaines commandes peuvent se comporter un peu
différemment lorsqu'elles sont lancées depuis l'intérieur d'un projet. Par
exemple, la commande fetch utilisera les comportements redéfinis par un spider
(comme l'attribut ``custom_settings``, qui permet de remplacer des paramètres)
si l'URL récupérée est associée à un spider particulier. C'est voulu, car la
commande ``fetch`` sert à vérifier comment les spiders téléchargent les pages.

.. _topics-commands-ref:

Commandes disponibles de l'outil
================================

Cette section contient la liste des commandes intégrées disponibles, avec une
description et quelques exemples d'utilisation. N'oubliez pas que vous pouvez
toujours obtenir plus d'informations sur chaque commande en lançant::

    scrapy <command> -h

Et vous pouvez voir toutes les commandes disponibles avec::

    scrapy -h

Il existe deux types de commandes : celles qui ne fonctionnent que depuis
l'intérieur d'un projet Scrapy (les commandes propres à un projet) et celles
qui fonctionnent aussi sans projet Scrapy actif (les commandes globales),
même si elles peuvent se comporter un peu différemment lorsqu'elles sont lancées
depuis l'intérieur d'un projet (car elles utilisent alors les paramètres
redéfinis par le projet).

Commandes globales :

* :command:`startproject`
* :command:`genspider`
* :command:`settings`
* :command:`runspider`
* :command:`shell`
* :command:`fetch`
* :command:`view`
* :command:`version`
* :command:`bench`

Commandes réservées aux projets :

* :command:`crawl`
* :command:`check`
* :command:`list`
* :command:`edit`
* :command:`parse`

.. command:: startproject

startproject
------------

* Syntaxe : ``scrapy startproject <project_name> [project_dir]``
* Nécessite un projet : *non*

Crée un nouveau projet Scrapy nommé ``project_name``, dans le répertoire
``project_dir``. Si ``project_dir`` n'est pas indiqué, il vaut ``project_name``
par défaut.

Exemple d'utilisation::

    $ scrapy startproject myproject

.. command:: genspider

genspider
---------

* Syntaxe : ``scrapy genspider [-t template] <name> <domain or URL>``
* Nécessite un projet : *non*

Crée un nouveau spider dans le dossier courant ou, si la commande est appelée
depuis l'intérieur d'un projet, dans le dossier ``spiders`` du projet courant.
Le paramètre ``<name>`` devient le ``name`` du spider, tandis que
``<domain or URL>`` sert à générer les attributs ``allowed_domains`` et
``start_urls`` du spider.

Exemple d'utilisation::

    $ scrapy genspider -l
    Available templates:
      basic
      crawl
      csvfeed
      xmlfeed

    $ scrapy genspider example example.com
    Created spider 'example' using template 'basic'

    $ scrapy genspider -t crawl scrapyorg scrapy.org
    Created spider 'scrapyorg' using template 'crawl'

Cette commande n'est qu'un raccourci pratique pour créer des spiders à partir
de modèles prédéfinis, mais ce n'est certainement pas la seule façon d'en
créer : vous pouvez aussi écrire vous-même les fichiers de code source du
spider au lieu d'utiliser cette commande.

.. _spider-templates:

Modèles de spider personnalisés
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pour définir vos propres modèles de spider, faites pointer
:setting:`TEMPLATES_DIR` vers un répertoire contenant un sous-répertoire
:file:`spiders`, et écrivez-y un fichier :file:`{name}.tmpl` pour chaque
modèle, où *name* est la valeur à passer à ``-t``. Vos modèles remplacent ceux
qui sont intégrés, lesquels se trouvent dans le répertoire :file:`templates` du
paquet ``scrapy`` : copiez donc ceux que vous souhaitez conserver.

Vous pouvez aussi passer à ``-t`` le chemin d'un fichier :file:`.tmpl` au lieu
d'un nom, pour l'utiliser sans toucher à :setting:`TEMPLATES_DIR`.

Les modèles sont rendus avec :class:`string.Template` : ``$variable`` et
``${variable}`` sont remplacés, et ``$$`` produit un seul ``$``, dont les
expressions régulières ont souvent besoin. Le rendu échoue pour toute variable
autre que les suivantes :

-   ``name`` : le nom du spider, tel que passé à la commande.

-   ``module`` : *name* sous forme de nom de module valide, également utilisé
    comme nom de fichier du spider généré.

-   ``classname`` : *module* en notation camel case, avec le suffixe
    ``Spider``.

-   ``url`` : l'URL passée à la commande, à laquelle le schéma ``https`` est
    ajouté si elle n'en avait pas.

-   ``domain`` : le domaine de *url*.

-   ``project_name`` : :setting:`BOT_NAME`.

-   ``ProjectName`` : *project_name* en notation camel case.

.. command:: crawl

crawl
-----

* Syntaxe : ``scrapy crawl <spider>``
* Nécessite un projet : *oui*

Lance un crawl avec le spider dont le :attr:`~scrapy.Spider.name` est donné, et
qui doit être l'un de ceux que :command:`list` affiche. Pour lancer un spider
depuis un fichier, utilisez plutôt :command:`runspider`.

Options prises en charge :

* ``-h, --help`` : affiche un message d'aide et quitte

* ``-a NAME=VALUE`` : définit un argument du spider (peut être répété)

* ``--output FILE`` ou ``-o FILE`` : ajoute les items extraits à la fin de FILE
  (utilisez ``-`` pour la sortie standard). Pour définir le format de sortie,
  ajoutez deux-points à la fin de l'URI de sortie (par exemple,
  ``-o FILE:FORMAT``)

* ``--overwrite-output FILE`` ou ``-O FILE`` : écrit les items extraits dans
  FILE en écrasant tout fichier existant. Pour définir le format de sortie,
  ajoutez deux-points à la fin de l'URI de sortie (par exemple,
  ``-O FILE:FORMAT``)

Exemples d'utilisation::

    $ scrapy crawl myspider
    [ ... myspider starts crawling ... ]

    $ scrapy crawl -o myfile:csv myspider
    [ ... myspider starts crawling and appends the result to the file myfile in CSV format ... ]

    $ scrapy crawl -O myfile:json myspider
    [ ... myspider starts crawling and saves the result in myfile in JSON format, overwriting the original content ... ]

.. command:: check

check
-----

* Syntaxe : ``scrapy check [-l] <spider>``
* Nécessite un projet : *oui*

Lance les vérifications des contrats (contract checks).

.. versionadded:: 2.19.0
   L'option ``-a``, pour passer des arguments au spider, comme dans :command:`crawl`.

.. skip: start

Exemples d'utilisation::

    $ scrapy check -l
    first_spider
      * parse
      * parse_item
    second_spider
      * parse
      * parse_item

    $ scrapy check
    F.F.
    ======================================================================
    FAIL: [first_spider] parse (@returns post-hook)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      ...
    scrapy.exceptions.ContractFail: Returned 92 requests, expected 0..4

    ======================================================================
    FAIL: [first_spider] parse_item (@scrapes post-hook)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      ...
    scrapy.exceptions.ContractFail: Missing fields: RetailPricex

    ----------------------------------------------------------------------
    Ran 4 contracts in 0.174s

    FAILED (failures=2)

.. skip: end

.. command:: list

list
----

* Syntaxe : ``scrapy list``
* Nécessite un projet : *oui*

Liste tous les spiders disponibles dans le projet courant. La sortie contient
un spider par ligne.

Exemple d'utilisation::

    $ scrapy list
    spider1
    spider2

.. command:: edit

edit
----

* Syntaxe : ``scrapy edit <spider>``
* Nécessite un projet : *oui*

Modifie le spider indiqué avec l'éditeur défini dans la variable d'environnement
``EDITOR`` ou (si elle n'est pas définie) dans le paramètre :setting:`EDITOR`.

Cette commande n'est fournie que comme un raccourci pratique pour le cas le plus
courant ; le développeur est bien sûr libre de choisir n'importe quel outil ou
IDE pour écrire et déboguer ses spiders.

Exemple d'utilisation::

    $ scrapy edit spider1

.. command:: fetch

fetch
-----

* Syntaxe : ``scrapy fetch <url>``
* Nécessite un projet : *non*

Télécharge l'URL donnée avec le downloader de Scrapy et écrit le contenu sur la
sortie standard.

L'intérêt de cette commande est qu'elle récupère la page comme le ferait le
spider. Par exemple, si le spider possède un attribut ``USER_AGENT`` qui
remplace le User Agent, c'est celui-là qui sera utilisé.

Cette commande permet donc de « voir » comment votre spider récupérerait une
certaine page.

Si elle est utilisée en dehors d'un projet, aucun comportement propre à un
spider n'est appliqué, et ce sont simplement les paramètres par défaut du
downloader de Scrapy qui sont utilisés.

Options prises en charge :

* ``--spider=SPIDER`` : ignore la détection automatique du spider et force
  l'utilisation d'un spider précis

* ``--headers`` : affiche les en-têtes HTTP de la requête et de la réponse au
  lieu du corps de la réponse

* ``--no-redirect`` : ne suit pas les redirections HTTP 3xx (par défaut, elles
  sont suivies)

Exemples d'utilisation::

    $ scrapy fetch --nolog http://www.example.com/some/page.html
    [ ... html content here ... ]

    $ scrapy fetch --nolog --headers http://www.example.com/
    > Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    > Accept-Language: en
    > User-Agent: Scrapy/2.16.0 (+https://scrapy.org)
    > Accept-Encoding: gzip, deflate, br, zstd
    >
    < Date: Wed, 08 Jul 2026 06:15:01 GMT
    < Content-Type: text/html
    < Server: cloudflare
    < Last-Modified: Wed, 01 Jul 2026 17:50:18 GMT
    < Allow: GET, HEAD
    < Cf-Cache-Status: HIT
    < Age: 8184
    < Cf-Ray: a17cf3b80eddf141-DME

.. command:: view

view
----

* Syntaxe : ``scrapy view <url>``
* Nécessite un projet : *non*

Ouvre l'URL donnée dans un navigateur, telle que votre spider Scrapy la
« verrait ». Les spiders voient parfois les pages différemment des utilisateurs
ordinaires : cette commande permet donc de vérifier ce que le spider « voit » et
de confirmer que c'est bien ce que vous attendez.

Options prises en charge :

* ``--spider=SPIDER`` : ignore la détection automatique du spider et force
  l'utilisation d'un spider précis

* ``--no-redirect`` : ne suit pas les redirections HTTP 3xx (par défaut, elles
  sont suivies)

Exemple d'utilisation::

    $ scrapy view http://www.example.com/some/page.html
    [ ... browser starts ... ]

.. command:: shell

shell
-----

* Syntaxe : ``scrapy shell [url]``
* Nécessite un projet : *non*

Démarre le shell Scrapy pour l'URL donnée (si elle est fournie), ou le laisse
vide si aucune URL n'est donnée. Il accepte aussi les chemins de fichiers
locaux de style UNIX, relatifs avec les préfixes ``./`` ou ``../``, ou absolus.
Voir :ref:`topics-shell` pour plus d'informations.

Options prises en charge :

* ``--spider=SPIDER`` : ignore la détection automatique du spider et force
  l'utilisation d'un spider précis

* ``-c code`` : évalue le code dans le shell, affiche le résultat et quitte

* ``--no-redirect`` : ne suit pas les redirections HTTP 3xx (par défaut, elles
  sont suivies) ; cela n'affecte que l'URL que vous pouvez passer en argument
  sur la ligne de commande ; une fois dans le shell, ``fetch(url)`` continue de
  suivre les redirections HTTP par défaut.

Exemple d'utilisation::

    $ scrapy shell http://www.example.com/some/page.html
    [ ... scrapy shell starts ... ]

    $ scrapy shell --nolog http://www.example.com/ -c '(response.status, response.url)'
    (200, 'http://www.example.com/')

    # shell follows HTTP redirects by default
    $ scrapy shell --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
    (200, 'http://example.com/')

    # you can disable this with --no-redirect
    # (only for the URL passed as command line argument)
    $ scrapy shell --no-redirect --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
    (302, 'http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F')


.. command:: parse

parse
-----

* Syntaxe : ``scrapy parse <url> [options]``
* Nécessite un projet : *oui*

Récupère l'URL donnée et l'analyse avec le spider qui la gère, en utilisant la
méthode passée avec l'option ``--callback``, ou ``parse`` si elle n'est pas
indiquée.

Options prises en charge :

* ``--spider=SPIDER`` : ignore la détection automatique du spider et force
  l'utilisation d'un spider précis

* ``-a NAME=VALUE`` : définit un argument du spider (peut être répété)

* ``--callback`` ou ``-c`` : méthode du spider à utiliser comme callback pour
  analyser la réponse

* ``--meta`` ou ``-m`` : meta de requête supplémentaire qui sera passée à la
  requête du callback. Ce doit être une chaîne JSON valide. Exemple :
  ``--meta='{"foo": "bar"}'``

* ``--cbkwargs`` : arguments nommés supplémentaires qui seront passés au
  callback. Ce doit être une chaîne JSON valide. Exemple : ``--cbkwargs='{"foo":
  "bar"}'``

* ``--pipelines`` : :ref:`fait traverser les pipelines aux items
  <test-item-pipeline>`

* ``--rules`` ou ``-r`` : utilise les règles de :class:`~scrapy.spiders.CrawlSpider`
  pour découvrir le callback (c'est-à-dire la méthode du spider) à utiliser pour
  analyser la réponse

* ``--noitems`` : n'affiche pas les items extraits

* ``--nolinks`` : n'affiche pas les liens extraits

* ``--nocolour`` : évite d'utiliser pygments pour colorer la sortie

* ``--depth`` ou ``-d`` : niveau de profondeur jusqu'auquel les requêtes doivent
  être suivies récursivement (par défaut : 1)

* ``--verbose`` ou ``-v`` : affiche des informations pour chaque niveau de
  profondeur

* ``--output`` ou ``-o`` : écrit les items extraits dans un fichier

.. skip: start

Exemple d'utilisation::

    $ scrapy parse http://www.example.com/ -c parse_item
    [ ... scrapy log lines crawling example.com spider ... ]

    >>> STATUS DEPTH LEVEL 1 <<<
    # Scraped Items  ------------------------------------------------------------
    [{'name': 'Example item',
     'category': 'Furniture',
     'length': '12 cm'}]

    # Requests  -----------------------------------------------------------------
    []

.. skip: end


.. command:: settings

settings
--------

* Syntaxe : ``scrapy settings [options]``
* Nécessite un projet : *non*

Obtient la valeur d'un paramètre de Scrapy.

Si elle est utilisée dans un projet, la commande affiche la valeur du paramètre
définie par le projet ; sinon, elle affiche la valeur par défaut de Scrapy pour
ce paramètre.

Exemple d'utilisation::

    $ scrapy settings --get BOT_NAME
    scrapybot
    $ scrapy settings --get DOWNLOAD_DELAY
    0

.. command:: runspider

runspider
---------

* Syntaxe : ``scrapy runspider <spider_file.py>``
* Nécessite un projet : *non*

Lance le spider défini dans le fichier Python donné, sans avoir besoin d'un
projet.

Options prises en charge : les mêmes que pour :command:`crawl`.

Exemple d'utilisation::

    $ scrapy runspider myspider.py
    [ ... spider starts crawling ... ]

.. command:: version

version
-------

* Syntaxe : ``scrapy version [-v]``
* Nécessite un projet : *non*

Affiche la version de Scrapy. Avec l'option ``-v``, elle affiche aussi les
informations sur Python, Twisted et la plateforme, ce qui est utile pour les
rapports de bogue.

.. command:: bench

bench
-----

* Syntaxe : ``scrapy bench``
* Nécessite un projet : *non*

Lance un test de performance rapide (benchmark). :ref:`benchmarking`.

.. _topics-commands-crawlerprocess:

Commandes qui lancent un crawl
==============================

De nombreuses commandes ont besoin de lancer un crawl d'un genre ou d'un autre,
en exécutant soit un spider fourni par l'utilisateur, soit un spider interne
spécial :

* :command:`bench`
* :command:`check`
* :command:`crawl`
* :command:`fetch`
* :command:`parse`
* :command:`runspider`
* :command:`shell`
* :command:`view`

Elles utilisent pour cela une instance interne de
:class:`scrapy.crawler.AsyncCrawlerProcess` ou de
:class:`scrapy.crawler.CrawlerProcess`. Dans la plupart des cas, ce détail ne
devrait pas avoir d'importance pour l'utilisateur qui lance la commande, mais il
peut devenir important lorsque l'utilisateur :ref:`a besoin d'un reactor Twisted
non standard <disable-asyncio>`.

Scrapy choisit laquelle de ces deux classes utiliser d'après la valeur des
paramètres :setting:`TWISTED_REACTOR` et :setting:`TWISTED_REACTOR_ENABLED`.
Si :setting:`TWISTED_REACTOR_ENABLED` vaut ``False``, il utilise
:class:`~scrapy.crawler.AsyncCrawlerProcess`. Sinon, si la valeur de
:setting:`TWISTED_REACTOR` est la valeur par défaut
(``'twisted.internet.asyncioreactor.AsyncioSelectorReactor'``),
:class:`~scrapy.crawler.AsyncCrawlerProcess` est utilisée ; dans les autres cas,
c'est :class:`~scrapy.crawler.CrawlerProcess` qui est utilisée. Les
:ref:`paramètres du spider <spider-settings>` ne sont pas pris en compte pour ce
choix, car ils sont chargés après que la décision a été prise. Cela peut
provoquer une erreur si le paramètre au niveau du projet est défini sur
:ref:`le reactor asyncio <install-asyncio>` (:ref:`explicitement
<project-settings>` ou :ref:`en utilisant la valeur par défaut de Scrapy
<default-settings>`) et que :ref:`le paramètre du spider exécuté
<spider-settings>` est défini sur :ref:`un reactor différent <disable-asyncio>`,
parce que :class:`~scrapy.crawler.AsyncCrawlerProcess` ne prend en charge que le
reactor asyncio. Dans ce cas, vous devez définir le paramètre
:setting:`FORCE_CRAWLER_PROCESS` à ``True`` (au niveau du projet ou via la ligne
de commande) afin que Scrapy utilise :class:`~scrapy.crawler.CrawlerProcess`,
qui prend en charge tous les reactors.

.. _custom-commands:

Commandes de projet personnalisées
==================================

Vous pouvez aussi ajouter vos propres commandes de projet grâce au paramètre
:setting:`COMMANDS_MODULE`. Cela vous permet de créer des commandes propres à
votre projet, qui sont automatiquement découvertes et rendues disponibles par
l'outil en ligne de commande ``scrapy``.

Créer des commandes personnalisées
----------------------------------

Pour créer une commande personnalisée, héritez de la classe
:class:`~scrapy.commands.ScrapyCommand` et implémentez les méthodes requises.
Cela vous permet d'étendre l'interface en ligne de commande de Scrapy avec vos
propres fonctionnalités, comme des utilitaires propres au projet, des outils de
traitement de données ou des aides au déploiement.

Lorsque vous créez une commande personnalisée, vous définissez son comportement
en fixant des attributs de classe et en redéfinissant des méthodes précises.
Voici ce qu'il faut savoir :

**Attributs que vous pouvez définir :**

* :attr:`~scrapy.commands.ScrapyCommand.requires_project` (bool) : si ``True``,
  la commande ne s'exécute qu'à l'intérieur d'un projet Scrapy (par défaut :
  ``False``).
* :attr:`~scrapy.commands.ScrapyCommand.requires_crawler_process` (bool) : si
  ``True``, une instance de :class:`~scrapy.crawler.AsyncCrawlerProcess` ou de
  :class:`~scrapy.crawler.CrawlerProcess` sera créée par Scrapy lors de
  l'exécution de la commande et rendue disponible dans l'attribut
  :attr:`~scrapy.commands.ScrapyCommand.crawler_process` (par défaut :
  ``True``).
* :attr:`~scrapy.commands.ScrapyCommand.default_settings` (dict) : paramètres
  qui remplaceront les paramètres par défaut lors de l'exécution de cette
  commande (par défaut : ``{}``).
* :attr:`~scrapy.commands.ScrapyCommand.exitcode` (int) : code de sortie du
  processus à définir lorsque la commande se termine (par défaut : ``0``).

**Méthodes que vous devez redéfinir :**

* :meth:`~scrapy.commands.ScrapyCommand.short_desc` : renvoie une courte
  description de la commande.
* :meth:`~scrapy.commands.ScrapyCommand.run` : point d'entrée principal pour
  l'exécution de la commande.

**Méthodes que vous pouvez redéfinir :**

* :meth:`~scrapy.commands.ScrapyCommand.syntax` : renvoie la syntaxe de la
  commande (de préférence sur une seule ligne, sans le nom de la commande).
* :meth:`~scrapy.commands.ScrapyCommand.long_desc` : renvoie une description
  détaillée de la commande.
* :meth:`~scrapy.commands.ScrapyCommand.add_options` : ajoute des options
  propres à la commande à l'analyseur d'arguments.
* :meth:`~scrapy.commands.ScrapyCommand.process_options` : traite les options
  de ligne de commande analysées et définit les paramètres avant que
  :attr:`~scrapy.commands.ScrapyCommand.crawler_process` soit instancié.

**Exemple de commande personnalisée :**

.. code-block:: python

    from scrapy.commands import ScrapyCommand
    import argparse


    class MyCustomCommand(ScrapyCommand):
        requires_project = True

        def syntax(self):
            return "[options] <spider_name>"

        def short_desc(self):
            return "Run my custom command"

        def add_options(self, parser):
            super().add_options(parser)
            parser.add_argument("--my-option", help="My custom option")

        def run(self, args, opts):
            # Command implementation here
            spider_name = args[0] if args else None
            print(f"Running custom command for spider: {spider_name}")

Pour des exemples réels, consultez les commandes Scrapy intégrées dans le répertoire `scrapy/commands`_.

.. _scrapy/commands: https://github.com/scrapy/scrapy/tree/master/scrapy/commands

.. autoclass:: scrapy.commands.ScrapyCommand
   :members:
   :undoc-members:

.. note:: Ceci est un :ref:`paramètre pré-crawler <pre-crawler-settings>`.

.. _Deploying your project: https://scrapyd.readthedocs.io/en/latest/deploy.html

Enregistrer des commandes via les points d'entrée de setup.py
-------------------------------------------------------------

Vous pouvez aussi ajouter des commandes Scrapy depuis une bibliothèque externe
en ajoutant une section ``scrapy.commands`` dans les points d'entrée (entry
points) du fichier ``setup.py`` de la bibliothèque.

L'exemple suivant ajoute la commande ``my_command`` :

.. skip: next

.. code-block:: python

  from setuptools import setup, find_packages

  setup(
      name="scrapy-mymodule",
      entry_points={
          "scrapy.commands": [
              "my_command=my_scrapy_module.commands:MyCommand",
          ],
      },
  )

.. setting:: COMMANDS_MODULE

COMMANDS_MODULE
---------------

Valeur par défaut : ``''`` (chaîne vide)

Un module dans lequel chercher les commandes Scrapy personnalisées. Il sert à
ajouter des commandes personnalisées à votre projet Scrapy.

Exemple :

.. code-block:: python

    COMMANDS_MODULE = "mybot.commands"
