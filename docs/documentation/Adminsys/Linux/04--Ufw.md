# UFW sous Debian 13[^1]

## 1. Présentation générale

UFW (*Uncomplicated Firewall*) est une interface simplifiée pour configurer netfilter/iptables (et nftables sur les systèmes récents, dont Debian 13 où UFW s'appuie désormais sur le backend nftables). Son objectif est de masquer la complexité syntaxique d'iptables/nftables derrière des commandes lisibles, tout en générant en arrière-plan les règles de filtrage correspondantes.

**Installation sous Debian 13 :**

```
sudo apt update

sudo apt install ufw
```

## 2. Politique par défaut

La plupart des règles décrites supposent que vous utilisez le règlement UFW par défaut, configuré pour autoriser le trafic sortant et refuser le trafic entrant par défaut. 

```
sudo ufw default deny incoming

sudo ufw default allow outgoing
```

**Activation/désactivation du pare-feu :**

```
sudo ufw enable

sudo ufw disable
```

**Vérification de l'état des règles :**

```
sudo ufw status verbose
```

## 3. Règles de base (allow/deny/reject)

#### Bloquer une adresse IP

Pour bloquer toutes les connexions réseau qui proviennent d'une adresse IP spécifique, on exécute `sudo ufw deny from <IP>`. `from <IP>` spécifie une adresse IP source, et il est possible de spécifier un sous-réseau (ex: /24) à la place d'une IP unique. L'adresse IP source peut être utilisée dans n'importe quelle règle, y compris une règle allow.

#### Restriction par interface réseau

Pour bloquer les connexions d'une adresse IP spécifique vers une interface réseau donnée (par exemple eth0), on ajoute l'option `in on eth0` à la règle. L'interface réseau peut être spécifiée dans n'importe quelle règle de pare-feu, ce qui permet de limiter la portée de la règle à un réseau spécifique.

#### Exemples par service courant

| Service | Commande |
| - | - |
| SSH (port 22) | `sudo ufw allow ssh` ou `sudo ufw allow 22` |
| SSH depuis un sous-réseau | `sudo ufw allow from 192.168.1.0/24 to 192.168.1.254 port 22` |
| HTTP | `sudo ufw allow http` ou `sudo ufw allow 80` |
| HTTPS | `sudo ufw allow https` ou `sudo ufw allow 443` |
| HTTP+HTTPS en une règle | `sudo ufw allow proto tcp from any to 192.168.1.254 port 80,443` |
| MySQL sur interface réseau | `sudo ufw allow in on eth0 to 192.168.1.254 port 3306` |


Lorsque plusieurs ports sont spécifiés dans une même règle, il faut indiquer le protocole avec `proto tcp`.

#### Limitation du débit (anti brute-force)

Une fonctionnalité importante pour sécuriser SSH :

```
sudo ufw limit ssh
```

Cette règle bloque temporairement (via le module `recent` d'iptables/nftables) une IP qui tente plus de 6 connexions en 30 secondes sur le port spécifié.

## 4. Gestion des règles

```
sudo ufw status numbered      # afficher les règles numérotées

sudo ufw delete <numéro>      # supprimer une règle par son numéro

sudo ufw insert 1 allow from 10.0.0.5  # insérer une règle à une position précise

sudo ufw reset                # réinitialiser toutes les règles (rollback)
```

## 5. Logging

```
sudo ufw logging on

sudo ufw logging medium
```

Les journaux sont consultables via `journalctl` ou dans `/var/log/ufw.log`, utile pour le diagnostic et l'audit.

## 6. Filtrage entre réseaux : la commande `ufw route`

Par défaut, UFW filtre uniquement le trafic **destiné à la machine elle-même** (chaînes INPUT/OUTPUT). Lorsque la machine fait office de **routeur/passerelle** entre plusieurs réseaux (interfaces multiples, routage activé via `net.ipv4.ip_forward=1`), il faut filtrer le trafic qui **traverse** la machine — c'est le rôle de la chaîne FORWARD, géré par `ufw route`.

#### Syntaxe générale

```
sudo ufw route allow in on <iface_entrée> out on <iface_sortie> from <réseau_src> to <réseau_dst> port <port> proto <protocole>

sudo ufw route deny  ...

sudo ufw route reject ...
```

#### Cas d'usage typique

Soit un serveur Debian 13 avec deux interfaces :

- `eth0` : réseau LAN interne (192.168.1.0/24) 

- `eth1` : réseau DMZ (192.168.2.0/24) 

**Autoriser le LAN à accéder à un serveur web dans la DMZ :**

```
sudo ufw route allow in on eth0 out on eth1 from 192.168.1.0/24 to 192.168.2.10 port 80 proto tcp

Autre possibilité avec moins de précisions concernant les interfaces d’entrée et sorite:

sudo ufw route allow from 192.168.1.0/24 to 192.168.2.10 port 80 proto tcp
```

**Bloquer tout autre trafic entre les deux réseaux par défaut :**

```
sudo ufw default deny routed
```

(Cette politique « routed » est l'équivalent de la politique par défaut pour la chaîne FORWARD, à ne pas confondre avec `default deny incoming/outgoing` qui concernent le trafic destiné/émis par la machine elle-même.)

**Autoriser tout le trafic entre deux interfaces (cas simple type passerelle de confiance) :**

```
sudo ufw route allow in on eth0 out on eth1
```

#### Activation impérative du forwarding IP

`ufw route` ne sert à rien sans le routage IP activé au niveau noyau :

```
sudo nano /etc/sysctl.conf

# décommenter ou ajouter :

net.ipv4.ip_forward=1


sudo sysctl -p
```

#### Points importants

- `ufw route` agit sur la chaîne **FORWARD** de netfilter, distincte des chaînes INPUT/OUTPUT utilisées par les règles classiques (`ufw allow`/`ufw deny`). 

- Les règles sont évaluées dans l'ordre, comme les règles classiques (voir `ufw status numbered`). 

- Très utilisé pour : segmentation réseau (LAN/DMZ/invités), contrôle du trafic entre VLANs si la machine fait du routage inter-VLAN, ou filtrage du trafic d'une machine faisant office de passerelle NAT. 

## 7. Bonnes pratiques de mise en œuvre

1. Toujours configurer les règles **avant** d'activer UFW (`ufw enable`), en particulier la règle autorisant SSH, pour éviter de se couper l'accès à distance. 

2. Utiliser `ufw status verbose` régulièrement pour auditer les règles actives. 

3. Pour les machines passerelles, séparer clairement la logique « trafic vers la machine » (`allow`/`deny`) de la logique « trafic traversant la machine » (`route`). 

4. Sauvegarder/versionner les fichiers `/etc/ufw/*.rules` avant toute modification manuelle. 


[^1]: Source - Documentation produite par David Balny
