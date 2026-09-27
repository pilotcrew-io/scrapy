.. note::
   Traduction française non officielle, à but pédagogique, réalisée par PilotCrew à partir de la documentation officielle de `Scrapy <https://scrapy.org>`_. En cas de doute ou d'ambiguïté, se référer à la version anglaise d'origine : `../../intro/install.rst`.

.. _intro-install:

====================
Guide d'installation
====================

.. _faq-python-versions:

Versions de Python prises en charge
===================================

Scrapy nécessite Python 3.10 ou une version supérieure, soit avec
l'implémentation CPython (celle utilisée par défaut), soit avec l'implémentation
PyPy (voir :ref:`python:implementations`).

.. _intro-install-scrapy:

Installer Scrapy
================

Si vous utilisez `Anaconda`_ ou `Miniconda`_, vous pouvez installer le paquet
depuis le canal `conda-forge`_, qui propose des paquets à jour pour Linux,
Windows et macOS.

Pour installer Scrapy avec ``conda``, exécutez::

  conda install -c conda-forge scrapy

Sinon, si vous savez déjà installer des paquets Python, vous pouvez installer
Scrapy et ses dépendances depuis PyPI avec::

    pip install Scrapy

Nous vous recommandons fortement d'installer Scrapy dans :ref:`un virtualenv dédié <intro-using-virtualenv>`,
afin d'éviter tout conflit avec les paquets de votre système.

Notez que cela peut parfois obliger à résoudre des problèmes de compilation
pour certaines dépendances de Scrapy, selon votre système d'exploitation.
Pensez donc à consulter les :ref:`intro-install-platform-notes`.

Pour des instructions plus détaillées et propres à chaque plateforme, ainsi que
des informations de dépannage, poursuivez votre lecture.


Points utiles à savoir
----------------------

Scrapy est écrit en Python pur et dépend de quelques paquets Python essentiels (entre autres) :

* `lxml`_, un analyseur XML et HTML performant
* `parsel`_, une bibliothèque d'extraction de données HTML/XML écrite au-dessus de lxml
* `w3lib`_, un outil polyvalent pour manipuler les URL et les encodages des pages web
* `twisted`_, un framework réseau asynchrone
* `cryptography`_ et `pyOpenSSL`_, pour répondre à divers besoins de sécurité au niveau du réseau

Certains de ces paquets dépendent eux-mêmes de paquets qui ne sont pas écrits
en Python et qui peuvent nécessiter des étapes d'installation supplémentaires
selon votre plateforme.
Veuillez consulter les :ref:`guides spécifiques à chaque plateforme ci-dessous <intro-install-platform-notes>`.

En cas de problème lié à ces dépendances,
veuillez vous référer à leurs instructions d'installation respectives :

* `Installation de lxml`_
* :doc:`Installation de cryptography <cryptography:installation>`

.. _Installation de lxml: https://lxml.de/installation.html


.. _intro-using-virtualenv:

Utiliser un environnement virtuel (recommandé)
----------------------------------------------

En bref : nous recommandons d'installer Scrapy dans un environnement virtuel,
quelle que soit la plateforme.

Les paquets Python peuvent être installés soit de manière globale (c'est-à-dire
pour tout le système), soit dans l'espace utilisateur. Nous ne recommandons pas
d'installer Scrapy pour tout le système.

Nous vous recommandons plutôt d'installer Scrapy dans ce qu'on appelle un
« environnement virtuel » (:mod:`venv`). Les environnements virtuels vous
permettent d'éviter les conflits avec les paquets Python du système déjà
installés (qui pourraient casser certains de vos outils et scripts système), tout
en installant les paquets normalement avec ``pip`` (sans ``sudo`` ni outil
similaire).

Consultez :ref:`tut-venv` pour savoir comment créer votre environnement virtuel.

Une fois l'environnement virtuel créé, vous pouvez y installer Scrapy avec ``pip``,
comme n'importe quel autre paquet Python.
(Consultez les :ref:`guides spécifiques à chaque plateforme <intro-install-platform-notes>`
ci-dessous pour connaître les dépendances non-Python que vous pourriez devoir
installer au préalable).

.. _extras:

Extras optionnels
=================

Scrapy propose des :ref:`extras <pypug:dependency-specifiers-extras>` optionnels
qui installent des dépendances supplémentaires pour activer des fonctionnalités
précises. Pour installer Scrapy avec un ou plusieurs extras, listez-les entre
crochets :

.. code-block:: console

    pip install scrapy[s3,images]

Les extras suivants sont disponibles :

