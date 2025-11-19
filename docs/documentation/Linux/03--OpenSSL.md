# OpenSSL sous Debian GNU/Linux

## 1. Introduction

OpenSSL est une boîte à outils de chiffrement comportant deux bibliothèques, libcrypto et libssl, fournissant respectivement une implémentation des algorithmes cryptographiques et du protocole de communication SSL/TLS, ainsi qu'une interface en ligne de commande, openssl.

Développée en C, OpenSSL est disponible sur les principaux systèmes d'exploitation et dispose de nombreux wrappers ce qui la rend utilisable dans une grande variété de langages informatiques. En 2014, deux tiers des sites Web l'utilisaient.[^1]

## 2. Création et utilisation d’un certificat X509 auto-signé et d'une clé privée sur un serveur Web Apache 2

### 2.1 Exemple avec l'algorithme de chiffrement RSA

Voici un exemple de génération d'une clé privée et d'un certificat X509 auto-signé sur Debian avec l'algorithme de chiffrement RSA. Pour rappel, RSA est de plus en plus considéré comme un algorithme cryptographique déprécié. Les algorithmes à courbes elliptiques sont à privilégier.

```bash
etudiant@web0:/etc/apache2$ sudo mkdir certs
etudiant@web0:/etc/apache2$ cd certs/
etudiant@web0:/etc/apache2/certs$ sudo openssl req -newkey rsa:4096 -keyout docs.key -x509 -days 365 -out docs.crt
Generating a RSA private key
..........................................................++++
.++++
writing new private key to 'docs.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:FR
State or Province Name (full name) [Some-State]:Indre et Loire
Locality Name (eg, city) []:Tours
Organization Name (eg, company) [Internet Widgits Pty Ltd]:ScopTi
Organizational Unit Name (eg, section) []:Service Informatique
Common Name (e.g. server FQDN or YOUR name) []:docs.sisr.sioplc.fr
Email Address []:postmaster@sisr.sioplc.fr
```

!!! Warning  "Attention"
    Tout d'abord, veillez à bien comprendre précisemment ce que fait la commande openssl et les informations demandées par la suite. Enfin soyez vigilant avec la passphrase exigée pour sécuriser l'accès à la clé privée. Bien que ce paramétrage renforce nettement la sécurité en cas de compromission de cette clé, cela peut devenir un point de blocage sur des VM ou des serveurs dédiés sur lesquels vous n'avez pas accès à la séquence de démarrage de Debian. En effet, la passphrase nécessaire pour déverrouiller la clé privée est demandé lors de cette séquence, bloquant ainsi votre VM ou votre serveur tant qu'elle n'a pas été rentrée par l'administrateur.

### 2.2 Exemple avec un algorithme de chiffrement à courbe elliptique

```bash
etudiant@web0:/etc/apache2$ cd certs/
etudiant@web0:/etc/apache2$ openssl ecparam -name prime256v1 -genkey -noout -out docs.key
etudiant@web0:/etc/apache2$ openssl req -new -x509 -key docs.key -out docs.crt -days 360
```

## 3. Mise en place d’une PKI (autorité de certification) interne pour la gestion des certificats avec OpenSSL

**Construction de l’arborescence de l’autorité :**

```bash
etudiant@pki:/etc/apache2/certs$ cd /etc/ssl
etudiant@pki:/etc/ssl$ sudo mkdir cubpki
etudiant@pki:/etc/ssl$ cd cubpki/
etudiant@pki:/etc/ssl/cubpki$ sudo mkdir certs crl newcerts private
etudiant@pki:/etc/ssl/cubpki$ sudo sh -c "echo '01' > serial"
etudiant@pki:/etc/ssl/cubpki$ sudo touch index.txt
etudiant@pki:/etc/ssl/cubpki$ sudo cp /usr/lib/ssl/openssl.cnf .
```

**Adapter le fichier /etc/ssl/cubpki/openssl.cnf à vos besoins :**

```
[ ca ]
default_ca	= CA_default		# The default ca section

####################################################################
[ CA_default ]

dir		= /etc/ssl/cubpki		# Where everything is kept
certs		= $dir/certs		# Where the issued certs are kept
crl_dir	= $dir/crl		# Where the issued crl are kept
database	= $dir/index.txt	# database index file.
#unique_subject	= no			# Set to 'no' to allow creation of
new_certs_dir   = $dir/newcerts         # default place for new certs.

certificate     = $dir/scopti-ac.crt      # The CA certificate
serial          	= $dir/serial           		# The current serial number
crlnumber    = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
crl             	= $dir/crl.pem          # The current CRL
private_key   = $dir/private/cubpki.key # The private key

x509_extensions = usr_cert              # The extensions to add to the cert
```

**Initialisation de la clé privée de l’AC :**

