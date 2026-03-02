Lien officiel : https://guacamole.apache.org

  

Apache Guacamole est un bastion d’accès sans agent, accessible uniquement via un navigateur web.

  

Remarque : l’installation se fait sur Debian 12 car la dernière version de Guacamole ne supporte pas le paquet `freerdp3-dev` disponible sur Debian 13.

  

---

  

## A. Prérequis – dépendances (construction guacamole-server)

  

Mise à jour et installation des dépendances :

```bash

apt-get update

apt-get install build-essential libcairo2-dev libjpeg62-turbo-dev libpng-dev \

libtool-bin uuid-dev libossp-uuid-dev libavcodec-dev libavformat-dev libavutil-dev \

libswscale-dev freerdp2-dev libpango1.0-dev libssh2-1-dev libtelnet-dev \

libvncserver-dev libwebsockets-dev libpulse-dev libssl-dev libvorbis-dev libwebp-dev

```

  

### Rôle des dépendances

- **Compilation** : build-essential, libtool-bin

- **Graphismes** : cairo, jpeg, png, webp, pango

- **Multimédia** : avcodec, avformat, avutil, swscale

- **Identifiants** : uuid-dev, libossp-uuid-dev

- **Accès distant** : freerdp2-dev (RDP), libssh2 (SSH), libtelnet, libvncserver

- **Réseau** : libwebsockets (WebSockets)

- **Sécurité** : libssl (SSL/TLS)

- **Audio** : libpulse-dev, libvorbis-dev

  

---

  

## B. Installation guacamole-server (code source)

  

Téléchargement :

```bash

cd /tmp

wget https://downloads.apache.org/guacamole/1.6.0/source/guacamole-server-1.6.0.tar.gz

```

  

Décompression :

```bash

tar -xzf guacamole-server-1.6.0.tar.gz

cd guacamole-server-1.6.0/

```

  

Configuration :

```bash

./configure --with-systemd-dir=/etc/systemd/system/

```

  

- `configure` : vérifie les dépendances et prépare la compilation

- `--with-systemd-dir` : installe les services systemd dans `/etc/systemd/system`

  

Compilation et installation :

```bash

make

make install

ldconfig

```

  

Démarrage du service guacd :

```bash

systemctl daemon-reload

systemctl enable --now guacd

systemctl status guacd

```

  

### Rôle de guacd

`guacd` est le service principal. Il reçoit les requêtes de l’interface web et établit les connexions (RDP, SSH, VNC). Il agit comme intermédiaire entre le navigateur et les machines distantes.

  

---

  

## C. Application web (guacamole-client)

  

L’application web est déployée via Tomcat 9 (Tomcat 10 n’est pas compatible).

  

Ajout du dépôt Debian 11 pour Tomcat 9 :

```bash

nano /etc/apt/sources.list.d/bullseye.list

```

Contenu :

```

deb http://deb.debian.org/debian/ bullseye main

```

  

Mise à jour et installation :

```bash

apt-get update

apt-get install tomcat9 tomcat9-admin tomcat9-common tomcat9-user

```

  

Téléchargement de l’application web :

```bash

cd /tmp

wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-1.6.0.war

```

  

Déploiement :

```bash

mv guacamole-1.6.0.war /var/lib/tomcat9/webapps/guacamole.war

systemctl restart tomcat9 guacd

```

  

Accès :

```

http://<IP>:8080/guacamole/

```

  

---

  

## D. Base de données MariaDB (authentification)

  

Création des dossiers de configuration :

```bash

mkdir -p /etc/guacamole/{extensions,lib}

```

  

Installation MariaDB :

```bash

apt-get install mariadb-server

```

  

Connexion et création de la base :

```sql

mysql -u root -p

CREATE DATABASE guacamole_db;

CREATE USER 'guaca_cub'@'localhost' IDENTIFIED BY 'Cub_007';

GRANT SELECT,INSERT,UPDATE,DELETE ON guacamole_db.* TO 'guaca_cub'@'localhost';

FLUSH PRIVILEGES;

EXIT;

```

  

Extension JDBC (MySQL) :

```bash

cd /tmp

wget https://downloads.apache.org/guacamole/1.6.0/binary/guacamole-auth-jdbc-1.6.0.tar.gz

tar -xzf guacamole-auth-jdbc-1.6.0.tar.gz

mv guacamole-auth-jdbc-1.6.0/mysql/guacamole-auth-jdbc-mysql-1.6.0.jar /etc/guacamole/extensions/

```

  

Connecteur MySQL :

```bash

wget https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/9.5.0/mysql-connector-j-9.5.0.jar

cp mysql-connector-j-9.5.0.jar /etc/guacamole/lib/

```

  

Initialisation de la base :

```bash

cd guacamole-auth-jdbc-1.6.0/mysql/schema/

cat *.sql | mysql -u root -p guacamole_db

```

  

Configuration Guacamole :

```bash

nano /etc/guacamole/guacamole.properties

```

Contenu :

```

mysql-hostname: 127.0.0.1

mysql-port: 3306

mysql-database: guacamole_db

mysql-username: guaca_cub

mysql-password: Cub_007

```

  

Configuration guacd :

```bash

nano /etc/guacamole/guacd.conf

```

Contenu :

```

[server]

bind_host = 0.0.0.0

bind_port = 4822

```

  

Redémarrage :

```bash

systemctl restart tomcat9 guacd mariadb

```

  

---

  

## E. HTTPS (optionnel)

  

Création du certificat :

```bash

mkdir -p /etc/ssl/guacamole

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \

-keyout /etc/ssl/guacamole/guacamole.key \

-out /etc/ssl/guacamole/guacamole.crt

```

  

Permissions :

```bash

chown root:tomcat /etc/ssl/guacamole/guacamole.key

chmod 640 /etc/ssl/guacamole/guacamole.key

chmod 644 /etc/ssl/guacamole/guacamole.crt

```

  

Tomcat HTTPS :

```bash

nano /etc/tomcat9/server.xml

```

Ajouter :

```xml

<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"

maxThreads="150" SSLEnabled="true">

    <SSLHostConfig>

        <Certificate certificateFile="/etc/ssl/guacamole/guacamole.crt"

                     certificateKeyFile="/etc/ssl/guacamole/guacamole.key" />

    </SSLHostConfig>

</Connector>

```

  

Redémarrage :

```bash

systemctl restart tomcat9

```

  

Accès :

```

https://<IP>:8443/guacamole

```

  

---

  

## F. Dépannage RDP (exemple)

  

Vérifier le service :

```bash

systemctl status guacd

```

  

Erreur fréquente :

```

RDP server closed/refused connection: Security negotiation failed

```

  

Solutions possibles :

- Vérifier la configuration RDP de la machine distante

- Utiliser le bon type de sécurité (ex. NLA)

- Réinstaller ou mettre à jour freerdp2-dev

- Vérifier les logs :

```bash

sudo tail -f /var/log/syslog

```

  

---

  

## G. Récapitulatif

- guacd : service de connexion

- guacamole-client : interface web (Tomcat)

- MariaDB : authentification et stockage

- HTTPS : recommandé pour la production

  

---