# VI. Supervision des équipements réseaux à l'aide du protocole SNMPv3

## a) Supervision active avec SNMP v3 sur un équipement Cisco

**Définition de la location de l’équipement et d’une adresse mail de contact**

```bash
SwL3-HKCUB(config)#snmp-server location Agence HK CUB
SwL3-HKCUB(config)#snmp-server contact postmaster@hongkong.cub.sioplc.fr
```

**Définir les droits en lecture-écriture et l’accès à la MIB (vue) de l’équipement**

```bash 
SwL3-HKCUB(config)#snmp-server view ReadOnly-View iso included
```

**Création d’un groupe spécifique avec l’application des droits établis lors de la commande précédente**

```bash
SwL3-HKCUB(config)#snmp-server group ReadOnly v3 priv read ReadOnly-View snmp-service
```

**Création d’un utilisateur et d’une authentification HMAC-SHA1, chiffrement de la connexion à l’aide d’un secret et d'AES**

```bash
SwL3-HKCUB(config)#snmp-server user zabbix ReadOnly v3 auth sha etudiant_007 priv aes 128 azerty2QWERTY access snmp-service
```

## b) Configuration d’un hôte sur Zabbix à l’aide du protocole SNMPv3

**Valider la connexion SNMP depuis le serveur Zabbix vers l’équipement en ligne de commande**

Installation des outils snmp
```bash
root@zabbix:~# apt install snmp
```

Test de la connectivité à l’équipement réseau
```bash
root@zabbix:~# snmpstatus -v3 -l authPriv -a SHA -A etudiant_007 -x AES -X azerty2QWERTY -u zabbix 192.168.8.126
```

Récupération d’une valeur précise (ici le modèle du switch et la version du firmware) à l’aide d’un OID
```bash
root@zabbix:~# snmpwalk -v3 -a SHA -A etudiant_007 -x AES -X azerty2QWERTY -u zabbix -l authPriv 192.168.8.126 1.3.6.1.2.1.1.1
```

