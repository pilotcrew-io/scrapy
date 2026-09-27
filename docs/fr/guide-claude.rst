============================================
Utiliser Scrapy avec Claude (guide débutant)
============================================

.. note::
   Ce guide n'existe pas dans la documentation officielle de `Scrapy <https://scrapy.org>`_. Il est écrit par PilotCrew, spécifiquement pour une personne qui ne sait pas (ou peu) programmer et qui veut utiliser Scrapy en se faisant aider par un assistant IA comme Claude. C'est le point de départ recommandé si c'est votre cas : commencez ici, pas par le :doc:`tutoriel technique <intro/tutorial>`.

.. contents:: Sommaire
   :local:
   :depth: 1

À qui s'adresse ce guide ?
===========================

Vous voulez récupérer automatiquement des informations sur un site web (des prix, des annonces, des articles, des horaires...) et les avoir dans un fichier bien rangé (Excel, CSV, JSON). C'est exactement ce que fait Scrapy. Le problème : Scrapy s'utilise en écrivant du code Python, et vous n'avez peut-être jamais programmé.

C'est là que Claude entre en jeu. Vous n'avez pas besoin d'apprendre Python : vous décrivez en français ce que vous voulez récupérer, et vous demandez à Claude d'écrire le code Scrapy correspondant. Vous restez aux commandes (c'est vous qui décidez quoi récupérer et où), Claude fait la partie technique.

Ce guide explique comment travailler à deux avec Claude sur un projet Scrapy : quoi installer, comment formuler vos demandes, comment réagir quand ça ne marche pas du premier coup, et les quelques règles de politesse à respecter envers les sites que vous visitez.

Ce dont vous avez besoin
=========================

* **Python installé sur votre ordinateur.** Si ce n'est pas déjà fait, suivez la page :doc:`intro/install` (elle explique comment faire, étape par étape, sur Windows, Mac et Linux).
* **Un terminal** (l'application en ligne de commande de votre ordinateur : Terminal sur Mac/Linux, PowerShell ou l'invite de commandes sur Windows). C'est l'endroit où vous tapez des commandes et où vous demandez à Claude Code de travailler pour vous.
* **Claude**, sous une des deux formes suivantes :

  * `Claude Code <https://claude.com/claude-code>`_ : un assistant qui travaille directement dans votre terminal, peut créer des fichiers, écrire du code et lancer des commandes à votre place. C'est la manière la plus confortable de suivre ce guide, car Claude peut tout faire pour vous du début à la fin.
  * `Claude.ai <https://claude.ai>`_ dans un navigateur : Claude peut aussi simplement vous écrire le code à copier-coller vous-même dans vos fichiers et votre terminal. Un peu plus manuel, mais ça fonctionne très bien aussi.

Aucune connaissance de Python n'est requise. Savoir ouvrir un terminal et copier-coller des commandes suffit.

Étape 1 : installer Scrapy
============================

Ouvrez votre terminal et installez Scrapy avec cette commande (voir :doc:`intro/install` si elle échoue) :

.. code-block:: bash

   pip install scrapy

Vous pouvez aussi simplement demander à Claude Code : « installe Scrapy sur mon ordinateur », il exécutera la commande et vérifiera que tout s'est bien passé.

Étape 2 : créer un projet avec Claude
=======================================

Un projet Scrapy est un dossier avec une structure bien précise (fichiers de configuration, dossier pour vos "spiders", etc.). Plutôt que de la créer à la main, demandez à Claude de le faire.

**Exemple de demande à donner à Claude Code :**

.. code-block:: text

   Crée un nouveau projet Scrapy nommé "mon_projet" dans le dossier courant.

Claude exécute la commande ``scrapy startproject mon_projet`` et vous avez un projet prêt à l'emploi. Si vous utilisez Claude.ai sans Claude Code, demandez-lui simplement la commande à taper, puis copiez-la dans votre terminal.

Étape 3 : décrire à Claude ce que vous voulez récupérer
==========================================================

C'est l'étape la plus importante, et celle où la qualité de votre demande change tout. Un "spider" est le morceau de code qui sait aller sur un site, trouver les informations, et les extraire. Plus vous donnez de détails précis à Claude, mieux il écrira ce spider du premier coup.

Une bonne demande contient en général :

1. **L'URL de départ** (la page par laquelle commencer).
2. **La ou les données à récupérer** pour chaque élément de la page (par exemple : titre, prix, image, lien).
3. **Comment on reconnaît un élément** sur la page si vous le savez (par exemple : "chaque produit est dans une carte avec la classe ``product-card``") — mais si vous ne le savez pas, ce n'est pas grave, Claude peut regarder la page lui-même.
4. **S'il faut suivre des liens** (par exemple : aller sur la page de chaque produit pour avoir plus de détails, ou passer à la page suivante d'une liste).
5. **Le format de sortie souhaité** (CSV, JSON, Excel...).

