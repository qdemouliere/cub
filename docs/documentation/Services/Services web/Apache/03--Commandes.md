# III. Principales commandes pour Apache 2

## 1. Gérer l’activité du daemon apache

```bash
sudo systemctl stop|start|restart|reload apache2
```

## 2. Active le site test contenu dans le répertoire /etc/apache2/sites-available

```bash
sudo a2ensite test
```

## 3. Désactive le site contenu dans le répertoire /etc/apache2/sites-enabled

```bash
sudo a2dissite test
```

## 5. Active ou désactive un module pour Apache

```bash
sudo a2enmod ssl
sudo a2dismod ssl
```
## 6. Consulter les logs d'erreur d'Apache

```bash
sudo cat /var/log/apache2/error.log
```

## 7. Consulter les logs d'accès au service Apache

```bash
sudo cat /var/log/apache2/access.log
```

## 8. Consulter en temps réel les nouvelles entrées dans un fichier de log Apache

```bash
sudo tail -f /var/log/apache2/error.log
```
