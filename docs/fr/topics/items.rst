.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/items.rst`.

.. _topics-items:

=====
Items
=====

.. module:: scrapy.item

Quand on fait du scraping, l'objectif principal est d'extraire des données
structurées à partir de sources non structurées, le plus souvent des pages web.
Les :ref:`spiders <topics-spiders>` peuvent renvoyer les données extraites sous
forme d'`items`, c'est-à-dire des objets Python qui définissent des paires
clé-valeur.

Scrapy prend en charge :ref:`plusieurs types d'items <item-types>`. Quand vous
créez un item, vous pouvez utiliser le type d'item de votre choix. Quand vous
écrivez du code qui reçoit un item, votre code doit :ref:`fonctionner avec
n'importe quel type d'item <supporting-item-types>`.

.. _item-types:

Types d'items
=============

Scrapy prend en charge les types d'items suivants grâce à la bibliothèque
`itemadapter`_ : les :ref:`dictionnaires <dict-items>`, les :ref:`objets Item
<item-objects>`, les :ref:`objets dataclass <dataclass-items>`, les :ref:`objets
attrs <attrs-items>` et les :ref:`modèles Pydantic <pydantic-items>`.

.. _itemadapter: https://github.com/scrapy/itemadapter

.. _dict-items:

Dictionnaires
-------------

En tant que type d'item, :class:`dict` est pratique et familier.

.. _item-objects:

Objets Item
-----------

:class:`Item` fournit une API semblable à celle de :class:`dict`, avec en plus
des fonctionnalités supplémentaires qui en font le type d'item le plus complet :

.. autoclass:: scrapy.Item
   :members: copy, deepcopy, fields
   :undoc-members:

Les objets :class:`Item` reproduisent l'API standard de :class:`dict`, y compris
sa méthode ``__init__``.

:class:`Item` vous permet de définir les noms des champs, de sorte que :

-   une exception :class:`KeyError` est levée quand on utilise un nom de champ
    non défini (autrement dit, cela empêche les fautes de frappe de passer
    inaperçues) ;

-   les :ref:`exporteurs d'items <topics-exporters>` (item exporters) peuvent
    exporter tous les champs par défaut, même si le premier objet extrait n'a
    pas de valeur pour chacun d'eux.

:class:`Item` vous permet aussi de définir des métadonnées de champ, qui peuvent
servir à :ref:`personnaliser la sérialisation
<topics-exporters-field-serialization>`.

:mod:`scrapy.utils.trackref` suit les objets :class:`Item` pour aider à
repérer les fuites de mémoire (voir :ref:`topics-leaks-trackrefs`).

Exemple :

.. code-block:: python

    from scrapy.item import Item, Field


    class CustomItem(Item):
        one_field = Field()
        another_field = Field()

.. _dataclass-items:

Objets dataclass
----------------

:func:`~dataclasses.dataclass` vous permet de définir des classes d'items avec
des noms de champs, de sorte que les :ref:`exporteurs d'items
<topics-exporters>` puissent exporter tous les champs par défaut, même si le
premier objet extrait n'a pas de valeur pour chacun d'eux.

De plus, les items ``dataclass`` vous permettent de :

* définir le type et la valeur par défaut de chaque champ défini ;

* définir des métadonnées de champ personnalisées grâce à
  :func:`dataclasses.field`, qui peuvent servir à :ref:`personnaliser la
  sérialisation <topics-exporters-field-serialization>`.

Exemple :

.. code-block:: python

    from dataclasses import dataclass


    @dataclass
    class CustomItem:
        one_field: str
        another_field: int

.. note:: Les types des champs ne sont pas vérifiés à l'exécution.

.. _attrs-items:

Objets attr.s
-------------

:func:`attr.s` vous permet de définir des classes d'items avec des noms de
champs, de sorte que les :ref:`exporteurs d'items <topics-exporters>` puissent
exporter tous les champs par défaut, même si le premier objet extrait n'a pas de
valeur pour chacun d'eux.

De plus, les items ``attr.s`` vous permettent de :

* définir le type et la valeur par défaut de chaque champ défini ;

* définir des :ref:`métadonnées <attrs:metadata>` de champ personnalisées, qui
  peuvent servir à :ref:`personnaliser la sérialisation
  <topics-exporters-field-serialization>`.

Pour utiliser ce type, le :doc:`paquet attrs <attrs:index>` doit être installé.

Exemple :

.. code-block:: python

    import attr


    @attr.s
    class CustomItem:
        one_field = attr.ib()
        another_field = attr.ib()


.. _pydantic-items:

Modèles Pydantic
----------------

Les modèles `Pydantic <https://docs.pydantic.dev/>`_ permettent de définir des
classes d'items avec des noms de champs, de sorte que les :ref:`exporteurs
d'items <topics-exporters>` puissent exporter tous les champs par défaut, même
si le premier objet extrait n'a pas de valeur pour chacun d'eux.

