# II. Configuration de base de Apache sous Debian

## 1. Les paramètres de base

Il y a deux versions d’Apache 2.4 mises à votre disposition MPM Worker et le MPM Prefork. Nous utiliserons le MPM Worker qui se base notamment sur des threads plutôt que sur des forks (copie d’un processus). En fonction des contextes, le choix ne sera pas systématiquement le même.

Voici une explication en détail des principaux paramètres contenus dans */etc/apache2/apache2.conf* :

* **ServerType standalone** .Le serveur s'exécutera seul, sans recourir au super-serveur xinetd. 

* **ServerRoot /etc/httpd** . Il s'agit du répertoire où le serveur trouvera son répertoire de configuration conf .On trouve dans /etc/httpd, un lien vers /var/log/httpd/access_log, le fichier-journal des accès aux ressources, réussis ou non (le consulter).

* **PidFile /var/run/httpd.pid**. C'est le fichier où le serveur en exécution stocke son premier numéro de processus (PID).

* **DocumentRoot /var/www/html**  fixe la racine du serveur Web, c'est-à-dire le répertoire de base où sont cherchées par défaut les pages html, lorsque l'URL ne comporte pas de chemin de répertoire.

* **User www-data  Group www-data**. Apache doit etre démarré par root, mais par sécurité ses processus auront pour propriétaire l'utilisateur www-data, sans privilège. 

* **ServerAdmin root@localhost**. S'il a un problème, le serveur écrit un message à cette adresse.

* **UserDir public_html**. Ce paramètre signifie que l'utilisateur toto peut publier ses pages WEB personnelles dans un sous-répertoire de son répertoire perso, qui doit être nommé public_html, c'est-à-dire dans /home/toto/ public_html. Sa page d'accueil sera alors accessible par l'URL : http://serveur/~toto , où serveur est le nom du serveur ou son adresse IP. 

* **DirectoryIndex index.html index.php index.htm** ... Il est courant d'omettre le nom du fichier de la page d'accueil d'un site ou de l'un de ses sous-répertoires. Pour ne pas retourner systématiquement une erreur 404 signalant une adresse erronnée, le serveur posséde une liste standard de noms de fichiers qu'il s'efforce de trouver dans le répertoire.  Cette liste ordonnée est indiquée par la clause DirectoryIndex.

* **AccessFileName .htaccess**. Cette clause fixe le nom du fichier à trouver dans un répertoire pour que son accès soit protégé, en imposant à l'utilisateur une authentification par nom et mot de passe. Ces comptes sont spécifiques à Apache et n'interfèrent pas avec les comptes Linux. Voir une annexe pour explication de sa mise en oeuvre. 

* **Timeout 300**. Paramètre important qui fixe le temps (en ms) d'attente maximum du serveur d'une réponse à une requete envoyée à un programme extérieur (comme un gestionnaire de base de données).

* **KeepAliveon, MaxKeepAliverequests, KeepAliveTimeout**. Autorise les connexions persistantes d'un client, afin de lui permettre l'envoi de plusieurs requetes sans déconnexion, avec un plafond fixé pour un client, pour servir aussi d'éventuels autres clients ! et un temps d'attente maxi de la requete suivante provenant du meme client. 

* **ServerName www**. Fixe un nouveau nom public pour le serveur, auquel on pourra s'adresser par les URL http://www/
Bien entendu le nom symbolique www doit être connu du DNS ou du fichier hosts local (sous GNU/Linux ou MS/Windows).

* **MinSpareServers, MaxSpareServers**. Nombres maximum et minimum de processus serveurs devant etre en permannence disponibles, en attente de nouvelles connexion clientes.

* **StartServers**. Nombre de processus serveurs démarrés à l'initialisation, en plus du processus père. Ceci explique pourquoi la requete ps aux|grep httpd renvoie 5 PID. 

