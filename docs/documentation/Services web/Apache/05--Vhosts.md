# V. Les Virtual Hosts sous Apache

## 1. Introduction

Le principe des Virtual Hosts sous Apache est de pouvoir héberger plusieurs sites web distincts accessibles pour des ip, des ports, ou des nom de domaine différents.

Il existe donc 3 types de VHosts : 

* Par nom de domaine (les plus utilisés).
* Par adresses IP + ports (ex: un premier site en 192.168.1.10:80 et un second en 192.168.1.10:8080)
* Par adresses IP distinctes sur le même serveur (ex: un premier site en 192.168.1.10 et un second en 192.168.1.11)

Au niveau de la société CUB, les VHosts par nom de domaine seront exclusivement mis en oeuvre.

## 2. Exemples de configuration

Dans cet exemple, nous allons créer 2 VHosts par nom de domaine. Un premier site qui correspondra à une forge logicielle et qui sera accessible par le nom de domaine FQDN forge.btssio.org. Puis un deuxième site qui correspondra à un site vitrine et qui sera accessible par le nom de domaine FQDN prod.btssio.org.

```bash
sudoedit /etc/apache2/sites-available/forge.conf
```

```
<VirtualHost *:80>
        ServerName forge.btssio.org
        ServerAlias forge.btssio.org
        ServerAdmin webmaster@btssio.org
        ErrorLog /var/log/apache2/forge.btssio.org-error_log
        CustomLog /var/log/apache2/forge.btssio.org-access_log
        DocumentRoot "/var/www/html/forge/"
        <Directory "/var/www/html/forge/">
                Options Indexes FollowSymLinks
                AllowOverride All
                Require all granted
        </Directory>
</VirtualHost>
```

```bash
sudoedit /etc/apache2/sites-available/prod.conf
```

```
<VirtualHost *:80>
         ServerName prod.btssio.org
        ServerAlias prod.btssio.org
        ServerAdmin webmaster@btssio.org
        ErrorLog /var/log/apache2/prod.btssio.org-error_log
        CustomLog /var/log/apache2/prod.btssio.org-access_log
        DocumentRoot "/var/www/html/prod/"
        <Directory "/var/www/html/prod/">
                Options Indexes FollowSymLinks
                AllowOverride All
                Require all granted
        </Directory>
</VirtualHost>
```

!!! Warning  "Attention"
    Une fois les 2 fichiers de configuration des Virtual Hosts créés dans le répertoire sites-available, il est indispensable de les rendre actifs via la commande a2ensite.

```bash
sudo a2ensite forge
sudo a2ensite prod
```
## 3. Configuration des enregistrements dans le serveur DNS maître faisant autorité

Pour que la distinction des sites par nom de domaine puisse se faire, il est indispensable que des enregistrements soient ajoutés dans votre zone DNS.

Reprenons l'exemple précédent. Nous disposons d'un serveur web 192.168.1.10 ayant pour nom de domaine FQDN www.btssio.org. Ce serveur héberge donc les 2 VHOSTS forge.btssio.org et prod.btssio.org.

Après s'être connecté sur le serveur DNS faisant autorité, il est nécessaire d'éditer le fichier de zone (Attention au numéro de série !), puis de mettre en place les éléments suivants :

```bash
sudoedit /var/cache/bind/db.btssio.org
```

```
...
www     IN A       192.168.1.10
forge   IN CNAME    www.btssio.org.
prod    IN CNAME    www.btssio.org
```

```bash
sudo systemctl restart bind9
```

## 4. Conclusion

Grâce notamment aux en-têtes HTTP, votre serveur Apache sera maintenant en mesure de rediriger le client web (le navigateur) vers le bon site.

!!! Warning  "Attention"
    Les différents caches (DNS, de navigateur) peuvent induire des erreurs. Ainsi lors de vos tests, assurez-vous que les caches soient correctement réinitialisés.