De plus, les items ``pydantic`` vous permettent aussi de :

* définir le type et la valeur par défaut de chaque champ défini, avec une
  validation des types à l'exécution ;

* définir des métadonnées de champ personnalisées grâce à `pydantic.Field
  <https://docs.pydantic.dev/latest/concepts/fields/>`_, qui peuvent servir à
  :ref:`personnaliser la sérialisation
  <topics-exporters-field-serialization>` ;

* bénéficier d'une validation et d'une conversion automatiques des données,
  fondées sur les annotations de type.

Pour utiliser ce type, le `paquet pydantic <https://docs.pydantic.dev/>`_ doit
être installé.

Exemple :

.. code-block:: python

    from pydantic import BaseModel, Field


    class CustomItem(BaseModel):
        one_field: str = Field(default="", description="First field")
        another_field: int = Field(default=0, description="Second field")

.. note:: Contrairement aux autres types d'items, les modèles Pydantic vérifient
    les types des champs à l'exécution et lèvent des erreurs de validation
    quand les données ont un type invalide.

Travailler avec les objets Item
===============================

.. _topics-items-declaring:

Déclarer des sous-classes d'Item
--------------------------------

Les sous-classes d'Item se déclarent avec une syntaxe de définition de classe
simple et des objets :class:`Field`. Voici un exemple :

.. code-block:: python

    import scrapy


    class Product(scrapy.Item):
        name = scrapy.Field()
        price = scrapy.Field()
        stock = scrapy.Field()
        tags = scrapy.Field()
        last_updated = scrapy.Field(serializer=str)

.. note:: Les personnes qui connaissent `Django`_ remarqueront que les items de
    Scrapy se déclarent de façon similaire aux `modèles Django`_ (Django
    Models), sauf qu'ils sont beaucoup plus simples, car il n'existe pas de
    notion de types de champs différents.

.. _Django: https://www.djangoproject.com/
.. _modèles Django: https://docs.djangoproject.com/en/dev/topics/db/models/


.. _topics-items-fields:

Déclarer des champs
-------------------

Les objets :class:`Field` servent à spécifier des métadonnées pour chaque champ.
Par exemple, ils peuvent stocker la fonction de sérialisation du champ
``last_updated`` illustré ci-dessus.

Vous pouvez spécifier n'importe quel type de métadonnées pour chaque champ. Il
n'y a aucune restriction sur les valeurs acceptées par les objets
:class:`Field`. Pour cette même raison, il n'existe pas de liste de référence de
toutes les clés de métadonnées disponibles. Chaque clé définie dans les objets
:class:`Field` peut être utilisée par un composant différent, et seuls ces
composants la connaissent. Vous pouvez aussi définir et utiliser n'importe
quelle autre clé de :class:`Field` dans votre projet, pour vos propres besoins.
L'objectif principal des objets :class:`Field` est de fournir un moyen de
définir toutes les métadonnées d'un champ en un seul endroit. En général, les
composants dont le comportement dépend de chaque champ utilisent certaines clés
de champ pour configurer ce comportement. Vous devez consulter leur
documentation pour savoir quelles clés de métadonnées sont utilisées par chaque
composant.

Il est important de noter que les objets :class:`Field` utilisés pour déclarer
l'item ne restent pas assignés en tant qu'attributs de classe. On peut y accéder
à la place grâce à l'attribut :attr:`~scrapy.Item.fields`.

.. autoclass:: scrapy.Field

    La classe :class:`Field` n'est qu'un alias de la classe native :class:`dict`
    et ne fournit aucune fonctionnalité ni aucun attribut supplémentaire. En
    d'autres termes, les objets :class:`Field` sont de simples dictionnaires
    Python. Une classe distincte est utilisée pour prendre en charge la
    :ref:`syntaxe de déclaration d'item <topics-items-declaring>` fondée sur les
    attributs de classe.

.. note:: Les métadonnées de champ peuvent aussi être déclarées pour les items
    ``dataclass`` et ``attrs``. Veuillez consulter la documentation de
    `dataclasses.field`_ et de `attr.ib`_ pour plus d'informations.

    .. _dataclasses.field: https://docs.python.org/3/library/dataclasses.html#dataclasses.field
    .. _attr.ib: https://www.attrs.org/en/stable/api-attr.html#attr.ib


Travailler avec les objets Item
-------------------------------

.. skip: start

Voici quelques exemples de tâches courantes réalisées avec des items, en
utilisant l'item ``Product`` :ref:`déclaré plus haut <topics-items-declaring>`.
Vous remarquerez que l'API est très proche de celle de :class:`dict`.

Créer des items
'''''''''''''''

.. code-block:: pycon

    >>> product = Product(name="Desktop PC", price=1000)
    >>> print(product)
    {'name': 'Desktop PC', 'price': 1000}


Lire les valeurs des champs
'''''''''''''''''''''''''''

