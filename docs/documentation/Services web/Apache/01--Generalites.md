# Généralités concernant Apache 2 sous Debian GNU/Linux

## 1. Installation d'un serveur LAMP (Linux/Apache/MySQL/PHP)

L’installation de Apache sous Debian GNU/Linux est relativement simple :

```bash
sudo aptitude install apache2
```

Si vous souhaitez mettre en œuvre des sites web dynamiques basés sur PHP7 et MySQL , vous devez également installer l’interpréteur PHP ainsi que le serveur MySQL et les librairies permettant de faire interagir ces applications entre elles :

```bash
sudo aptitude install php mariadb-server php-mysql libapache2-mod-php
```

Voilà le triptyque est maintenant prêt à fonctionner.

## 2. Arborescence du service Apache 2 sur Debian

Toute la configuration du service se réalise à partir du dossier /etc/apache2/ :

* Le fichier **apache2.conf** contient tous les éléments de configuration concernant le service apache2 globalement. Ce fichier est très rarement modifié sauf pour des besoins spécifiques.

* Le dossier **mods-enabled** contient les modules activés pour le service apache alors que mods-available contient tous les modules installés mais qui ne sont pas forcément actifs.

* Le fichier **ports.conf** permet de définir les ports TCP d’écoute du service Apache2. C’est ici que l’on retrouvera la directive Listen.

* Le répertoire **sites-enabled** contient la configuration des différents sites hébergés par Apache et qui sont actifs (par défaut *000-default* qui affiche la page It Works !) alors que le dossier **sites-available** contient la configuration de tous les sites mais pas forcément actifs.

* Le répertoire **conf.d** contient les éléments de configuration complémentaires ; en général des configurations globales au serveur.

* Le fichier **magic** permet de définir le comportement du serveur vis à vis des types de fichiers (.doc, .pdf….).

* Le fichier envvars sert à définir les variables d’environnement du serveur. Mis à part quelques rares cas précis vous n’aurez pas à modifier celui-ci.