```bash
etudiant@scopti-ac:/etc/ssl/scopti-AC$ sudo openssl genrsa -aes256 -out private/cubpki.key 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
.......................................................................................................................................................++++
......++++
e is 65537 (0x010001)
Enter pass phrase for private/cubpki.key:
Verifying - Enter pass phrase for private/cubpki.key:
```

La commande suivante permet de générer une clé privée RSA de 4096 bits protégée par une passphrase chiffrée avec l’algorithme de chiffrement symétrique AES 256. Pour le mot de passe, rentrez etudiant_007 (ce mot de passe protège votre clef privée, il est donc indispensable !)

**Mise en place de la clé publique de l’AC :**

```bash
etudiant@pki:/etc/ssl/cubpki$ sudo openssl req -new -x509 -nodes -sha256 -days 365 -key private/cubpki.key -out cubpki.crt
Enter pass phrase for private/cubpki.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:FR
State or Province Name (full name) [Some-State]:Indre et Loire
Locality Name (eg, city) []:Tours
Organization Name (eg, company) [Internet Widgits Pty Ltd]:ScopTi
Organizational Unit Name (eg, section) []:Autorite de certification
Common Name (e.g. server FQDN or YOUR name) []:ac.sisr.sioplc.fr
Email Address []:postmaster@sisr.sioplc.fr
```

La commande suivante permet de générer le nouveau certificat de l’autorité de certification valable 1 an au format X509 qui sera signé et basé sur la clef privée générée précédemment.

Les informations indiquées permettront aux personnes récupérant le certificat d’en savoir plus sur votre identité en tant que AC. Il est possible de diffuser ce certificat notamment sur votre site web.

## 4. Génération et validation d’une demande de certificat depuis le serveur Web 

**Mise en place d’une clé privée sur le serveur web :**

```bash
etudiant@www:~* cd /etc/apache2/
etudiant@www:/etc/apache2$ sudo mkdir certs
etudiant@www:/etc/apache2/certs$ sudo openssl genrsa -aes256 -out docssisr.key 4096
Generating RSA private key, 4096 bit long modulus (2 primes)
.......................................................++++
.....................................................................................................++++
e is 65537 (0x010001)
Enter pass phrase for docssisr.key:
Verifying - Enter pass phrase for docssisr.key:
```

**Génération de la demande de certificat sur le serveur web :**

```bash
etudiant@www:/etc/apache2/certs$ sudo openssl req -new -key docssisr.key -out docssisr.csr
Enter pass phrase for docssisr.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:FR
State or Province Name (full name) [Some-State]:Indre et Loire
Locality Name (eg, city) []:Tours
Organization Name (eg, company) [Internet Widgits Pty Ltd]:ScopTi
Organizational Unit Name (eg, section) []:Service Informatique
Common Name (e.g. server FQDN or YOUR name) []:docs.sisr.sioplc.fr
Email Address []:postmaster@sisr.sioplc.fr

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:etudiant_007
An optional company name []:
```

**Signature de la demande et production du certificat par l ‘AC sur le serveur PKI :**

Vous devez copier la demande de certificat (.csr) vers l’AC afin que celle-ci la signe et génère un certificat signé (.crt). 

```bash
etudiant@pki:/etc/ssl/cubpki$ sudo openssl ca -config ./openssl.cnf -policy policy_anything -out docssisr.crt -infiles docssisr.csr 
Using configuration from ./openssl.cnf
Enter pass phrase for /etc/ssl/cubpki/private/cubpki.key:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Nov  5 15:25:10 2021 GMT
            Not After : Nov  5 15:25:10 2022 GMT
        Subject:
            countryName               = FR
            stateOrProvinceName       = Indre et Loire
            localityName              = Tours
            organizationName          = ScopTi
            organizationalUnitName    = Service Informatique
            commonName                = docs.sisr.sioplc.fr
            emailAddress              = postmaster@sisr.sioplc.fr
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                F5:AF:DF:5A:F6:34:8F:4D:1C:2B:47:37:E8:C3:94:6C:ED:72:5E:93
            X509v3 Authority Key Identifier: 
                keyid:77:44:5C:3B:C1:A1:F7:18:96:18:7B:82:E5:6C:99:8C:DD:F4:25:AD

Certificate is to be certified until Nov  5 15:25:10 2022 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

## 4. Pour aller plus loin

Pour ceux qui voudraient aller plus loin et comprendre les différents formats et types de fichier utilisés (p7, p10, p12, der, pem, key, crt, cer) par des clés privées et des certificats X509, je vous renvoie à cet excellent article complet :

* [https://www.arsouyes.org/articles/2021/2021-06-21_PKCS_pem_der_key_crt/](https://www.arsouyes.org/articles/2021/2021-06-21_PKCS_pem_der_key_crt/).


[^1]: Source - Wikipédia