.. list-table::
   :header-rows: 1

   * - Extra
     - Fonctionnalité fournie
   * - ``bpython``
     - :ref:`shell bpython <shell-config>`
   * - ``color``
     - :setting:`LOG_COLOR`
   * - ``gcs``
     - :ref:`Google Cloud Storage <topics-feed-storage-gcs>` pour les
       :ref:`exports de feed <topics-feed-exports>` et les
       :ref:`pipelines de médias <media-pipeline-gcs>`
   * - ``httpx``
     - :ref:`httpx-handler`, y compris sa prise en charge de HTTP/2 et des
       proxys SOCKS
   * - ``images``
     - :ref:`Pipeline d'images <images-pipeline>`
   * - ``ipython``
     - :ref:`shell IPython <shell-config>`
   * - ``ptpython``
     - :ref:`shell ptpython <shell-config>`
   * - ``robotparser``
     - :ref:`Analyse de robots.txt avec Robotexclusionrulesparser <rerp-parser>`
   * - ``s3``
     - Stockage :ref:`Amazon S3 <topics-feed-storage-s3>` pour les
       :ref:`exports de feed <topics-feed-exports>`, les
       :ref:`pipelines de médias <media-pipelines-s3>` et les
       :ref:`téléchargements S3 <s3-handler>`
   * - ``twisted-http2``
     - :ref:`twisted-http2-handler`
   * - ``uvloop``
     - boucle d'événements `uvloop <https://github.com/MagicStack/uvloop>`_


.. _intro-install-platform-notes:

Notes d'installation selon la plateforme
========================================

.. _intro-install-windows:

Windows
-------

Bien qu'il soit possible d'installer Scrapy sous Windows avec pip, nous vous
recommandons d'installer `Anaconda`_ ou `Miniconda`_ et d'utiliser le paquet du
canal `conda-forge`_, ce qui évitera la plupart des problèmes d'installation.

Une fois `Anaconda`_ ou `Miniconda`_ installé, installez Scrapy avec::

  conda install -c conda-forge scrapy

Pour installer Scrapy sous Windows avec ``pip`` :

.. warning::
    Cette méthode d'installation nécessite « Microsoft Visual C++ » pour
    installer certaines dépendances de Scrapy, ce qui demande nettement plus
    d'espace disque qu'Anaconda.

#. Téléchargez et exécutez `Microsoft C++ Build Tools`_ pour installer le Visual Studio Installer.

#. Lancez le Visual Studio Installer.

#. Dans la section Workloads (Charges de travail), sélectionnez **C++ build tools**.

#. Vérifiez les détails de l'installation et assurez-vous que les paquets
   suivants sont sélectionnés comme composants facultatifs :

    * **MSVC**  (par ex. MSVC v142 - VS 2019 C++ x64/x86 build tools (v14.23) )

    * **Windows SDK**  (par ex. Windows 10 SDK (10.0.18362.0))

#. Installez les Visual Studio Build Tools.

Vous devriez maintenant pouvoir :ref:`installer Scrapy <intro-install-scrapy>` avec ``pip``.

.. _intro-install-ubuntu:

Ubuntu 14.04 ou supérieur
-------------------------

Scrapy est actuellement testé avec des versions suffisamment récentes de lxml,
twisted et pyOpenSSL, et il est compatible avec les distributions Ubuntu
récentes. Mais il devrait aussi fonctionner avec des versions plus anciennes
d'Ubuntu, comme Ubuntu 14.04, avec toutefois d'éventuels problèmes pour les
connexions TLS.

**N'utilisez pas** le paquet ``python-scrapy`` fourni par Ubuntu : il est en
général trop ancien et met du temps à rattraper la dernière version de Scrapy.


Pour installer Scrapy sur un système Ubuntu (ou basé sur Ubuntu), vous devez
installer ces dépendances::

    sudo apt-get install python3 python3-dev python3-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev

- ``python3-dev``, ``zlib1g-dev``, ``libxml2-dev`` et ``libxslt1-dev``
  sont nécessaires pour ``lxml``
- ``libssl-dev`` et ``libffi-dev`` sont nécessaires pour ``cryptography``

Dans un :ref:`virtualenv <intro-using-virtualenv>`,
vous pouvez ensuite installer Scrapy avec ``pip`` ::

    pip install scrapy

.. note::
    Les mêmes dépendances non-Python peuvent être utilisées pour installer
    Scrapy sous Debian Jessie (8.0) et versions ultérieures.


.. _intro-install-macos:

macOS
-----

La compilation des dépendances de Scrapy nécessite la présence d'un compilateur
C et des fichiers d'en-tête de développement. Sous macOS, ils sont
généralement fournis par les outils de développement Xcode d'Apple. Pour
installer les outils en ligne de commande de Xcode, ouvrez une fenêtre de
terminal et exécutez::

    xcode-select --install