.. code-block:: pycon

    >>> product["name"]
    Desktop PC
    >>> product.get("name")
    Desktop PC

    >>> product["price"]
    1000

    >>> product["last_updated"]
    Traceback (most recent call last):
        ...
    KeyError: 'last_updated'

    >>> product.get("last_updated", "not set")
    not set

    >>> product["lala"]  # getting unknown field
    Traceback (most recent call last):
        ...
    KeyError: 'lala'

    >>> product.get("lala", "unknown field")
    'unknown field'

    >>> "name" in product  # is name field populated?
    True

    >>> "last_updated" in product  # is last_updated populated?
    False

    >>> "last_updated" in product.fields  # is last_updated a declared field?
    True

    >>> "lala" in product.fields  # is lala a declared field?
    False


Définir les valeurs des champs
''''''''''''''''''''''''''''''

.. code-block:: pycon

    >>> product["last_updated"] = "today"
    >>> product["last_updated"]
    today

    >>> product["lala"] = "test"  # setting unknown field
    Traceback (most recent call last):
        ...
    KeyError: 'Product does not support field: lala'


Accéder à toutes les valeurs renseignées
''''''''''''''''''''''''''''''''''''''''

Pour accéder à toutes les valeurs renseignées, utilisez simplement l'API
habituelle de :class:`dict` :

.. code-block:: pycon

    >>> product.keys()
    ['price', 'name']

    >>> product.items()
    [('price', 1000), ('name', 'Desktop PC')]


.. _copying-items:

Copier des items
''''''''''''''''

Pour copier un item, vous devez d'abord décider si vous voulez une copie
superficielle (shallow copy) ou une copie profonde (deep copy).

Si votre item contient des valeurs :term:`mutables <mutable>`, comme des listes
ou des dictionnaires, une copie superficielle conserve des références vers les
mêmes valeurs mutables dans toutes les copies.

Par exemple, si vous avez un item avec une liste de tags et que vous créez une
copie superficielle de cet item, l'item d'origine et la copie ont la même liste
de tags. Ajouter un tag à la liste de l'un des items l'ajoute aussi à l'autre
item.

Si ce n'est pas le comportement souhaité, utilisez plutôt une copie profonde.

Voir :mod:`copy` pour plus d'informations.

Pour créer une copie superficielle d'un item, vous pouvez soit appeler
:meth:`~scrapy.Item.copy` sur un item existant
(``product2 = product.copy()``), soit instancier votre classe d'item à partir
d'un item existant (``product2 = Product(product)``).

Pour créer une copie profonde, appelez plutôt :meth:`~scrapy.Item.deepcopy`
(``product2 = product.deepcopy()``).


Autres tâches courantes
'''''''''''''''''''''''

Créer des dictionnaires à partir d'items :

.. code-block:: pycon

    >>> dict(product)  # create a dict from all populated values
    {'price': 1000, 'name': 'Desktop PC'}

Créer des items à partir de dictionnaires :

.. code-block:: pycon

    >>> Product({"name": "Laptop PC", "price": 1500})
    {'name': 'Laptop PC', 'price': 1500}

    >>> Product({"name": "Laptop PC", "lala": 1500})  # warning: unknown field in dict
    Traceback (most recent call last):
        ...
    KeyError: 'Product does not support field: lala'


Étendre des sous-classes d'Item
-------------------------------

Vous pouvez étendre des items (pour ajouter d'autres champs ou modifier des
métadonnées de certains champs) en déclarant une sous-classe de votre item
d'origine.

Par exemple :

.. code-block:: python

    class DiscountedProduct(Product):
        discount_percent = scrapy.Field(serializer=str)
        discount_expiration_date = scrapy.Field()

Vous pouvez aussi étendre les métadonnées d'un champ en reprenant les
métadonnées précédentes et en y ajoutant d'autres valeurs, ou en modifiant des
valeurs existantes, comme ceci :

.. code-block:: python

    class SpecificProduct(Product):
        name = scrapy.Field(Product.fields["name"], serializer=my_serializer)

Cela ajoute (ou remplace) la clé de métadonnées ``serializer`` du champ
``name``, tout en conservant toutes les valeurs de métadonnées déjà existantes.

.. skip: end


.. _supporting-item-types:

Prendre en charge tous les types d'items
========================================

Dans le code qui reçoit un item, par exemple les méthodes des :ref:`item
pipelines <topics-item-pipeline>` ou des :ref:`spider middlewares
<topics-spider-middleware>`, la bonne pratique consiste à utiliser la classe
:class:`~itemadapter.ItemAdapter` pour écrire du code qui fonctionne avec
n'importe quel type d'item pris en charge.

Autres classes liées aux items
==============================

.. autoclass:: ItemMeta
