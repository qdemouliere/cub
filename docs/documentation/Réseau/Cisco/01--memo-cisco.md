# Mémo des commandes Cisco

## 1. Configurations générales

### 1.1 Commandes de base

**Passer du mode utilisateur au mode privilège**

```
Router> enable

Router#
```

**Passer du mode privilège au mode configuration**

```
Router# configure terminal

Router(config)#
```

**Changer le nom de l'équipement**

```
Router(config)# hostname Rw-B315-qd
```

**Empêcher les recherches DNS indésirables**

```
Router(config)# no ip domain-lookup
```

**Mettre un mot de passe pour le passage en mode privilège en clair ou haché**

```
Router(config)# enable password monmotdepasse 

Router(config)# enable secret monmotdepasse 
```

**Mettre un mot de passe pour les accès par le câble console**

```
Router(config)# line console 0
Router(config-line)# password rootroot
Router(config-line)# login
```

**Mettre un mot de passe pour l’accès Telnet avec 5 sessions simultanées**

```
Router(config)# line vty 0 4
Router(config-line)# transport input telnet
Router(config-line)# password rootroot
Router(config-line)# login
```

**Mettre en place un message du jour**

```
Router(config)# banner motd # Welcome to available users #
```

**Chiffrer l’ensemble des mots de passe présents dans le fichier de configuration du routeur via le chiffre de Vigenère**

```
Router(config)# service password-encryption
```

**Définir une taille minimum pour les mots de passe utilisés sur le système (ici 10 caractères)**

```
Router(config)# security passwords min-length 10
```

Lorsque vous souhaitez annuler une commande tapée précédemment il suffit de la retaper en rajoutant un **no** devant.


**Sauvegarde de la configuration en cours vers la configuration lancée au démarrage**

```
Router# copy running-config startup-config

Autre possibilité plus rapide :

Router# write
```

**Sauvegarde de la configuration vers un serveur TFTP**

```
Router# copy startup-config tftp
```

**Voir la configuration de l'équipement**

```
Router# show running-config ou show startup-config
```

### 1.2 Mise en place d'un accès distant SSH

**Création d’un nouvel utilisateur**

```
Router(config)# username adminsio privilege 15 password azerty2QWERTY
```

L’option privilege permet de définir un niveau d’autorisation pour l’utilisateur créé. Ce niveau va de 0 à 15. 0 fournit un niveau d’habilitation minimum alors que 15 donne un accès au mode privilégié directement.


**Définition du domaine DNS (même si il n’existe pas)**

```
Router(config)#  ip domain-name sio.local
```

**Génération des clefs RSA SSH et activation du protocole + options de time out et du maximum d'authentifications**

```
Router(config)#  crypto key generate rsa
Router(config)#  ip ssh time-out 60
Router(config)#  ip ssh authentication-retries 2
```

**Forcer l’utilisation de SSH à la place de Telnet**

```
Router(config)#  line vty 0 4
Router(config-line)# transport input SSH
Router(config-line)# login local
```

**N’autoriser qu’un réseau ou une machine à pouvoir se connecter en SSH**

```
Router(config)#  ip access-list standard 10
Routeur(config-ext-nacl)# permit 192.0.2.0 0.0.0.255
```

```
Router(config)#  line vty 0 4
Router(config-line)# access-class 10 in
```

**Informations concernant SSH sur le routeur**

```
Router# show ip ssh
Router# show ssh
```

## 2. Configurations spécifiques au commutateur

### 2.1 Configurations globales

**Observer la table de commutation (correspondance port - adresse MAC)**

```
Switch# show mac-address-table
```

**Affecter une adresse IP au switch**

```
Switch(config)# interface vlan 1
Switch(config-if)# ip address 192.0.2.10 255.255.255.0
Switch(config-if)# no shutdown*
```

**Définir une configuration réseau sur un commutateur en fonction d’un VLAN**

```
Switch(config)# interface vlan 1
Switch(config-if)# no shutdown
Switch(config-if)# ip address 192.0.2.1 255.255.255.0
```

