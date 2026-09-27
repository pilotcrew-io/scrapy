.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../topics/selectors.rst`.

.. _topics-selectors:

==========
Sélecteurs
==========

Quand vous scrapez des pages web, la tâche la plus courante consiste à
extraire des données à partir du code source HTML. Plusieurs bibliothèques
permettent d'y parvenir, par exemple :

-   `BeautifulSoup`_ est une bibliothèque de web scraping très populaire chez
    les programmeurs Python. Elle construit un objet Python à partir de la
    structure du code HTML et gère raisonnablement bien le balisage mal formé,
    mais elle a un inconvénient : elle est lente.

-   `lxml`_ est une bibliothèque d'analyse (parsing) XML, qui analyse aussi le
    HTML, avec une API « pythonique » basée sur
    :mod:`~xml.etree.ElementTree`. (lxml ne fait pas partie de la bibliothèque
    standard de Python.)

Scrapy fournit son propre mécanisme d'extraction de données. Ces objets
s'appellent des sélecteurs, car ils « sélectionnent » certaines parties du
document HTML à l'aide d'expressions `XPath`_ ou `CSS`_.

`XPath`_ est un langage permettant de sélectionner des nœuds dans des
documents XML, et qui peut aussi être utilisé avec du HTML. `CSS`_ est un
langage servant à appliquer des styles à des documents HTML. Il définit des
sélecteurs pour associer ces styles à des éléments HTML précis.

.. note::
    Les sélecteurs Scrapy sont une fine couche autour de la bibliothèque
    `parsel`_ ; le but de cette couche est de mieux intégrer les sélecteurs
    aux objets Response de Scrapy.

    `parsel`_ est une bibliothèque de web scraping autonome, qui peut être
    utilisée sans Scrapy. Elle s'appuie sur la bibliothèque `lxml`_ et
    implémente une API simple par-dessus l'API de lxml. Cela signifie que les
    sélecteurs Scrapy ont une vitesse et une précision d'analyse similaires à
    celles de lxml.

Pour transformer des pages entières en texte propre ou en markdown, ou pour
convertir les chaînes que vous sélectionnez en dates, en prix ou en d'autres
valeurs, consultez :ref:`extraction`.

.. _BeautifulSoup: https://www.crummy.com/software/BeautifulSoup/
.. _lxml: https://lxml.de/
.. _XPath: https://www.w3.org/TR/xpath/all/
.. _CSS: https://www.w3.org/TR/selectors
.. _parsel: https://parsel.readthedocs.io/en/latest/

Utiliser les sélecteurs
=======================

Construire des sélecteurs
-------------------------

.. highlight:: python

.. skip: start

Les objets Response exposent une instance de :class:`~scrapy.Selector`
dans l'attribut ``.selector`` :

.. code-block:: pycon

    >>> response.selector.xpath("//span/text()").get()
    'good'

Interroger des réponses avec XPath et CSS est si courant que les réponses
incluent deux raccourcis supplémentaires : ``response.xpath()`` et
``response.css()`` :

.. code-block:: pycon

    >>> response.xpath("//span/text()").get()
    'good'
    >>> response.css("span::text").get()
    'good'

.. skip: end

Les sélecteurs Scrapy sont des instances de la classe
:class:`~scrapy.Selector`, construites en lui passant soit un objet
:class:`~scrapy.http.TextResponse`, soit du balisage sous forme de chaîne (dans
l'argument ``text``).

En général, il n'est pas nécessaire de construire des sélecteurs Scrapy à la
main : l'objet ``response`` est disponible dans les callbacks des spiders, donc
dans la plupart des cas il est plus pratique d'utiliser les raccourcis
``response.css()`` et ``response.xpath()``. En utilisant ``response.selector``
ou l'un de ces raccourcis, vous vous assurez aussi que le corps de la réponse
n'est analysé qu'une seule fois.

Mais si nécessaire, il est possible d'utiliser ``Selector`` directement.
Construction à partir d'un texte :

.. code-block:: pycon

    >>> from scrapy.selector import Selector
    >>> body = "<html><body><span>good</span></body></html>"
    >>> Selector(text=body).xpath("//span/text()").get()
    'good'

Construction à partir d'une réponse (:class:`~scrapy.http.HtmlResponse` est
l'une des sous-classes de :class:`~scrapy.http.TextResponse`) :

.. code-block:: pycon

    >>> from scrapy.selector import Selector
    >>> from scrapy.http import HtmlResponse
    >>> response = HtmlResponse(url="http://example.com", body=body, encoding="utf-8")
    >>> Selector(response=response).xpath("//span/text()").get()
    'good'

``Selector`` choisit automatiquement les meilleures règles d'analyse
(XML ou HTML) en fonction du type de l'entrée.

Utiliser les sélecteurs
-----------------------

.. invisible-code-block: python

    html_response = response = load_response(
        "https://docs.scrapy.org/en/latest/_static/selectors-sample1.html",
        "../../_static/selectors-sample1.html",
    )

Pour expliquer comment utiliser les sélecteurs, nous allons employer le
``Scrapy shell`` (qui permet de tester de manière interactive) et une page
d'exemple hébergée sur le serveur de documentation de Scrapy :

    https://docs.scrapy.org/en/latest/_static/selectors-sample1.html

.. _topics-selectors-htmlcode:

Par souci d'exhaustivité, voici son code HTML complet :

.. literalinclude:: ../../_static/selectors-sample1.html
   :language: html

.. highlight:: sh

Commençons par ouvrir le shell ::

    scrapy shell https://docs.scrapy.org/en/latest/_static/selectors-sample1.html

Ensuite, une fois le shell chargé, la réponse est disponible dans la variable
de shell ``response``, et le sélecteur qui lui est attaché se trouve dans
l'attribut ``response.selector``.

Comme nous travaillons avec du HTML, le sélecteur utilise automatiquement un
analyseur HTML.

.. highlight:: python

En regardant le :ref:`code HTML <topics-selectors-htmlcode>` de cette page,
construisons un XPath pour sélectionner le texte contenu dans la balise
title :

.. code-block:: pycon

    >>> response.xpath("//title/text()")
    [<Selector query='//title/text()' data='Example website'>]

Pour extraire réellement les données textuelles, vous devez appeler les
méthodes ``.get()`` ou ``.getall()`` du sélecteur, comme ceci :

.. code-block:: pycon

    >>> response.xpath("//title/text()").getall()
    ['Example website']
    >>> response.xpath("//title/text()").get()
    'Example website'

``.get()`` renvoie toujours un seul résultat ; s'il y a plusieurs
correspondances, le contenu de la première est renvoyé ; s'il n'y a aucune
correspondance, None est renvoyé. ``.getall()`` renvoie une liste contenant
tous les résultats.

Remarquez que les sélecteurs CSS peuvent sélectionner des nœuds de texte ou
d'attribut grâce aux pseudo-éléments CSS3 :

.. code-block:: pycon

    >>> response.css("title::text").get()
    'Example website'

Comme vous pouvez le constater, les méthodes ``.xpath()`` et ``.css()``
renvoient une instance de :class:`~scrapy.selector.SelectorList`, qui est une
liste de nouveaux sélecteurs. Cette API permet de sélectionner rapidement des
données imbriquées :

.. code-block:: pycon

    >>> response.css("img").xpath("@src").getall()
    ['image1_thumb.jpg',
    'image2_thumb.jpg',
    'image3_thumb.jpg',
    'image4_thumb.jpg',
    'image5_thumb.jpg']

Si vous voulez extraire uniquement le premier élément correspondant, vous
pouvez appeler la méthode ``.get()`` du sélecteur (ou son alias
``.extract_first()``, couramment utilisé dans les anciennes versions de
Scrapy) :

.. code-block:: pycon

    >>> response.xpath('//div[@id="images"]/a/text()').get()
    'Name: My image 1 '

Elle renvoie ``None`` si aucun élément n'a été trouvé :

.. code-block:: pycon

    >>> response.xpath('//div[@id="not-exists"]/text()').get() is None
    True

Une valeur de retour par défaut peut être passée en argument, pour être
utilisée à la place de ``None`` :

.. code-block:: pycon

    >>> response.xpath('//div[@id="not-exists"]/text()').get(default="not-found")
    'not-found'

Au lieu d'utiliser un XPath comme ``'@src'``, il est possible d'interroger les
attributs avec la propriété ``.attrib`` d'un :class:`~scrapy.Selector` :

.. code-block:: pycon

    >>> [img.attrib["src"] for img in response.css("img")]
    ['image1_thumb.jpg',
    'image2_thumb.jpg',
    'image3_thumb.jpg',
    'image4_thumb.jpg',
    'image5_thumb.jpg']

Par raccourci, ``.attrib`` est aussi disponible directement sur SelectorList ;
elle renvoie les attributs du premier élément correspondant :

.. code-block:: pycon

    >>> response.css("img").attrib["src"]
    'image1_thumb.jpg'

C'est surtout utile quand un seul résultat est attendu, par exemple quand on
sélectionne par id, ou quand on sélectionne des éléments uniques sur une page
web :

.. code-block:: pycon

    >>> response.css("base").attrib["href"]
    'http://example.com/'

Récupérons maintenant l'URL de base et quelques liens d'images :

.. code-block:: pycon

    >>> response.xpath("//base/@href").get()
    'http://example.com/'

    >>> response.css("base::attr(href)").get()
    'http://example.com/'

    >>> response.css("base").attrib["href"]
    'http://example.com/'

    >>> response.xpath('//a[contains(@href, "image")]/@href').getall()
    ['image1.html',
    'image2.html',
    'image3.html',
    'image4.html',
    'image5.html']

    >>> response.css("a[href*=image]::attr(href)").getall()
    ['image1.html',
    'image2.html',
    'image3.html',
    'image4.html',
    'image5.html']

    >>> response.xpath('//a[contains(@href, "image")]/img/@src').getall()
    ['image1_thumb.jpg',
    'image2_thumb.jpg',
    'image3_thumb.jpg',
    'image4_thumb.jpg',
    'image5_thumb.jpg']

    >>> response.css("a[href*=image] img::attr(src)").getall()
    ['image1_thumb.jpg',
    'image2_thumb.jpg',
    'image3_thumb.jpg',
    'image4_thumb.jpg',
    'image5_thumb.jpg']

.. _topics-selectors-css-extensions:

Extensions des sélecteurs CSS
-----------------------------

Selon les standards du W3C, les `sélecteurs CSS`_ ne permettent pas de
sélectionner des nœuds de texte ni des valeurs d'attribut. Mais dans un
contexte de web scraping, pouvoir les sélectionner est tellement essentiel que
Scrapy (parsel) implémente quelques **pseudo-éléments non standard** :

* pour sélectionner des nœuds de texte, utilisez ``::text``
* pour sélectionner des valeurs d'attribut, utilisez ``::attr(name)`` où
  *name* est le nom de l'attribut dont vous voulez la valeur

.. warning::
    Ces pseudo-éléments sont spécifiques à Scrapy et à Parsel. Ils ne
    fonctionneront très probablement pas avec d'autres bibliothèques comme
    `lxml`_ ou `PyQuery`_.

.. _PyQuery: https://pypi.org/project/pyquery/

Exemples :

* ``title::text`` sélectionne les nœuds de texte enfants d'un élément
  ``<title>`` descendant :

.. code-block:: pycon

    >>> response.css("title::text").get()
    'Example website'

* ``*::text`` sélectionne tous les nœuds de texte descendants du contexte
  courant du sélecteur :

.. skip: next
.. code-block:: pycon

    >>> response.css("#images *::text").getall()
    ['\n   ',
    'Name: My image 1 ',
    '\n   ',
    'Name: My image 2 ',
    '\n   ',
    'Name: My image 3 ',
    '\n   ',
    'Name: My image 4 ',
    '\n   ',
    'Name: My image 5 ',
    '\n  ']

* ``foo::text`` ne renvoie aucun résultat si l'élément ``foo`` existe mais ne
  contient aucun texte (c'est-à-dire que son texte est vide) :

.. code-block:: pycon

  >>> response.css("img::text").getall()
  []

  Cela signifie que ``.css('foo::text').get()`` peut renvoyer None même si un
  élément existe. Utilisez ``default=''`` si vous voulez toujours obtenir une
  chaîne :

.. code-block:: pycon

    >>> response.css("img::text").get()
    >>> response.css("img::text").get(default="")
    ''

* ``a::attr(href)`` sélectionne la valeur de l'attribut *href* des liens
  descendants :

.. code-block:: pycon

    >>> response.css("a::attr(href)").getall()
    ['image1.html',
    'image2.html',
    'image3.html',
    'image4.html',
    'image5.html']

.. note::
    Voir aussi : :ref:`selecting-attributes`.

.. note::
    Vous ne pouvez pas enchaîner ces pseudo-éléments. Mais en pratique cela
    n'aurait pas beaucoup de sens : les nœuds de texte n'ont pas d'attributs,
    et les valeurs d'attribut sont déjà des chaînes de caractères et n'ont pas
    de nœuds enfants.

.. _sélecteurs CSS: https://www.w3.org/TR/selectors-3/#selectors

.. _topics-selectors-nesting-selectors:

Imbriquer des sélecteurs
------------------------

Les méthodes de sélection (``.xpath()`` ou ``.css()``) renvoient une liste de
sélecteurs du même type, vous pouvez donc appeler à nouveau ces méthodes de
sélection sur ces sélecteurs. Voici un exemple :

.. code-block:: pycon

    >>> links = response.xpath('//a[contains(@href, "image")]')
    >>> links.getall()
    ['<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg" alt="image1"></a>',
    '<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg" alt="image2"></a>',
    '<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg" alt="image3"></a>',
    '<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg" alt="image4"></a>',
    '<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg" alt="image5"></a>']

    >>> for index, link in enumerate(links):
    ...     href_xpath = link.xpath("@href").get()
    ...     img_xpath = link.xpath("img/@src").get()
    ...     print(f"Link number {index} points to url {href_xpath!r} and image {img_xpath!r}")
    ...
    Link number 0 points to url 'image1.html' and image 'image1_thumb.jpg'
    Link number 1 points to url 'image2.html' and image 'image2_thumb.jpg'
    Link number 2 points to url 'image3.html' and image 'image3_thumb.jpg'
    Link number 3 points to url 'image4.html' and image 'image4_thumb.jpg'
    Link number 4 points to url 'image5.html' and image 'image5_thumb.jpg'

.. _selecting-attributes:

Sélectionner les attributs d'un élément
---------------------------------------

Il existe plusieurs façons d'obtenir la valeur d'un attribut. Tout d'abord, on
peut utiliser la syntaxe XPath :

.. code-block:: pycon

    >>> response.xpath("//a/@href").getall()
    ['image1.html', 'image2.html', 'image3.html', 'image4.html', 'image5.html']

La syntaxe XPath présente quelques avantages : c'est une fonctionnalité
standard de XPath, et les ``@attributs`` peuvent être utilisés dans d'autres
parties d'une expression XPath, par exemple pour filtrer selon la valeur d'un
attribut.

Scrapy fournit aussi une extension des sélecteurs CSS (``::attr(...)``) qui
permet d'obtenir des valeurs d'attribut :

.. code-block:: pycon

    >>> response.css("a::attr(href)").getall()
    ['image1.html', 'image2.html', 'image3.html', 'image4.html', 'image5.html']

En plus de cela, il existe une propriété ``.attrib`` de Selector. Vous pouvez
l'utiliser si vous préférez rechercher les attributs dans du code Python, sans
passer par des XPath ni par les extensions CSS :

.. code-block:: pycon

    >>> [a.attrib["href"] for a in response.css("a")]
    ['image1.html', 'image2.html', 'image3.html', 'image4.html', 'image5.html']

Cette propriété est aussi disponible sur SelectorList ; elle renvoie un
dictionnaire contenant les attributs du premier élément correspondant. C'est
pratique quand on s'attend à ce qu'un sélecteur donne un seul résultat (par
exemple quand on sélectionne par ID d'élément, ou quand on sélectionne un
élément unique dans une page) :

.. code-block:: pycon

    >>> response.css("base").attrib
    {'href': 'http://example.com/'}
    >>> response.css("base").attrib["href"]
    'http://example.com/'

La propriété ``.attrib`` d'une SelectorList vide est vide :

.. code-block:: pycon

    >>> response.css("foo").attrib
    {}

Utiliser les sélecteurs avec des expressions régulières
-------------------------------------------------------

:class:`~scrapy.Selector` possède aussi une méthode ``.re()`` pour extraire des
données à l'aide d'expressions régulières. Cependant, contrairement aux
méthodes ``.xpath()`` ou ``.css()``, ``.re()`` renvoie une liste de chaînes de
caractères. On ne peut donc pas imbriquer des appels à ``.re()``.

Voici un exemple servant à extraire les noms d'images à partir du :ref:`code
HTML <topics-selectors-htmlcode>` ci-dessus :

.. code-block:: pycon

    >>> response.xpath('//a[contains(@href, "image")]/text()').re(r"Name:\s*(.*)")
    ['My image 1 ',
    'My image 2 ',
    'My image 3 ',
    'My image 4 ',
    'My image 5 ']

Il existe un utilitaire supplémentaire, nommé ``.re_first()``, qui est pour
``.re()`` ce que ``.get()`` (et son alias ``.extract_first()``) est aux
sélecteurs. Utilisez-le pour extraire seulement la première chaîne
correspondante :

.. code-block:: pycon

    >>> response.xpath('//a[contains(@href, "image")]/text()').re_first(r"Name:\s*(.*)")
    'My image 1 '

.. _old-extraction-api:

extract() et extract_first()
----------------------------

Si vous utilisez Scrapy depuis longtemps, vous connaissez probablement les
méthodes de sélecteur ``.extract()`` et ``.extract_first()``. De nombreux
articles de blog et tutoriels les utilisent également. Ces méthodes sont
toujours prises en charge par Scrapy, et il n'est **pas prévu** de les
déprécier.

Cependant, la documentation d'utilisation de Scrapy est désormais écrite avec
les méthodes ``.get()`` et ``.getall()``. Nous pensons que ces nouvelles
méthodes donnent un code plus concis et plus lisible.

Les exemples suivants montrent la correspondance entre ces méthodes.

1.  ``SelectorList.get()`` est identique à ``SelectorList.extract_first()`` :

.. code-block:: pycon

    >>> response.css("a::attr(href)").get()
    'image1.html'
    >>> response.css("a::attr(href)").extract_first()
    'image1.html'

2.  ``SelectorList.getall()`` est identique à ``SelectorList.extract()`` :

.. code-block:: pycon

    >>> response.css("a::attr(href)").getall()
    ['image1.html', 'image2.html', 'image3.html', 'image4.html', 'image5.html']
    >>> response.css("a::attr(href)").extract()
    ['image1.html', 'image2.html', 'image3.html', 'image4.html', 'image5.html']

3.  ``Selector.get()`` est identique à ``Selector.extract()`` :

.. code-block:: pycon

    >>> response.css("a::attr(href)")[0].get()
    'image1.html'
    >>> response.css("a::attr(href)")[0].extract()
    'image1.html'

4.  Par cohérence, il existe aussi ``Selector.getall()``, qui renvoie une
    liste :

.. code-block:: pycon

    >>> response.css("a::attr(href)")[0].getall()
    ['image1.html']

La principale différence est donc que le résultat des méthodes ``.get()`` et
``.getall()`` est plus prévisible : ``.get()`` renvoie toujours un seul
résultat, et ``.getall()`` renvoie toujours une liste de tous les résultats
extraits. Avec la méthode ``.extract()``, il n'était pas toujours évident de
savoir si le résultat était une liste ou non ; pour obtenir un seul résultat,
il fallait appeler soit ``.extract()``, soit ``.extract_first()``.


.. _topics-selectors-xpaths:

Travailler avec XPath
=====================

Voici quelques conseils qui peuvent vous aider à utiliser XPath efficacement
avec les sélecteurs Scrapy. Si vous connaissez encore mal XPath, vous
voudrez peut-être commencer par consulter ce `tutoriel XPath`_.

.. note::
    Certains de ces conseils sont tirés de `cet article du blog de Zyte`_.

.. _tutoriel XPath: http://www.zvon.org/comp/r/tut-XPath_1.html
.. _cet article du blog de Zyte: https://www.zyte.com/blog/xpath-tips-from-the-web-scraping-trenches/


.. _topics-selectors-relative-xpaths:

Travailler avec des XPath relatifs
----------------------------------

Gardez à l'esprit que si vous imbriquez des sélecteurs et que vous utilisez un
XPath commençant par ``/``, ce XPath sera absolu par rapport au document, et
non relatif au ``Selector`` sur lequel vous l'appelez.

Par exemple, supposons que vous vouliez extraire tous les éléments ``<p>``
situés à l'intérieur d'éléments ``<div>``. Vous commenceriez par récupérer tous
les éléments ``<div>`` :

.. code-block:: pycon

    >>> divs = response.xpath("//div")

Au premier abord, vous pourriez être tenté d'utiliser l'approche suivante, qui
est fausse, car elle extrait en réalité tous les éléments ``<p>`` du document,
et pas seulement ceux qui sont à l'intérieur d'éléments ``<div>`` :

.. code-block:: pycon

    >>> for p in divs.xpath("//p"):  # this is wrong - gets all <p> from the whole document
    ...     print(p.get())
    ...

Voici la bonne manière de procéder (remarquez le point qui préfixe le XPath
``.//p``) :

.. code-block:: pycon

    >>> for p in divs.xpath(".//p"):  # extracts all <p> inside
    ...     print(p.get())
    ...

Un autre cas courant consiste à extraire tous les enfants directs ``<p>`` :

.. code-block:: pycon

    >>> for p in divs.xpath("p"):
    ...     print(p.get())
    ...

Pour plus de détails sur les XPath relatifs, consultez la section `Location
Paths`_ de la spécification XPath.

.. _Location Paths: https://www.w3.org/TR/xpath-10/#location-paths

Pour une recherche par classe, envisager le CSS
-----------------------------------------------

Comme un élément peut porter plusieurs classes CSS, la façon XPath de
sélectionner des éléments par classe est plutôt verbeuse ::

    *[contains(concat(' ', normalize-space(@class), ' '), ' someclass ')]

Si vous utilisez ``@class='someclass'``, vous risquez de manquer des éléments
qui ont d'autres classes ; et si vous utilisez simplement
``contains(@class, 'someclass')`` pour compenser, vous risquez d'obtenir plus
d'éléments que voulu, s'ils ont un autre nom de classe qui contient la chaîne
``someclass``.

Il se trouve que les sélecteurs Scrapy permettent d'enchaîner les sélecteurs :
la plupart du temps, vous pouvez donc sélectionner par classe en CSS, puis
passer à XPath quand c'est nécessaire :

.. code-block:: pycon

    >>> from scrapy import Selector
    >>> sel = Selector(
    ...     text='<div class="hero shout"><time datetime="2014-07-23 19:00">Special date</time></div>'
    ... )
    >>> sel.css(".shout").xpath("./time/@datetime").getall()
    ['2014-07-23 19:00']

C'est plus propre que l'astuce XPath verbeuse montrée ci-dessus. N'oubliez pas
simplement d'utiliser le ``.`` dans les expressions XPath qui suivent.

Attention à la différence entre //node[1] et (//node)[1]
--------------------------------------------------------

``//node[1]`` sélectionne tous les nœuds qui apparaissent en premier sous leur
parent respectif.

``(//node)[1]`` sélectionne tous les nœuds du document, puis ne garde que le
premier d'entre eux.

Exemple :

.. code-block:: pycon

    >>> from scrapy import Selector
    >>> sel = Selector(text="""
    ...     <ul class="list">
    ...         <li>1</li>
    ...         <li>2</li>
    ...         <li>3</li>
    ...     </ul>
    ...     <ul class="list">
    ...         <li>4</li>
    ...         <li>5</li>
    ...         <li>6</li>
    ...     </ul>""")
    ...
    >>> xp = lambda x: sel.xpath(x).getall()

Ceci récupère tous les premiers éléments ``<li>``, quel que soit leur parent :

.. code-block:: pycon

    >>> xp("//li[1]")
    ['<li>1</li>', '<li>4</li>']

Et ceci récupère le premier élément ``<li>`` de tout le document :

.. code-block:: pycon

    >>> xp("(//li)[1]")
    ['<li>1</li>']

Ceci récupère tous les premiers éléments ``<li>`` situés sous un parent
``<ul>`` :

.. code-block:: pycon

    >>> xp("//ul/li[1]")
    ['<li>1</li>', '<li>4</li>']

Et ceci récupère le premier élément ``<li>`` situé sous un parent ``<ul>`` dans
tout le document :

.. code-block:: pycon

    >>> xp("(//ul/li)[1]")
    ['<li>1</li>']

Utiliser des nœuds texte dans une condition
-------------------------------------------

Quand vous devez utiliser le contenu textuel comme argument d'une `fonction de
chaîne XPath`_, évitez d'utiliser ``.//text()`` et utilisez simplement ``.``
à la place.

En effet, l'expression ``.//text()`` produit une collection d'éléments de
texte, c'est-à-dire un *node-set* (ensemble de nœuds). Et quand un node-set est
converti en chaîne, ce qui arrive lorsqu'il est passé en argument à une
fonction de chaîne comme ``contains()`` ou ``starts-with()``, on obtient
uniquement le texte du premier élément.

Exemple :

.. code-block:: pycon

    >>> from scrapy import Selector
    >>> sel = Selector(
    ...     text='<a href="#">Click here to go to the <strong>Next Page</strong></a>'
    ... )

Conversion d'un *node-set* en chaîne :

.. code-block:: pycon

    >>> sel.xpath("//a//text()").getall()  # take a peek at the node-set
    ['Click here to go to the ', 'Next Page']
    >>> sel.xpath("string(//a[1]//text())").getall()  # convert it to string
    ['Click here to go to the ']

Un *nœud* converti en chaîne, en revanche, regroupe le texte du nœud lui-même
et celui de tous ses descendants :

.. code-block:: pycon

    >>> sel.xpath("//a[1]").getall()  # select the first node
    ['<a href="#">Click here to go to the <strong>Next Page</strong></a>']
    >>> sel.xpath("string(//a[1])").getall()  # convert it to string
    ['Click here to go to the Next Page']

Ainsi, utiliser le node-set ``.//text()`` ne sélectionnera rien dans ce cas :

.. code-block:: pycon

    >>> sel.xpath("//a[contains(.//text(), 'Next Page')]").getall()
    []

Mais utiliser le ``.`` pour désigner le nœud fonctionne :

.. code-block:: pycon

    >>> sel.xpath("//a[contains(., 'Next Page')]").getall()
    ['<a href="#">Click here to go to the <strong>Next Page</strong></a>']

.. _fonction de chaîne XPath: https://www.w3.org/TR/xpath-10/#section-String-Functions

.. _topics-selectors-xpath-variables:

Variables dans les expressions XPath
------------------------------------

XPath vous permet de référencer des variables dans vos expressions, avec la
syntaxe ``$somevariable``. C'est assez proche des requêtes paramétrées ou des
requêtes préparées (prepared statements) du monde SQL, où l'on remplace
certains arguments de ses requêtes par des espaces réservés comme ``?``, qui
sont ensuite remplacés par les valeurs passées avec la requête.

Voici un exemple pour trouver un élément d'après la valeur de son attribut
« id », sans l'écrire en dur dans l'expression (comme nous l'avons fait
précédemment) :

.. code-block:: pycon

    >>> # `$val` used in the expression, a `val` argument needs to be passed
    >>> response.xpath("//div[@id=$val]/a/text()", val="images").get()
    'Name: My image 1 '

Voici un autre exemple, pour trouver l'attribut « id » d'une balise ``<div>``
contenant cinq enfants ``<a>`` (ici nous passons la valeur ``5`` sous forme
d'entier) :

.. code-block:: pycon

    >>> response.xpath("//div[count(a)=$cnt]/@id", cnt=5).get()
    'images'

Toutes les références à des variables doivent avoir une valeur associée lors de
l'appel à ``.xpath()`` (sinon vous obtiendrez une exception
``ValueError: XPath error:``). Cela se fait en passant autant d'arguments
nommés que nécessaire.

`parsel`_, la bibliothèque qui alimente les sélecteurs Scrapy, donne plus de
détails et d'exemples sur les `variables XPath`_.

.. _variables XPath: https://parsel.readthedocs.io/en/latest/usage.html#variables-in-xpath-expressions


.. _removing-namespaces:

Supprimer les espaces de noms (namespaces)
------------------------------------------

.. skip: start

Quand on travaille sur des projets de scraping, il est souvent très pratique de
se débarrasser complètement des espaces de noms et de ne travailler qu'avec les
noms d'éléments, afin d'écrire des XPath plus simples et plus commodes. Vous
pouvez utiliser pour cela la méthode :meth:`.Selector.remove_namespaces`.

Illustrons cela avec le flux atom du blog Python Insider.

.. highlight:: sh

Tout d'abord, nous ouvrons le shell avec l'URL que nous voulons scraper ::

    $ scrapy shell https://feeds.feedburner.com/PythonInsider

Voici le début du fichier ::

    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet ...
    <feed xmlns="http://www.w3.org/2005/Atom"
          xmlns:openSearch="http://a9.com/-/spec/opensearchrss/1.0/"
          xmlns:blogger="http://schemas.google.com/blogger/2008"
          xmlns:georss="http://www.georss.org/georss"
          xmlns:gd="http://schemas.google.com/g/2005"
          xmlns:thr="http://purl.org/syndication/thread/1.0"
          xmlns:feedburner="http://rssnamespace.org/feedburner/ext/1.0">
      ...

Vous pouvez voir plusieurs déclarations d'espaces de noms, dont un espace par
défaut ``"http://www.w3.org/2005/Atom"`` et un autre utilisant le préfixe
``gd:`` pour ``"http://schemas.google.com/g/2005"``.

.. highlight:: python

Une fois dans le shell, nous pouvons essayer de sélectionner tous les objets
``<link>`` et constater que cela ne fonctionne pas (car l'espace de noms XML
Atom masque ces nœuds) :

.. code-block:: pycon

    >>> response.xpath("//link")
    []

Mais une fois que nous appelons la méthode :meth:`.Selector.remove_namespaces`,
tous les nœuds sont accessibles directement par leur nom :

.. code-block:: pycon

    >>> response.selector.remove_namespaces()
    >>> response.xpath("//link")
    [<Selector query='//link' data='<link rel="alternate" type="text/html" h'>,
        <Selector query='//link' data='<link rel="next" type="application/atom+'>,
        ...

Si vous vous demandez pourquoi la suppression des espaces de noms n'est pas
toujours appelée par défaut, plutôt que d'avoir à l'appeler manuellement, c'est
pour deux raisons, données par ordre d'importance :

1. Supprimer les espaces de noms oblige à parcourir et à modifier tous les
   nœuds du document, ce qui est une opération assez coûteuse à effectuer par
   défaut pour tous les documents explorés par Scrapy.

2. Il peut y avoir des cas où l'utilisation des espaces de noms est réellement
   nécessaire, lorsque des noms d'éléments entrent en conflit entre différents
   espaces de noms. Ces cas sont toutefois très rares.

.. skip: end

Utiliser les extensions EXSLT
-----------------------------

Étant construits au-dessus de `lxml`_, les sélecteurs Scrapy prennent en charge
certaines extensions `EXSLT`_ et sont livrés avec ces espaces de noms
pré-enregistrés, utilisables dans les expressions XPath :


========  =====================================    =======================
préfixe   espace de noms                           usage
========  =====================================    =======================
re        \http://exslt.org/regular-expressions    `regular expressions`_
set       \http://exslt.org/sets                   `set manipulation`_
========  =====================================    =======================

Expressions régulières
~~~~~~~~~~~~~~~~~~~~~~

La fonction ``test()``, par exemple, peut se révéler très utile quand les
fonctions XPath ``starts-with()`` ou ``contains()`` ne suffisent pas.

Exemple : sélectionner les liens dans des éléments de liste dont l'attribut
« class » se termine par un chiffre :

.. code-block:: pycon

    >>> from scrapy import Selector
    >>> doc = """
    ... <div>
    ...     <ul>
    ...         <li class="item-0"><a href="link1.html">first item</a></li>
    ...         <li class="item-1"><a href="link2.html">second item</a></li>
    ...         <li class="item-inactive"><a href="link3.html">third item</a></li>
    ...         <li class="item-1"><a href="link4.html">fourth item</a></li>
    ...         <li class="item-0"><a href="link5.html">fifth item</a></li>
    ...     </ul>
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> sel.xpath("//li//@href").getall()
    ['link1.html', 'link2.html', 'link3.html', 'link4.html', 'link5.html']
    >>> sel.xpath(r'//li[re:test(@class, "item-\d$")]//@href').getall()
    ['link1.html', 'link2.html', 'link4.html', 'link5.html']

.. warning:: La bibliothèque C ``libxslt`` ne prend pas en charge nativement
    les expressions régulières EXSLT, donc l'implémentation de `lxml`_ utilise
    des points d'accroche (hooks) vers le module ``re`` de Python. Par
    conséquent, utiliser des fonctions d'expressions régulières dans vos
    expressions XPath peut ajouter une légère pénalité de performance.

Opérations sur les ensembles
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Elles peuvent être pratiques, par exemple, pour exclure des parties d'un arbre
de document avant d'extraire des éléments de texte.

Exemple : extraction de microdonnées (contenu d'exemple tiré de
https://schema.org/Product) avec des groupes d'itemscopes et les itemprops
correspondants :

.. skip: next

.. code-block:: pycon

    >>> doc = """
    ... <div itemscope itemtype="http://schema.org/Product">
    ...   <span itemprop="name">Kenmore White 17" Microwave</span>
    ...   <img src="kenmore-microwave-17in.jpg" alt='Kenmore 17" Microwave' />
    ...   <div itemprop="aggregateRating"
    ...     itemscope itemtype="http://schema.org/AggregateRating">
    ...    Rated <span itemprop="ratingValue">3.5</span>/5
    ...    based on <span itemprop="reviewCount">11</span> customer reviews
    ...   </div>
    ...   <div itemprop="offers" itemscope itemtype="http://schema.org/Offer">
    ...     <span itemprop="price">$55.00</span>
    ...     <link itemprop="availability" href="http://schema.org/InStock" />In stock
    ...   </div>
    ...   Product description:
    ...   <span itemprop="description">0.7 cubic feet countertop microwave.
    ...   Has six preset cooking categories and convenience features like
    ...   Add-A-Minute and Child Lock.</span>
    ...   Customer reviews:
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Not a happy camper</span> -
    ...     by <span itemprop="author">Ellie</span>,
    ...     <meta itemprop="datePublished" content="2011-04-01">April 1, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1">
    ...       <span itemprop="ratingValue">1</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">The lamp burned out and now I have to replace
    ...     it. </span>
    ...   </div>
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Value purchase</span> -
    ...     by <span itemprop="author">Lucas</span>,
    ...     <meta itemprop="datePublished" content="2011-03-25">March 25, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1"/>
    ...       <span itemprop="ratingValue">4</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">Great microwave for the price. It is small and
    ...     fits in my apartment.</span>
    ...   </div>
    ...   ...
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> for scope in sel.xpath("//div[@itemscope]"):
    ...     print("current scope:", scope.xpath("@itemtype").getall())
    ...     props = scope.xpath("""
    ...                 set:difference(./descendant::*/@itemprop,
    ...                                .//*[@itemscope]/*/@itemprop)""")
    ...     print(f"    properties: {props.getall()}")
    ...     print("")
    ...

    current scope: ['http://schema.org/Product']
        properties: ['name', 'aggregateRating', 'offers', 'description', 'review', 'review']

    current scope: ['http://schema.org/AggregateRating']
        properties: ['ratingValue', 'reviewCount']

    current scope: ['http://schema.org/Offer']
        properties: ['price', 'availability']

    current scope: ['http://schema.org/Review']
        properties: ['name', 'author', 'datePublished', 'reviewRating', 'description']

    current scope: ['http://schema.org/Rating']
        properties: ['worstRating', 'ratingValue', 'bestRating']

    current scope: ['http://schema.org/Review']
        properties: ['name', 'author', 'datePublished', 'reviewRating', 'description']

    current scope: ['http://schema.org/Rating']
        properties: ['worstRating', 'ratingValue', 'bestRating']


Ici, nous parcourons d'abord les éléments ``itemscope`` et, pour chacun,
nous cherchons tous les éléments ``itemprops`` en excluant ceux qui se trouvent
eux-mêmes à l'intérieur d'un autre ``itemscope``.

.. _EXSLT: https://exslt.github.io/
.. _regular expressions: https://exslt.github.io/regexp/index.html
.. _set manipulation: https://exslt.github.io/set/index.html

Autres extensions XPath
-----------------------

Les sélecteurs Scrapy fournissent aussi une fonction d'extension XPath dont on
regrettait cruellement l'absence, ``has-class``, qui renvoie ``True`` pour les
nœuds possédant toutes les classes HTML indiquées.

Pour le HTML suivant :

.. code-block:: pycon

    >>> from scrapy.http import HtmlResponse
    >>> response = HtmlResponse(
    ...     url="http://example.com",
    ...     body="""
    ... <html>
    ...     <body>
    ...         <p class="foo bar-baz">First</p>
    ...         <p class="foo">Second</p>
    ...         <p class="bar">Third</p>
    ...         <p>Fourth</p>
    ...     </body>
    ... </html>
    ... """,
    ...     encoding="utf-8",
    ... )

Vous pouvez l'utiliser ainsi :

.. code-block:: pycon

    >>> response.xpath('//p[has-class("foo")]')
    [<Selector query='//p[has-class("foo")]' data='<p class="foo bar-baz">First</p>'>,
    <Selector query='//p[has-class("foo")]' data='<p class="foo">Second</p>'>]
    >>> response.xpath('//p[has-class("foo", "bar-baz")]')
    [<Selector query='//p[has-class("foo", "bar-baz")]' data='<p class="foo bar-baz">First</p>'>]
    >>> response.xpath('//p[has-class("foo", "bar")]')
    []

Le XPath ``//p[has-class("foo", "bar-baz")]`` est donc à peu près équivalent
au CSS ``p.foo.bar-baz``. Notez qu'il est plus lent dans la plupart des cas :
c'est une fonction en Python pur qui est appelée pour chaque nœud concerné,
alors que la recherche CSS est traduite en XPath et s'exécute donc plus
efficacement. Côté performances, son usage est donc limité aux situations qui
se décrivent difficilement avec des sélecteurs CSS.

Parsel simplifie également l'ajout de vos propres extensions XPath avec
:func:`~parsel.xpathfuncs.set_xpathfunc`.

.. _topics-selectors-ref:

Référence des sélecteurs intégrés
=================================

.. module:: scrapy.selector

Objets Selector
---------------

.. autoclass:: scrapy.Selector

  .. automethod:: xpath

      .. note::

          Par commodité, cette méthode peut être appelée avec
          ``response.xpath()``

  .. automethod:: css

      .. note::

          Par commodité, cette méthode peut être appelée avec
          ``response.css()``

  .. automethod:: jmespath

      .. note::

          Par commodité, cette méthode peut être appelée avec
          ``response.jmespath()``

  .. automethod:: get

     Voir aussi : :ref:`old-extraction-api`

  .. autoattribute:: attrib

     Voir aussi : :ref:`selecting-attributes`.

  .. automethod:: re

  .. automethod:: re_first

  .. automethod:: register_namespace

  .. automethod:: remove_namespaces

  .. automethod:: __bool__

  .. automethod:: getall

     Cette méthode est ajoutée à Selector par cohérence ; elle est plus utile
     avec SelectorList. Voir aussi : :ref:`old-extraction-api`

Objets SelectorList
-------------------

.. autoclass:: SelectorList

   .. automethod:: xpath

   .. automethod:: css

   .. automethod:: jmespath

   .. automethod:: getall

      Voir aussi : :ref:`old-extraction-api`

   .. automethod:: get

      Voir aussi : :ref:`old-extraction-api`

   .. automethod:: re

   .. automethod:: re_first

   .. autoattribute:: attrib

      Voir aussi : :ref:`selecting-attributes`.

.. _selector-examples:

Exemples
========

.. _selector-examples-html:

Exemples de sélecteurs sur une réponse HTML
-------------------------------------------

Voici quelques exemples de :class:`~scrapy.Selector` pour illustrer plusieurs
concepts. Dans tous les cas, nous supposons qu'un :class:`~scrapy.Selector` a
déjà été instancié avec un objet :class:`~scrapy.http.HtmlResponse`, comme
ceci :

.. code-block:: python

      sel = Selector(html_response)

1. Sélectionner tous les éléments ``<h1>`` du corps d'une réponse HTML, en
   renvoyant une liste d'objets :class:`~scrapy.Selector` (c'est-à-dire un
   objet :class:`SelectorList`) :

   .. code-block:: python

      sel.xpath("//h1")

2. Extraire le texte de tous les éléments ``<h1>`` du corps d'une réponse HTML,
   en renvoyant une liste de chaînes de caractères :

   .. code-block:: python

      sel.xpath("//h1").getall()  # this includes the h1 tag
      sel.xpath("//h1/text()").getall()  # this excludes the h1 tag

3. Parcourir toutes les balises ``<p>`` et afficher leur attribut class :


   .. code-block:: python

      for node in sel.xpath("//p"):
          print(node.attrib["class"])


.. _selector-examples-xml:

Exemples de sélecteurs sur une réponse XML
------------------------------------------

.. skip: start

Voici quelques exemples pour illustrer des concepts avec des objets
:class:`~scrapy.Selector` instanciés avec un objet
:class:`~scrapy.http.XmlResponse` :

.. code-block:: python

      sel = Selector(xml_response)

1. Sélectionner tous les éléments ``<product>`` du corps d'une réponse XML, en
   renvoyant une liste d'objets :class:`~scrapy.Selector` (c'est-à-dire un
   objet :class:`SelectorList`) :

   .. code-block:: python

      sel.xpath("//product")

2. Extraire tous les prix d'un `flux XML Google Base`_, ce qui nécessite
   d'enregistrer un espace de noms :

   .. code-block:: python

      sel.register_namespace("g", "http://base.google.com/ns/1.0")
      sel.xpath("//g:price").getall()

.. skip: end

.. _flux XML Google Base: https://support.google.com/merchants/answer/14987622
