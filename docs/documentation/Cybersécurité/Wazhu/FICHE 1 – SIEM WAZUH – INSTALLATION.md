## Liens

- Lien officiel de l’installation :  

  https://documentation.wazuh.com/current/installation-guide/index.html

- Liens directs pour le dépannage :

  - Wazuh-dashboard : https://documentation.wazuh.com/current/user-manual/wazuh-dashboard/troubleshooting.html

  - Wazuh-agent : https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/troubleshooting.html

  
  Les exigences matérielles dépendent fortement du nombre de terminaux protégés et de charges de travail cloud. Ce nombre permet d'estimer la quantité de données à analyser et le nombre d'alertes de sécurité à stocker et indexer.

  

L’interface est en anglais. Un traducteur automatique du navigateur peut avoir été utilisé lors des captures d’écran.

  

---

  

## Sommaire

- A. Installation

  - A.1 Installation rapide

  - A.2 Installation des composants

    - A.2.1 Prérequis

    - A.2.2 Indexeur

    - A.2.3 Manager

    - A.2.4 Filebeat

    - A.2.5 Dashboard

  - A.3 Docker Compose

- B. Agents

  - B.1 Groupes

  - B.2 Installation agent

    - B.2.1 Windows

    - B.2.2 Linux

  - B.3 Gestion groupes

- C. Mise à jour

  - C.1 Serveur

  - C.2 Agents

  

---

  

## A. Installation

  

### A.1 Installation rapide

```bash

curl -sO https://packages.wazuh.com/4.11/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

```

Patientez : le déploiement peut prendre plusieurs minutes.

  

Une fois terminé :

```

User: admin

Password: <ADMIN_PASSWORD>

```

  

Activer l’indexeur au démarrage :

```bash

systemctl enable wazuh-indexer.service

```

  

Accès : https://<dashboard-ip>

  

Redémarrage éventuel :

```bash

systemctl restart wazuh-dashboard

systemctl restart wazuh-indexer

systemctl restart wazuh-manager

systemctl restart filebeat

```

  

Récupérer les mots de passe :

```bash

sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt

```

  

Désinstallation :

```bash

sudo bash wazuh-install.sh --uninstall

```

  

---

  

### A.2 Composants

Trois composants :

- indexeur

- manager

- dashboard

  

---

  

#### A.2.1 Prérequis indexeur

Certificats :

```bash

curl -sO https://packages.wazuh.com/4.9/wazuh-certs-tool.sh

curl -sO https://packages.wazuh.com/4.9/config.yml

```

Modifier config.yml avec l’IP du serveur.

  

Générer :

```bash

bash ./wazuh-certs-tool.sh -A

tar -cvf ./wazuh-certificates.tar -C ./wazuh-certificates/ .

```

  

Dépendances :

```bash

apt-get install debconf adduser procps

apt-get install gnupg apt-transport-https

```

  

Importer la clé :

```bash

curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \

gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import

chmod 644 /usr/share/keyrings/wazuh.gpg

```

  

Ajouter dépôt :

```bash

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \

| tee -a /etc/apt/sources.list.d/wazuh.list

apt update

```

  

---

  

#### A.2.2 Indexeur

Installation :

```bash

apt -y install wazuh-indexer

```

  

Configuration :

```

network.host: "192.168.2.1"

```

  

Démarrage :

```bash

systemctl enable wazuh-indexer

systemctl start wazuh-indexer

```

  

Test :

```bash

curl -k -u admin:admin https://192.168.2.1:9200

```

  

---

  

#### A.2.3 Manager

```bash

apt-get -y install wazuh-manager

systemctl enable wazuh-manager

systemctl start wazuh-manager

systemctl status wazuh-manager

```

  

---

  

#### A.2.4 Filebeat

Installation :

```bash

apt-get -y install filebeat

```

  

Configurer :

```

hosts: ["192.168.2.1:9200"]

```

  

Keystore :

```bash

filebeat keystore create

echo admin | filebeat keystore add username --stdin --force

echo admin | filebeat keystore add password --stdin --force

```

  

Démarrage :

```bash

systemctl enable filebeat

systemctl start filebeat

filebeat test output

```

  

---

  

#### A.2.5 Dashboard

Installation :

```bash

apt-get install debhelper tar curl libcap2-bin

apt-get -y install wazuh-dashboard

```

  

Configurer :

```

server.host: 192.168.2.1

opensearch.hosts: https://192.168.2.1:9200

```

  

Démarrage :

```bash

systemctl enable wazuh-dashboard

systemctl start wazuh-dashboard

```

  

---

  

### A.3 Docker Compose

Prérequis : installation de Docker Compose.

  

Cloner :

```bash

git clone https://github.com/wazuh/wazuh-docker.git -b v4.10.1

```

  

Certificats :

```bash

docker-compose -f generate-indexer-certs.yml run --rm generator

```

  

Démarrer :

```bash

docker-compose up -d

```

  

Identifiants par défaut :

- user : admin

- password : SecretPassword

  

---

  

## B. Agents

  

### B.1 Groupes

Dans l’interface :

Server management → Endpoint Groups → Add New group

  

---

  

### B.2 Installation agent

L’agent envoie des données chiffrées au serveur.

  

#### Windows

Prérequis :

- droits administrateur

- PowerShell 3.0+

  

Commande :

```powershell

Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.11.1-1.msi \

-OutFile $env:tmp\wazuh-agent;

msiexec.exe /i $env:tmp\wazuh-agent /q \

WAZUH_MANAGER='192.168.2.1' \

WAZUH_AGENT_GROUP='serveurs' \

WAZUH_AGENT_NAME='CubAD'

```

  

Démarrer :

```powershell

NET START WazuhSvc

```

  

Logs :

```

C:\Program Files (x86)\ossec-agent\ossec.log

```

  

---

  

#### Linux

Prérequis : root

  

Installation :

```bash

wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.10.1-1_amd64.deb

sudo WAZUH_MANAGER='192.168.2.1' \

WAZUH_AGENT_GROUP='serveurs' \

WAZUH_AGENT_NAME='CubDHCP' \

dpkg -i ./wazuh-agent_4.10.1-1_amd64.deb

```

  

Démarrer :

```bash

systemctl daemon-reload

systemctl enable wazuh-agent

systemctl start wazuh-agent

```

  

Logs :

```bash

sudo tail -f /var/ossec/logs/ossec.log

```

  

---

  

### B.3 Gestion groupes

Ajouter :

```bash

/var/ossec/bin/agent_groups -q -a -i 001 -g Serveurs_Linux

```

  

Retirer :

```bash

/var/ossec/bin/agent_groups -q -r -i 001 -g default

```

  

---

  

## C. Mise à jour

  

### C.1 Serveur

```bash

sudo apt-get update

sudo apt-get full-upgrade

```

  

Vérifications :

```bash

systemctl status wazuh-indexer

systemctl status wazuh-dashboard

/var/ossec/bin/wazuh-control info

```

  

---

  

### C.2 Agents

  

#### Ubuntu/Debian

```bash

wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.11.1-1_amd64.deb

sudo dpkg -i ./wazuh-agent_4.11.1-1_amd64.deb

sudo systemctl daemon-reload

sudo systemctl restart wazuh-agent

```

  

---

  

#### Windows

Version :

```

C:\Program Files (x86)\ossec-agent\VERSION

```

  

Installer la dernière version MSI puis redémarrer le service.