**Définir une passerelle par défaut**

```
Switch(config)# ip default-gateway 192.0.2.254
```


**Définir un serveur DNS résolveur**

```
Switch(config)# ip name-server 192.0.2.253
```

### 2.2 Configurer des VLAN et mettre en oeuvre le protocole 802.1q

**Création d’un VLAN**

```
Switch(config)# vlan 10
Switch(config-vlan)# name informatique
Switch(config-vlan)# exit
```

**Affectation d’un port du commutateur dans un VLAN (mode access)**

```
Switch(config)# interface FastEthernet 0/24
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit
```

**Affectation d’un groupe de ports dans un VLAN (mode access)**

```
Switch(config)# interface range FastEthernet 0/20-24
Switch(config-range-if)# switchport mode access
Switch(config-range-if)# switchport access vlan 10
Switch(config-range-if)# exit
```

**Configuration d’un port permettant de faire transiter des trames étiquetées 802.1q appartenant à plusieurs VLAN**

```
Switch(config)# interface FastEthernet 0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan all ou numéro ou plage-de-numéro
Switch(config-if)# exit
```

Il est également possible d’affecter une rangée de ports en trunk avec la commande **interface range** comme dans l’exemple précédent pour les ports en mode access.

??? Warning "Attention"
    **Attention !** Dans le cas de la configuration d’un commutateur de niveau 3 **Cisco 3750**, il est nécessaire de taper une commande supplémentaire avec la commande *switchport mode trunk*.

```
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
```

**Définir un VLAN natif sur un port configuré en mode trunk/tag**

```
Switch(config)# interface FastEthernet 0/24
Switch(config-if)# switchport mode trunk native vlan <1-4094>
```

**Visualisation des informations liées aux VLAN sur le commutateur**

```
Switch# show vlan
Switch# show interfaces switchport
Switch# show interfaces trunk
Switch# show ip interface
```

### 2.2 Configuration du mode port-security sur un port du commutateur


**Activation de port-security**

```
Switch(config)# interface GigabitEthernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
```

**Définition des adresses MAC autorisées**

```
Switch(config)# interface FastEthernet 0/1
Switch(config-if)# switchport port-security maximum 10
```

**Dynamiquement**

```
Switch(config-if)# switchport port-security mac-address sticky
```

**De façon statique**

```
Switch(config-if)# switchport port-security mac-address 0000.0000.0003
```

**Définition du mode de violation**

```
Switch(config-if)# switchport port-security violation {protect | restrict | shutdown\}
```

- **Mode protect :** dès que la “violation” est constatée, le port arrête de transférer le trafic des adresses non autorisées sans envoyer de message de log.

- **Mode restrict :** dès que la “violation” est constatée, le port arrête de transférer le trafic des adresses non autorisées et transmet un message de log.

- **Mode shutdown :** dès que la “violation” est constatée, le port passe en état err-disabled (shutdown) et un message de log est envoyé.

## 3. Paramétrages spécifiques d'un routeur

### 3.1 Configuration des interfaces réseaux d’un routeur

??? Warning "Attention"
    **Attention !** Par défaut toutes les interfaces du routeur sont éteintes, il faut donc les activer pendant le paramétrage.


**Configuration d’une interface Gigabit Ethernet (le nom change pour 10 ou 100M)**

```
Router(config)# interface GigabitEthernet 0/0
Router(config-sub-if)# no shutdown
Router(config-sub-if)# ip address 192.168.1.254 255.255.255.0 
Router(config-sub-if)# ipv6 address FE80::1 link-local
Router(config-sub-if)# ipv6 address 2001:DB8:FFFF:B::1/64
Router(config-sub-if)# description Passerelle sur le LAN
Router(config-sub-if)# exit
```

**Configuration d’une interface Série**


