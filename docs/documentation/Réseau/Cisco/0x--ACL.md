# Les ACL sur les équipements Cisco

## 1. Explications sur les ACL Cisco (access control list)

Les ACL permettent de filtrer les accès entre les différents réseaux ou de filtrer les accès au routeur lui même.
Les paramètres controlés sont:

* Adresse IP source
* Adresse IP de destination
* Protocole utilisé (ICMP, TPC, UDP, IPSEC)
* Numéros de port source et destination

Les ACL peuvent être appliquées sur le trafic entrant ou sortant. Il y a deux actions: 

* soit le trafic est interdit, 
* soit le trafic est autorisé. 

!!! Warning  "Attention"
    Les ACL sont prises en compte de façon séquentielle. Il faut donc placer les instructions les plus précises en premier et l'instruction la plus générique en dernier. Une fois une ACL générée, une règle implicite finale est appliquée et elle interdit tout le trafic.

Voici un schéma permettant de définir les différentes étapes lors de l'utilisation d'ACL sur une interface :

![](../../../media/Cisco/acl1.gif)
    
## 2. Différence entre les ACL standards et étendues

L'ACL standard filtre uniquement à partir des adresses IP sources. Elle est de la forme:

* _access-list numéro-de-la-liste {permit|deny} {host|source source-wildcard|any}_

Le numéro de l'ACL standard est compris entre 1 et 99.

L'ACL étendue filtre sur les adresses IP source et destination, sur le protocole et les numéros de port. Elle est de la forme:

* _access-list numéro de la liste {deny|permit} protocole source masque-source [operateur [port]] destination masque-destination [operateur [port]][established][log]_

Quelques opérateurs:
* eq : égal
* neq : différent
* gt : plus grand que
* lt : moins grand que

Le numéro de l'ACL étendue est compris entre 100 et 199.

Il est possible de nommer les ACL. Dans ce cas, on précisera dans la commande si ce sont des acls standards ou étendues.

```bash
(config)#ip access-list standard ACL_NAT
```

```bash
(config)#ip access-list extended ACL_VLAN_Admin
```

## 3.Notion de masque générique (wildcard mask)

Les ACL utilisent un masque permettant de sélectionner des plages d'adresses.

**Fonctionnement:**

En binaire, seuls les bits de l'adresse qui correspondent au bit à 0 du masque sont vérifiés. Par exemple, avec 172.16.2.0 0.0.255.255, la partie vérifiée par le routeur sera 172.16.

Sur le couple suivant: 0.0.0.0 0.0.0.0, toutes les adresses sont concernées (any). Sur ce couple: 192.168.2.3 255.255.255.255, on vérifie uniquement l'hote ayant l'IP 192.168.2.3 (host)

**Exemple** : Le masque 255.255.255.128 donnera comme wildcard mask 0.0.0.127

## 4. Comment appliquer les ACL ?

On crée l'ACL puis ensuite on applique l'ACL à une interface en entrée ou en sortie (in ou out).

Si l'ACL doit être mofifiée, il sera nécessaire de supprimer celle-ci puis de la recréer entièrement. Une façon pratique de faire est de conserver l'acl dans un fichier texte puis de faire un copier/coller.

```bash
(config)#int Gig0/0
(config-subif)#ip access-group IPv4_LAN in
```

## 5. Exemple de configuration

Création d'une entrée d'une access-list :

Dans l'exemple:

* On autorise la machine 192.168.2.12 à se connecter via ssh à toutes les machines du réseau 192.168.3.0/24,
* On autorise les réponses DNS en provenance de la machine 192.168.2.30,
* On autorise les paquets entrants pour les connexions tcp établies,
* Enfin on bloque et on journalise le reste du trafic.

```bash
R2(config)#ip access-list extended reseau-secretariat
R2(config-ext-nacl)#permit tcp host 192.168.2.12 gt 1023 192.168.3.0 0.0.0.255 eq 22
R2(config-ext-nacl)#permit udp host 192.168.2.30 eq 53 192.168.3.0 0.0.0.255 gt 1023
R2(config-ext-nacl)#permit tcp any any established
R2(config-ext-nacl)#deny ip any any log
```

Application de la liste d'accès à une interface :

```bash
R2(config)#int fa1/1
R2(config-if)#ip access-group reseau-secretariat in
R2(config-if)#
```

Affichage de la configuration de l'interface :

```bash
R2#sh run int fa1/1
Building configuration...

Current configuration : 136 bytes
!
interface FastEthernet1/1
ip address 192.168.3.2 255.255.255.0
ip access-group reseau-secretariat out
duplex auto
speed auto
end
```

Affichage de la liste de contrôle :

```bash
R2#show access-lists reseau-secretariat
Extended IP access list reseau-secretariat
10 permit tcp host 192.168.2.1 gt 1023 192.168.3.0 0.0.0.255 eq 22
20 permit tcp any any established
30 deny ip any any log
R2#
```

Suppression d'une acl :

```bash
R2(config)#no ip access-list extended reseau-secretariat
R2(config)#end
```

Suppression de l'association de la liste de contrôle à une interface :

```bash
R2(config)#int fa1/1
R2(config-if)#no ip access-group reseau-secretariat out
R2(config-if)#
```
