# IV. Protection et paramétrage d'un site par htaccess

## 1. Principes de fonctionnement

L'intérêt principal du .htaccess est de pouvoir configurer l'accès à une partie de l'arborescence d'un site sans avoir accès au fichier de configuration d'Apache. Ainsi, il sera possible de définir un certain nombre de paramétres directement en ajoutant ce fichier à la racine du répertoire concerné.

Il s'agit de réglementer l'accès à une arborescence réservée ou privée au sein d'un site WEB.

Seuls les utilisateurs dument authentifiés seront autorisés à y accéder après vérification de leur nom et mot de passe fournis. 

**Mécanisme :**

1.	vérification de la permission accordé au répertoire de pouvoir redéfinir localement les droits d'accès (par le contenu du fichier .htaccess. Cela est déterminé par la clause AllowOverride concernant le répertoire et présente dans le fichier de configuration commonhttpd.conf.

2.	s'il y a AllowOverride none (=interdiction d'outrepasser, de supplanter le présent paramétrage par le fichier htaccess), alors le fichier .htaccess sera totalement ignoré (c'est la valeur par défaut).

3.	s'il y a, pour ce répertoire, au moins AllowOverride AuthConfig, la demande d'authentification est déclenchée automatiquement lorsque le serveur WEB prendra en compte le contenu du fichier .htaccess présent et l'appliquera au répertoire courant, et à TOUS ses sous-répertoires éventuels.

4.	Les noms et mots de passe des utilisateurs autorisés ne sont pas présents dans .htaccess par souci de sécurité. Ce fichier contient une clause AuthUserFile qui indique au serveur où se trouve le fichier contenant noms et mots de passe. 

5.	Remarque : AccessFileName .htaccess est la clause qui fixe le nom global d'un tel fichier de paramétrage local à un répertoire. On pourrait donc en changer. 

## 2. Exemples de configuration avec htaccess

### 2.1 Exemple A

```bash
sudoedit /var/www/html/scan/.htaccess
```

```
AuthType Basic
AuthName "Acces restreint"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

### 2.2 Exemple B

```bash
sudoedit /var/www/html/commentcamarche/.htaccess
```

```
ErrorDocument 403 http://www.commentcamarche.net/accesrefuse.php3
AuthUserFile /repertoire/de/votre/fichier/.FichierDeMotDePasse
AuthGroupFile /dev/null
AuthName "Accès sécurisé au site CCM"
AuthType Basic
<LIMIT GET POST>
Require valid-user
</LIMIT>
```