Exemples de demandes prêtes à adapter
---------------------------------------

**Récupérer une liste simple (une seule page) :**

.. code-block:: text

   Dans mon projet Scrapy "mon_projet", crée un spider nommé "produits" qui va sur
   https://exemple-boutique.fr/catalogue et récupère, pour chaque produit affiché,
   le titre, le prix et le lien vers la fiche produit. Exporte le résultat dans un
   fichier produits.csv.

**Suivre la pagination (plusieurs pages) :**

.. code-block:: text

   Fais en sorte que le spider "produits" suive aussi le lien "page suivante" en bas
   de la liste, pour récupérer tous les produits du catalogue, pas seulement la
   première page.

**Aller chercher plus de détails sur chaque fiche :**

.. code-block:: text

   Pour chaque produit du spider "produits", va aussi sur sa page de détail et
   récupère en plus la description complète et la disponibilité en stock.

**Vous ne connaissez pas la structure du site :**

.. code-block:: text

   Voici l'URL d'une page d'annonces immobilières : https://exemple-immo.fr/annonces.
   Regarde la page et propose-moi une liste des informations qu'on pourrait en
   extraire (prix, surface, ville, nombre de pièces...), puis écris un spider
   Scrapy qui les récupère toutes.

Claude Code peut ouvrir la page lui-même pour repérer la structure du HTML avant d'écrire le spider ; avec Claude.ai, collez un extrait du code source de la page (clic droit → « Afficher le code source ») pour l'aider.

Étape 4 : lancer le spider et regarder le résultat
=====================================================

Une fois le spider écrit, on le lance depuis le dossier du projet :

.. code-block:: bash

   scrapy crawl produits -o produits.csv

``produits`` est le nom du spider, ``-o produits.csv`` dit à Scrapy d'enregistrer le résultat dans ce fichier (voir :doc:`topics/feed-exports` pour les autres formats possibles : JSON, XML...). Demandez simplement à Claude Code : « lance le spider produits et enregistre le résultat en CSV », il exécutera la commande et pourra vous résumer ce qu'il a trouvé.

Ouvrez ensuite le fichier avec Excel, LibreOffice Calc ou un éditeur de texte pour vérifier que les données sont correctes.

Étape 5 : quand ça ne marche pas du premier coup
====================================================

C'est normal, et c'est même la situation la plus fréquente : les sites web ont des structures compliquées, et un premier essai récupère rarement tout parfaitement. La bonne nouvelle, c'est que corriger un spider avec Claude est en général très rapide. Décrivez simplement ce qui ne va pas.

**Le fichier de sortie est vide ou incomplet :**

.. code-block:: text

   Le spider produits n'a récupéré aucun résultat (le fichier produits.csv est vide).
   Voici ce que la commande a affiché dans le terminal : [collez le message ici].
   Peux-tu regarder pourquoi et corriger le spider ?

**Une information est fausse ou manquante :**

.. code-block:: text

   Dans produits.csv, la colonne "prix" est souvent vide alors que le prix est bien
   visible sur le site. Peux-tu vérifier le sélecteur utilisé pour le prix et le
   corriger ?

**Le site semble différent de ce que Claude attendait :**

.. code-block:: text

   Le site a changé depuis la dernière fois : les produits ne sont plus dans des
   balises <div class="product-card"> mais ailleurs. Peux-tu regarder à nouveau
   la page https://exemple-boutique.fr/catalogue et adapter le spider ?