Il existe un `problème connu <https://github.com/pypa/pip/issues/2468>`_ qui
empêche ``pip`` de mettre à jour les paquets du système. Il faut le contourner
pour installer correctement Scrapy et ses dépendances. Voici quelques solutions
proposées :

* *(Recommandé)* **N'utilisez pas** le Python du système. Installez une
  nouvelle version à jour, qui n'entre pas en conflit avec le reste de votre
  système. Voici comment faire avec le gestionnaire de paquets `homebrew`_ :

  * Installez `homebrew`_ en suivant les instructions de https://brew.sh/

  * Mettez à jour votre variable ``PATH`` pour indiquer que les paquets
    homebrew doivent être utilisés avant les paquets du système (remplacez
    ``.bashrc`` par ``.zshrc`` si vous utilisez `zsh`_ comme shell par
    défaut)::

      echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bashrc

  * Rechargez ``.bashrc`` pour vous assurer que les changements ont bien été pris en compte::

      source ~/.bashrc

  * Installez Python::

      brew install python

*   *(Facultatif)* :ref:`Installez Scrapy dans un environnement virtuel Python
    <intro-using-virtualenv>`.

  Cette méthode contourne le problème de macOS décrit ci-dessus, mais c'est
  aussi une bonne pratique générale pour gérer les dépendances, et elle peut
  compléter la première méthode.

Après l'une de ces solutions de contournement, vous devriez pouvoir installer Scrapy::

  pip install Scrapy


PyPy
----

Nous recommandons d'utiliser la dernière version de PyPy.
Pour PyPy3, seule l'installation sous Linux a été testée.

La plupart des dépendances de Scrapy disposent désormais de wheels binaires
pour CPython, mais pas pour PyPy. Cela signifie que ces dépendances seront
compilées pendant l'installation. Sous macOS, vous risquez de rencontrer un
problème lors de la compilation de la dépendance Cryptography. La solution à ce
problème est décrite `ici
<https://github.com/pyca/cryptography/issues/2692#issuecomment-272773481>`_ :
il faut exécuter ``brew install openssl`` puis exporter les options que cette
commande recommande (uniquement nécessaire pendant l'installation de Scrapy).
L'installation de Scrapy sous Linux ne pose pas de problème particulier, au-delà
de l'installation des dépendances de compilation. L'installation de Scrapy avec
PyPy sous Windows n'a pas été testée.

Vous pouvez vérifier que Scrapy est correctement installé en exécutant ``scrapy bench``.
Si cette commande affiche des erreurs telles que
``TypeError: ... got 2 unexpected keyword arguments``, cela signifie
que la dépendance ``PyPyDispatcher`` n'a pas été installée. Pour corriger ce
problème, exécutez ``pip install 'PyPyDispatcher>=2.1.0'``.


.. _intro-install-troubleshooting:

Dépannage
=========

AttributeError: 'module' object has no attribute 'OP_NO_TLSv1_1'
----------------------------------------------------------------

Après avoir installé ou mis à jour Scrapy, Twisted ou pyOpenSSL, vous pouvez
obtenir une exception avec la trace d'appels (traceback) suivante::

    […]
      File "[…]/site-packages/twisted/protocols/tls.py", line 63, in <module>
        from twisted.internet._sslverify import _setAcceptableProtocols
      File "[…]/site-packages/twisted/internet/_sslverify.py", line 38, in <module>
        TLSVersion.TLSv1_1: SSL.OP_NO_TLSv1_1,
    AttributeError: 'module' object has no attribute 'OP_NO_TLSv1_1'

Cette exception apparaît parce que votre système ou votre environnement
virtuel contient une version de pyOpenSSL que votre version de Twisted ne
prend pas en charge.

Pour installer une version de pyOpenSSL prise en charge par votre version de
Twisted, réinstallez Twisted avec l'option d'extra :code:`tls` ::

    pip install twisted[tls]

Pour plus de détails, consultez `Issue #2473 <https://github.com/scrapy/scrapy/issues/2473>`_.

.. _Python: https://www.python.org/
.. _lxml: https://lxml.de/index.html
.. _parsel: https://pypi.org/project/parsel/
.. _w3lib: https://pypi.org/project/w3lib/
.. _twisted: https://twisted.org/
.. _cryptography: https://cryptography.io/en/latest/
.. _pyOpenSSL: https://pypi.org/project/pyOpenSSL/
.. _setuptools: https://pypi.org/pypi/setuptools
.. _homebrew: https://brew.sh/
.. _zsh: https://www.zsh.org/
.. _Anaconda: https://www.anaconda.com/docs/main
.. _Miniconda: https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html
.. _Microsoft C++ Build Tools: https://visualstudio.microsoft.com/visual-cpp-build-tools/
.. _conda-forge: https://conda-forge.org/