* **MaxClients**. Nombre maximum de processus qu'Apache peut lancer et gérer simultanément. Ce nombre ne peut pas excéder 254 

* **MaxRequestsPerChild**. Nombre maximum de requetes HTTP traitées par un processus enfant avant qu'il ne soit éliminé.

* **ThreadsPerChild** définit le nombre de threads crées au sein de l’un de ces processus. La valeur de ce paramètre est limitée par la directive ThreadsLimit. 

## 2. Paramétrage d'un répertoire

Chaque répertoire auquel Apache accéde peut etre configuré particulièrement (ceci s'applique aussi à ses sous-répertoires). Le paramétrage de rep se précise dans un conteneur, ensemble de clauses situées entre les balises <Directory rep> et </Directory>. Attention, contrairement aux permissions Unix, les clauses s'appliquent **AUSSI à TOUS les sous-répertoires**. En revanche, une directive <Directory rep> spécifique à l'un des sous-répertoires s'impose. 

### 2.1 Restriction des accès 

Pour un répertoire donné, dans son conteneur **<Directory>**, on peut préciser les hôtes dont les requêtes seront traitées, et ceux dont les requêtes seront rejetées. Pour cela, il faut d'abord préciser une règle générale avec la directive require qui précise les règles à appliquer aux machines. 

**Exemple 1 :** On décide de bloquer l’accès du répertoire à tout le monde.
Require all denied

**Exemple 2 :** On autorise n’importe qui à accéder à un répertoire.
Require all granted

**Exemple 3 :** Seule l’adresse IPv4 78.10.0.1 aura accès au répertoire.
Require host 78.10.0.1

### 2.2 Paramétres d'Options

Ils permettent de controler l'action d'Apache sur les répertoires.

| **Option**     | **Signification**                                                                             |
|----------------|-----------------------------------------------------------------------------------------------|
| All - None     | toutes - aucune option(s) permise(s)                                                          |
| ExecCGI        | exécution de scripts autorisée                                                                |
| FollowSymLinks | le serveur suivra les liens symboliques rencontrés dans le répertoire                         |
| Includes       | permet l'utilisation de Server Side Includ                                                    |
| IncludesNOEXEC | permet l'utilisation de SSI sauf les directives #exec et #include                             |
| Indexes        | autorise l'affichage du contenu d'un répertoire (si un fichier par défaut n'y est pas trouvé) |

## 3. Configuration basique d'un site

```bash
sudoedit /etc/apache2/sites-available/lapastille
```

```
<VirtualHost  *:80>
    ServerName lapastille.st2siobp.local
    DocumentRoot /var/www/lapastille
    DirectoryIndex index.html
    <Directory "/var/www/lapastille">
          Options -Indexes
          AllowOverride All
          Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error-lapastille.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</Virtualhost>
```

1.	La directive **ServerName** permet de définir un nom pour l’hôte, il correspond en général au nom DNS FQDN du serveur.

2.	La directive **DocumentRoot** permet de définir la racine du site.

3.	La directive **DirectoryIndex** permet de définir quel sera l’index du site.

4.	La directive **Options -Indexes** permet d’interdire l’affichage du contenu du répertoire du site en cas d’absence d’un Index.

5.	**AllowOverride All** permet, lorsque le serveur trouve un fichier .htaccess (comme spécifié par la directive AccessFileName), de  savoir lesquelles des directives déclarées dans ce fichier peuvent remplacer des directives des fichiers de configuration du serveur.

6.	La directive **ErrorLog** permet de définir où placer les logs du site.

7.	La directive **LogLevel** permet de spécifier un niveau de sévérité de journalisation pour chaque module. Vous pouvez ainsi résoudre un problème propre à un module particulier en augmentant son volume de journalisation sans augmenter ce volume pour les autres modules.

8.	La directive **CustomLog** permet de définir un  fichier de log personnalisé. Dans le cas présent, le fichier access.log sera commun aux différents sites.