Pour l’interface série, il faut penser à mettre en place un clock rate sur l’interface DCE (femelle) considéré comme un modem grâce à la commande clock rate suivi du débit (Desired clock rate, in bits per second: 1200, 2400, 4800, 9600, 19200, 38400, 56000, 64000, 72000, 125000, 148000, 250000, 500000, 800000, 1000000, 1300000, 2000000, 4000000, or 8000000).

```
Router(config)# interface Serial 1/0
Router(config-if)# no shutdown
Router(config-if)# ip address 172.16.0.254 255.255.255.252
Router(config-if)# clock rate 250000
```

**Configuration d’une sous-interface pour le routage inter-VLAN**

```
Router(config)# interface GigabitEthernet 0/0
Router(config-if)# no shutdown
Router(config-if)# exit
Router(config)# interface GigabitEthernet 0/0.10
Router(config-sub-if)# encapsulation dot1Q 10
Router(config-sub-if)# ip address 192.168.10.254 255.255.255.0
Router(config-sub-if)# description Interface VLAN 10 Informatique
Router(config-sub-if)# exit
```

!!! info "Important"
    **Important :** Lorsque l’on met en place des sous-interfaces pour du routage inter-VLAN sur un routeur, il est fortement conseillé de ne pas affecter d’adresse IP à l’interface physique mais de penser à bien l’activer. Ensuite si vous souhaitez mettre en place du NAT ou des ACL, il faut affecter ses dernières aux sous-interfaces créées et non à l’interface physique.


**Afficher les informations concernant les interfaces**

```
Router# show ip interface brief
```

### 3.2 Configuration du routage statique sur un routeur


**Activation du routage**

```
Router(config)# ip routing
Router(config)# ipv6 unicast-routing (pour le protocole IPv6)
```

**Créer une route**

??? Warning "Attention"
    **Attention !** Sur le matériel Cisco, les réseaux directement connectés au routeur n’ont pas besoin d’être déclarés avec la commande ip route.

```
Router(config)# ip route destination masque passerelle 
Router(config)# ipv6 route destination masque passerelle (en IPv6)
```

Exemple :

Router(config)# ip route 10.0.0.0 255.0.0.0 172.16.0.253
Router(config)# ipv6 route 2001:DB8:FFFF:C::/64 2001:DB8:FFFF:B::FFFF


**Mise en place d’une route par défaut**

```
Router(config)# ip route 0.0.0.0 0.0.0.0 172.16.0.253
Router(config)# ipv6 route :: 2001:DB8:FFFF:B::FFFF (en IPv6)
```

**Afficher les informations concernant les routes**

```
Router# show ip route
```

### 3.3 Configuration du routage dynamique avec RIP v2 sur un routeur

```
Router(config)# router rip
Router(config-router)# version 2
```

**On indique les réseaux directement connectés au routeur**

```
Router(config-router)# network 192.168.1.0
```

**On indique les interfaces qui ne sont pas concernées par les mises à jour RIP**

```
Router(config-router)# passive-interface FastEthernet 0/0
```

**Désactivation de l’auto-gestion des masques avec les classes par le protocole RIP**

```
Router(config-router)#no auto-summary
```

**Redistribution d’une route statique**

```
Router(config-router)# redistribute static
```

**Propagation d’une route par défaut**

```
Router(config-router)# default-information originate
```

**Passer en mode verbeux pour observer le fonctionnement du protocole RIP**

```
Router# debug ip rip
```

**Annuler le mode verbeux**

```
Router# undebug all
```

### 3.4 Mise en place de la NAT sur un routeur Cisco

**I. Désigner les interfaces Inside et Outside (Interne et Externe)**

```
Router(config)# interface type number
Router (config-if)# ip nat inside
Router (config)# interface type number
Router (config-if)# ip nat outside
```

**II. Définir les adresses locales soumises au NAT dans l’optique des les masquer pour qu’elles puissent avoir accès à l’extérieur et que l’extérieur ne voit que l’adresse du routeur.**

```
Router(config)# access-list access-list_number permit source_ip wildcard_mask
Ou
Router(config)# ip access-list standard access-list_number 
```

