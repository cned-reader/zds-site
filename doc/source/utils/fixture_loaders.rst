===========================================
Le chargement de jeux de données (fixtures)
===========================================

Zeste de savoir étant un projet très complet il est nécessaire de pouvoir charger un ensemble de jeux de données
de manière automatique pour créer une nouvelle instance à des fins de test ou pour forker le site.

Pour faciliter la tâche, trois outils sont mis à disposition des développeurs, testeurs et utilisateurs.

Les données sérialisables pour une base fonctionnelle
-----------------------------------------------------

Un premier ensemble de données simples est accessible par la commande intégrée à django ``python manage.py loaddata``.

Cette commande s'attend à une liste de fichier au format yaml et supporte les *wildcards*.
Nous possédons un ensemble de données sérialisées dans le dossier fixtures:

- ``categories.yaml`` : contient le chargement de 3 catégories de tutoriels, 2 sous catégories de tutoriels rangées dans les bonnes catégories parentes
- ``forums.yaml`` : contient le chargment de 4 catégories de forum et de 10 forums dans ces catégories
- ``licences.yaml`` : contient le chargement de 2 licences dont la licence par défaut (tous droit réservés)
- ``mps.yaml`` : **nécessite le chargement des users**, contient la création d'un MP d'un membre à un admin
- ``topics.yaml``: **nécessite le chargement des users**, contient la création de plusieurs topics dans les forums dont un résolu
- ``users.yaml``: Crée 6 utilisateurs:
    - admin/admin avec les droits d'administration
    - staff/admin faisant partie du groupe staff
    - user/user un utilisateur normal et sans problème
    - ïtrema/ïtrema un utilisateur normal, sans problème mais qui aime l'utf8
    - Anonymous/anonymous : le compte d'anonymisation
    - External/external: le compte pour accueillir les cours externes des auteurs ne voulant pas devenir membre ou quittant le site

De ce fait, le moyen le plus simple de charger l'ensemble des données de base est ``python manage.py loaddata fixtures/*.yaml``.

Les données complexes voire les scénarios
-----------------------------------------

Certaines données, pour exister, ont besoin de ressources supplémentaires qui ne sont pas forcément sérialisables.
Par exemple un tutoriel a besoin d'un dépôt git, une option d'aide (ZEP 03) a besoin d'une icône qui sera accessible depuis
le web...

Ces données ont besoin d'être traitées par une routine avant d'être créées. Ces routines existent déjà dans les objets
appelés Factory qui sont dans chacun des fichiers ``factories.py`` de tous les modules.

Pour utiliser ces fabriques d'objet, vous aurez une nouvelle fois recours au format yaml afin de décrire les
fabriques que vous désirez utiliser ainsi que les paramètres à envoyer auxdites fabriques.

Le format du fichier est celui-ci:

.. sourcecode:: yaml

    -   factory: zds.module.factories.YourFactory
        fields:
            champ_string: "valeur1"
            champ_int: 0

    -   factory: zds.utils.factories.YourFactory
        fields:
            champ_string: "valeur2"
            champ_int: 1

Les fichiers de factory déjà existant sont rangés dans le dossier ``fixtures/advanced``.

Pour utiliser un fichier yaml de factory, il vous suffit de lancer la commande ``python manage.py load_factory_data chemin_vers_vos_fichier.yaml``.
Cette méthode est compatible avec les *wildcards*.

Pour utiliser les factories, il vous faudra vous référer à la documentation de ces dernières puisque les champs associés peuvent
être de deux types :

- les champs de base qui sont aussi présents avec la même orthographe dans le modèle de données
- les champs personnalisés qui sont faits pour indiquer des comportements complémentaires à la commande
  par exemple, avec la zds.utils.HelpWrittingFactory, utiliser ``fixture_image_path`` vous permettra de renseigner le chemin relatif de l'image dans le dossier ``fixtures`` plutôt que le chemin absolu.

Bien que ce module soit optionnel, si vous désirez qu'il soit possible de demander de l'aide sur les tutoriels et articles, 
il vous faudra utiliser ``python manage.py load_factory_data fixtures/advanced/aide_tuto_media.yaml``.

Tester sur un jeu de données massif
-----------------------------------

.. attention::
    L'utilisation de la commande qui suit peut prendre du temps

Afin de tester avec un jeu de données qui se rapproche le plus possible de ce qui peut se trouver en exploitation, et aussi
trouver une variété suffisante pour être confiant en vos tests, nous avons développé une commande qui génère une immense
quantité de données.

Pour l'utiliser il suffit de lancer ``python manage.py load_fixtures size=SIZE type=LIST_OF_TYPE``.

.. note::
    Vous pouvez remplacer ``size`` par ``sizes``, ``taille`` ou ``level``.
    Vous pouvez remplacer ``type`` par ``types``.

Les types à charger sont en fait les modèles de données qui seront créés.

Chaque modèle de données aura son propre *coefficient de création* c'est à dire le nombre d'éléments qui seront créés de base.
Ce coefficient sera à multiplier par le *coefficient de taille* dirrigé par :

- size=low : *coefficient de taille* = 1
- size=medium: *coefficient de taille* = 2
- size=high: *coefficient de taille* = 3

+---------------------------------+-----------------------------------+-----------------------------+
|Type                             | Modèles créés                     | *coefficient de création*   |
+=================================+===================================+=============================+
|member                           |Profile (simple membres)           |10                           |
+---------------------------------+-----------------------------------+-----------------------------+
|staff                            |Profile (avec droit de staff)      |3                            |
+---------------------------------+-----------------------------------+-----------------------------+
|gallery                          |Gallery/UserGallery(au hasard)     |3 (par user)                 |
|                                 +-----------------------------------+-----------------------------+
|                                 |Image                              |5 (par gallery)              |
+---------------------------------+-----------------------------------+-----------------------------+
|category_forum                   |forum.Category                     |8                            |
+---------------------------------+-----------------------------------+-----------------------------+
|category_content                 |Licence                            |"CB-BY", "CC-BY-ND",         |
|                                 |                                   |"CC-BY-ND-SA", "CC-BY-SA",   |
|                                 |                                   |"CC","CC-BY-IO","Tout-Droits"|
|                                 +-----------------------------------+-----------------------------+
|                                 |utils.Category                     |5                            |
|                                 +-----------------------------------+-----------------------------+
|                                 |utils.SubCategory                  |10                           |
+---------------------------------+-----------------------------------+-----------------------------+
|forum                            |utils.Forum                        |8                            |
+---------------------------------+-----------------------------------+-----------------------------+
|tag                              |Tag                                |50                           |
+---------------------------------+-----------------------------------+-----------------------------+
|topic                            |Topic (dont sticky et locked)      |20                           |
+---------------------------------+-----------------------------------+-----------------------------+
|post                             |Post                               |10(en moyenne par topic)     |
+---------------------------------+-----------------------------------+-----------------------------+
|article                          |Article(40% en validation, 20% pub)|10                           |
+---------------------------------+-----------------------------------+-----------------------------+
|reaction                         |Reaction                           |20(en moyenne par article    |
+---------------------------------+-----------------------------------+-----------------------------+
|tutorial                         |Tutorial                           |10                           |
|                                 |40% en validation, 30% pub         |                             |
|                                 |Principalement des bigtuto         |                             |
+---------------------------------+-----------------------------------+-----------------------------+
|note                             |Note                               |20(en moyenne par tutoriel)  |
+---------------------------------+-----------------------------------+-----------------------------+