Dans tous les cas, copiez-collez le message d'erreur exact affiché dans le terminal : c'est l'information la plus utile pour Claude. Le fait de savoir *lire* un message d'erreur n'est pas nécessaire de votre côté, Claude s'en charge.

Étape 6 : exporter proprement vos données
=============================================

Scrapy sait exporter directement en CSV, JSON, JSON Lines ou XML avec l'option ``-o`` montrée plus haut. Pour des besoins plus précis (nom de fichier avec la date, plusieurs formats à la fois, envoi vers un espace de stockage en ligne...), demandez à Claude :

.. code-block:: text

   Configure mon projet pour que chaque exécution du spider produits crée
   automatiquement un fichier "produits-AAAA-MM-JJ.json" avec la date du jour.

Voir :doc:`topics/feed-exports` pour tout ce que Scrapy sait faire à l'export.

Étape 7 : les bonnes manières envers les sites que vous visitez
====================================================================

Un spider mal réglé peut envoyer beaucoup de requêtes très vite à un site, ce qui peut le ralentir ou déclencher un blocage de votre adresse IP. Quelques règles simples, à faire respecter en le demandant à Claude :

* **Respectez le fichier robots.txt du site**, qui indique ce que le site autorise ou non à un robot. Scrapy le fait automatiquement par défaut (paramètre ``ROBOTSTXT_OBEY``).
* **Ne partez pas trop vite** : demandez à Claude d'ajouter un délai entre les requêtes (``DOWNLOAD_DELAY``) si vous scrapez beaucoup de pages.
* **Vérifiez les conditions d'utilisation du site** avant de récupérer ses données, en particulier si vous comptez les republier ou les utiliser commercialement.
* **Ne récupérez pas de données personnelles** (noms, emails, numéros de téléphone de particuliers) sans réfléchir à ce que vous allez en faire : le RGPD s'applique aussi aux données que vous collectez vous-même sur le web.

**Exemple de demande :**

.. code-block:: text

   Configure mon projet Scrapy pour respecter robots.txt et attendre au moins
   1 seconde entre deux requêtes, pour ne pas surcharger le site.

Plus de détails dans :doc:`topics/practices`.

Pièges fréquents (et comment les décrire à Claude)
======================================================

**Le contenu n'apparaît pas dans le code source de la page.**
   Le site charge ses données avec du JavaScript après le chargement initial (souvent le cas des sites modernes). Dites-le à Claude : « le contenu que je veux semble chargé en JavaScript, il n'est pas dans le code source de base ». Claude pourra proposer une solution adaptée (regarder les requêtes réseau du site, ou utiliser un outil de rendu JavaScript).

**Le site demande de se connecter (login) avant d'afficher les données.**
   Précisez-le dès le départ : « il faut se connecter avec un compte avant de voir la liste ». C'est un cas plus avancé, dites-le explicitement pour que Claude adapte son approche (et ne lui donnez jamais votre mot de passe en clair dans la conversation : demandez-lui plutôt comment le stocker de façon sûre).

**Le site vous bloque après quelques requêtes.**
   Décrivez le symptôme précisément (page d'erreur, code 403, CAPTCHA qui apparaît...) et demandez à Claude de ralentir le spider ou d'ajuster les en-têtes de requête (``User-Agent``).

**Il y a une pagination, un "voir plus", ou un défilement infini.**
   Dites à Claude comment la liste continue sur le site (numéro de page dans l'URL, bouton "page suivante", chargement au scroll) : c'est souvent la partie la plus délicate à deviner seul.

Pour aller plus loin
========================

Une fois à l'aise, ces pages de référence (traduites, mais plus techniques) permettent de comprendre ce que Claude a écrit pour vous, ou d'affiner vous-même :

* :doc:`topics/spiders` : comment fonctionne un spider en détail.
* :doc:`topics/selectors` : comment Scrapy repère les éléments dans une page.
* :doc:`topics/items` : comment structurer les données récupérées.
* :doc:`topics/settings` : tous les réglages disponibles (vitesse, robots.txt, etc.).
* :doc:`topics/shell` : un outil pour tester l'extraction d'une page en direct.

Vous n'avez besoin de les lire que si vous êtes curieux ou curieuse : pour un usage courant avec Claude, ce guide suffit.