Puis ajoutez les réseaux autorisés à l'aide de la commande permit. Le terme wildcard_mask désigne un masque inversé, pour un masque de sous-réseau de 255.255.255.0 le wildcard mask sera 0.0.0.255.

Exemple : access-list 1 permit 10.0.0.0 0.255.255.255

**IV. Mise en place de la translation LAN vers IP publique**

```
Router(config)# ip nat inside source list numero-acl interface int-publique overload
```

Exemple : **ip nat inside source list 1 interface GigabitEthernet 0/1 overload**

**V. Définir une règle de redirection de port (port forwarding)**

```
(config)# ip nat inside source static type_protocol address_privée port address_publique port
```

**VI. Vérifier que la translation est effective sur l’équipement**

```
Router# show ip nat translations
```

### 3.5 Configuration de HSRP sur des routeurs Cisco


**Configuration du routeur A**

```
RouterA(config)# int f0/1
RouterA(config-if)# ip address 192.0.2.2 255.255.255.0
RouterA(config-if)# no shutdown
RouterA(config-if)# standby 10 ip 192.0.2.1
RouterA(config-if)# standby 10 priority 150
RouterA(config-if)# standby 10 preempt
```

**Configuration du routeur B**

```
RouterB(config)# int f0/1*
RouterB(config-if)# ip address 192.0.2.3 255.255.255.0
RouterB(config-if)# no shutdown
RouterB(config-if)# standby 10 ip 192.0.2.1
RouterB(config-if)# standby 10 priority 110
```

La valeur de priorité la plus haute définit le routeur actif (maître). La valeur par défaut est fixée à 100. La préemption (preempt) permet de s’assurer que le routeur actif reprendra son rôle dès que la situation sera revenue à la normale en cas de bascule sur le routeur secondaire en mode failover.


**Vérifications HSRP**

```
RouterA# show standby
RouterA# show standby neighbors
RouterA# show standby brief
```

## 4. Réinitialisation des équipements

### 4.1 Réinitialisation d’un routeur Cisco 


La première méthode consiste à sauvegarder la startup-config initiale sur votre serveur TFTP puis de la restaurer à la fin du TP sur le routeur ou le commutateur.

La seconde méthode est la suivante :


```
Router#configure terminal
Router(config)# config-register 0x2102
Router(config)# end
Router# write erase
Router# reload
System configuration has been modified. Save? \[yes/no\]: n
Proceed with reload? \[confirm\]
```

### 4.2 Démarrer en mode rommon et restaurer les paramètres d’usine 


Dès le démarrage Ctrl-Pause sous Hyperterminal ou Ctrl-A puis F sous minicom. Vous arrivez en mode rommon1>, tapez :

```
rommon1> confreg 0x2142 (il vous reste à redémarrer manuellement l’équipement).

```

### 4.3 Autoriser la sauvegarde des modifications en startup-config

Il est possible que votre routeur ou commutateur soit en mode lecture seule. Pour le vérifier, taper la commande show version et regarder si la configuration du registre est 0x2142. Si c’est le cas, il faut la modifier en 0x2102.

```
Router#configure terminal
Router(config)#config-register 0x2102
Router(config)# do write
Router(config)#do reload
```
ou en mode rommon

```
rommon1\> confreg 0x2102 (il vous reste à redémarrer l’équipement).

```


### 4.4 Réinitialiser un commutateur  Cisco 2960 SI


Pour réinitialiser le commutateur : 

Appuyez sur le bouton Mode et maintenez-le enfoncé. Après environ 3 secondes, les DELs du commutateur commencent à clignoter. Continuez de maintenir le bouton Mode enfoncé. Les DELs arrêtent de clignoter après 7 secondes supplémentaires et le commutateur redémarre ensuite.

Soit si vous êtes déjà connecté en console :


```
switch# delete vlan.dat
Delete flash:/vlan.dat? [confirm]*
switch# erase startup-config
switch# reload
System configuration has been modified. Save? [yes/no]: n
Proceed with reload? [confirm]
```




