# VI. Activation de TLS avec Apache 2.4

## 1. Introduction

Apache supporte bien évidemment la mise en place du protocole HTTPS et l'activation de SSL/TLS. Pour rappel, TLS permet d'assurer la confidentialité, l'intégrité, et l'authentification.

* La confidentialité est assurée par le chiffrement des connexions entre les clients web et le serveur web.
* Le contrôle d'intégrité est inclus au protocole TLS par l'usage de fonctions de hachage telles que SHA-256.
* L'authentification se fait à l'aide des certificats X509 et de l'empreinte unique de la clé publique qu'ils contiennent.

## 2. Configuration d'Apache 2

### 2.1 Certificats X509 et clés privées

Au préalable, le répertoire /etc/apache2/certs doit disposer de 3 fichiers indispensables pour activer le protocole https :

1. La clé privée du serveur web (ex: docssisr.key).
2. Le certificat X509 signé par l’autorité de certification interne (ex: docssisr.crt).
3. Le certificat X509 de l’autorité de certification interne ou publique (cubpki.crt) pour la vérification de la signature et de la chaîne de confiance.

!!! Warning  "Attention"
    Il est également possible de n'avoir que **2 fichiers**, ce qui est le cas par exemple lorsque que vous utilisez l'autorité de certification publique Let's Encrypt. Dans ce cas précis, vous aurez un premier fichier qui contient le certificat X509 du serveur signé par l'autorité + le ou les certificats des autorités de certification associées à ce celui-ci (ex: fullchain.pem). Puis le fichier contenant la clé privée (ex: privkey.pem).

```bash
etudiant@www:/etc/apache2/certs$ ls
docssisr.crt  docssisr.key  scopti-ac.crt
```

## 3. Configuration du serveur Apache2

Voici une configuration type d’Apache2 pour HTTPS :

```bash
sudoedit sites-available/docs.conf
```

```
<VirtualHost *:443>
    ServerAdmin postmaster@sisr.sioplc.fr
    ServerName docs.sisr.sioplc.fr

    DocumentRoot /var/www/html/docs

    SSLEngine on
    # Si vous ne disposez que de deux fichiers SSLCACertificateFile est inutile
    SSLCertificateFile /etc/apache2/certs/docssisr.crt
    SSLCertificateKeyFile /etc/apache2/certs/docssisr.key
    SSLCACertificateFile /etc/apache2/certs/scopti-ac.crt 
    SSLOptions StrictRequire
    SSLProtocol TLSv1.3
    SSLCipherSuite      ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256
    SSLHonorCipherOrder on
    SSLCompression      off

    <Directory /var/www/html/docs/>
        Options +Indexes
        AuthType basic
        AuthName "Acces protege"
        AuthBasicProvider file
        AuthUserFile "/etc/apache2/passwd"
        Require valid-user
    </Directory>
</VirtualHost>
```

Il faut également penser à activer le module SSL au niveau du service Apache2 puis le recharger.

```bash
sudo a2enmod ssl
Considering dependency setenvif for ssl:
Module setenvif already enabled
Considering dependency mime for ssl:
Module mime already enabled
Considering dependency socache_shmcb for ssl:
Enabling module socache_shmcb.
Enabling module ssl.
See /usr/share/doc/apache2/README.Debian.gz on how to configure SSL and create self-signed certificates.
```

```bash
sudo service apache2 reload
```

Voilà le résultat dans un navigateur web :

![](../../media/apache/https_browser.png)
