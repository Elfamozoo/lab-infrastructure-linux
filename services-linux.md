# Services Linux : DNS, FTP, web, fichiers, messagerie, logs

## Sommaire

- [6. Configuration DNS avec BIND9](#6-configuration-dns-avec-bind9)
  - [6.1 Pré-requis DNS](#61-pré-requis-dns)
  - [6.2 Installation](#62-installation)
  - [6.3 Déclaration des zones](#63-déclaration-des-zones)
  - [6.4 Fichiers des zones](#64-fichiers-des-zones)
  - [6.5 Vérification et démarrage DNS](#65-vérification-et-démarrage-dns)
- [7. Configuration d’un Serveur FTP(S) avec vsftpd](#7-configuration-dun-serveur-ftps-avec-vsftpd)
  - [7.1 Installation de vsftpd](#71-installation-de-vsftpd)
  - [7.2 Configuration initiale](#72-configuration-initiale)
  - [7.3 Activation de FTPS](#73-activation-de-ftps)
  - [7.4 Redémarrage et tests](#74-redémarrage-et-tests)
- [8. Installation et Configuration d’un Serveur Web Apache2](#8-installation-et-configuration-dun-serveur-web-apache2)
  - [8.1. Présentation d’Apache](#81-présentation-dapache)
- [8.2 Installation](#82-installation)
  - [8.3 Configuration de base](#83-configuration-de-base)
  - [8.4 Hôtes virtuels par numéro de port](#84-hôtes-virtuels-par-numéro-de-port)
  - [8.5 Hôtes virtuels par nom DNS (VirtualHosts)](#85-hôtes-virtuels-par-nom-dns-virtualhosts)
- [9. Partage de Fichiers avec Samba](#9-partage-de-fichiers-avec-samba)
  - [9.1 Installation de Samba](#91-installation-de-samba)
  - [9.2 Création des répertoires partagés](#92-création-des-répertoires-partagés)
  - [9.5 Finalisation sur Linux](#95-finalisation-sur-linux)
- [10. Serveur de Messagerie Postfix & Dovecot](#10-serveur-de-messagerie-postfix--dovecot)
  - [10.1 Pré-requis](#101-pré-requis)
  - [10.2. Installation des paquets](#102-installation-des-paquets)
  - [10.3. Configuration de Postfix](#103-configuration-de-postfix)
  - [10.4 Vérification SMTP](#104-vérification-smtp)
  - [10.5. Configuration de Dovecot (IMAP)](#105-configuration-de-dovecot-imap)
  - [10.6 Installation de RainLoop (Webmail)](#106-installation-de-rainloop-webmail)
- [11. Serveur de centralisation de logs (Rsyslog + MySQL + LogAnalyzer)](#11-serveur-de-centralisation-de-logs-rsyslog--mysql--loganalyzer)
  - [11.1 Topologie et rôle](#111-topologie-et-rôle)
  - [11.2 Installation du LAMP et de MySQL](#112-installation-du-lamp-et-de-mysql)
  - [11.4. Installation de LogAnalyzer. Installation de LogAnalyzer](#114-installation-de-loganalyzer-installation-de-loganalyzer)
## 6. Configuration DNS avec BIND9

Pour une infrastructure complète, un DNS interne <domaine> résout noms et IP.

### 6.1 Pré-requis DNS

VM Debian dédiée ou conjointe, IP statique (ex. 10.10.10.2).

Paquet bind9 + dnsutils.

### 6.2 Installation

apt update && apt install bind9 dnsutils

### 6.3 Déclaration des zones

- Éditez /etc/bind/named.conf :

![image-16.png](assets/image-16.png)

### 6.4 Fichiers des zones

- Zone directe <domaine>

![image-17.png](assets/image-17.png)

- Zone inversé

![image-18.png](assets/image-18.png)

### 6.5 Vérification et démarrage DNS

- Redémarrage et logs :

*systemctl**restart bind9*

*systemctl**enable bind9*

- *systemctl****status**bind9*

![image-19.png](assets/image-19.png)

- Test avec la commande dig sur Ebay.fr :

![image-20.png](assets/image-20.png)

- Test avec la commande dig sur un PC Client du réseau local :

![image-21.png](assets/image-21.png)

## 7. Configuration d’un Serveur FTP(S) avec vsftpd

Pour permettre le transfert de fichiers sécurisé, nous installerons **vsftpd** (Very Secure FTP Daemon) et activerons FTPS.

### 7.1 Installation de vsftpd

- Installez le paquet et mettez à jour le système :

*apt**update &&**apt**–y upgrade*

*apt**-y**install****vsftpd*

### 7.2 Configuration initiale

- Ouvrez /etc/vsftpd.conf et appliquez les changements suivants (le fichier est assez conséquent) :

![image-22.png](assets/image-22.png)

- Créez deux utilisateurs Linux (exemple ici avec user1) :

![image-23.png](assets/image-23.png)

- Leurs répertoires personnels seront utilisés comme racines FTP. Ajustez les permissions si nécessaire :

*find**/home/user1 -type d -**exec**chmod 750 {} \;*

*find**/home/user1 -type f -**exec**chmod 640 {} \;*

- Ajoutez la liste des utilisateurs autorisés dans */**etc**/**vsftpd.chroot_list* :

![image-24.png](assets/image-24.png)

### 7.3 Activation de FTPS

- Pour chiffrer les échanges FTP et renforcer la sécurité, suivez ces étapes:

Rendons-nous dans le répertoire dédié aux clés SSL

Nous allons y générer le fichier vsftpd.pem, contenant à la fois le certificat voulu et une clé RSA de 2048 bits, valable 365 jours :

*openssl****req**-x509 -**nodes**-**newkey**rsa:2048 -**keyout****vsftpd.pem**-out**vsftpd.pem**-**days**365*

- Le système nous demandera de saisir des informations pour compléter le certificat. Exemple :

![image-25.png](assets/image-25.png)

- Restreignons les permissions sur le fichier nouvellement créé au seul utilisateur propriétaire :

![image-26.png](assets/image-26.png)

Il faut maintenant modifier le fichier de configuration de vsftpd afin qu’il tienne compte du certificat :

*nano**/**etc**/**vsftpd.conf*

- Nous y ajouterons les lignes suivantes :

![image-27.png](assets/image-27.png)

Reste à redémarrer le service :

*systemctl**restart**vsftpd*

### 7.4 Redémarrage et tests

- Avec FileZilla, contrairement à l’étape sans SSL/TLS, nous opterons pour une « Connexion FTP explicite sur TLS » :

![image-28.png](assets/image-28.png)

- Initialement inconnu de notre PC, le certificat est affiché. À nous de l’approuver s’il donne des données cohérentes avec notre infrastructure :

![image-29.png](assets/image-29.png)

- On arrive donc à nous connecter en FTPS :

![image-30.png](assets/image-30.png)

## 8. Installation et Configuration d’un Serveur Web Apache2

Pour héberger des sites web internes ou prototypes, Apache2 constitue une solution robuste et modulable.

### 8.1. Présentation d’Apache

Apache est le serveur HTTP le plus utilisé au monde, compatible Linux et multiplateforme.

Version ciblée : Debian 10/11/12 (Apache 2.4.x).

Avantages : modularité, communauté active, large documentation.

## 8.2 Installation

- Mettez à jour vos dépôts puis installez le paquet :
*apt**update &&**apt****install**apache2 -y*

- Vérifiez la version :
*apache**2 -v*

- Assurez-vous que le service est actif :
- *systemctl****status**apache2*

![image-31.png](assets/image-31.png)

### 8.3 Configuration de base

- Définir ServerName pour éviter les erreurs au démarrage :
echo "ServerName <domaine>" >> /etc/apache2/apache2.conf

- Tester la configuration :
apache2 -t

- Redémarrer Apache :
*systemctl**restart apache2*

### 8.4 Hôtes virtuels par numéro de port

La méthode la plus utilisée consiste à faire correspondre un nom de domaine virtuel à un répertoire.

- Créer les répertoires :
*mkdir**-p /var/www/**vhosts**/site1 /var/www/**vhosts**/site2*

- Créer une page d’accueil pour le site 1 avec la commande :
*nano**/var/www/**vhosts**/site1/index.html*

- Ajouter ces lignes dans le fichier « /var/www/vhosts/site1/index.html » :

![image-32.png](assets/image-32.png)

- Faire la même chose pour la page 2.

![image-33.png](assets/image-33.png)

- Créer un fichier de configuration «/etc/apache2/sites-available/port_vhosts.conf » avec la commande :
*nano**/**etc**/apache2/sites-**available**/**port_vhosts.conf*

- Ajouter ces lignes dans le fichier « /etc/apache2/sites-available/port_vhosts.conf »

![image-34.png](assets/image-34.png)

Le site 1 est configuré pour répondre sur le port 80 et le site 2 est configuré pour répondre sur le port 8080.

Configurer Apache pour qu’il écoute sur le port 8080, en plus du port 80.

Il faut, pour cela, éditer le fichier « /etc/apache2/ports.conf » avec la commande :

*nano**/**etc**/apache2/**ports.conf*

- Ajouter la ligne « Listen 8080 » dans le fichier « /etc/apache2/ports.conf » :

![image-35.png](assets/image-35.png)

Activer la configuration des hôtes virtuels avec la commande :

*a**2ensite**port_vhosts*

Redémarrer le service apache2 :

*systemctl****reload**apache2*

- Visualiser la configuration via le navigateur web et constater la présence de 2 sites sur la même adresse IP mais différenciés par le numéro de port

![image-36.png](assets/image-36.png)

![image-37.png](assets/image-37.png)

### 8.5 Hôtes virtuels par nom DNS (VirtualHosts)

Pour cette configuration, une seule adresse IP est utilisée.

Il s’agit de la méthode recommandée et aussi la plus utilisée puisqu’il est plus facile de retenir un nom qu’une adresse IP.

Plusieurs noms DNS sont associés à une seule adresse IP et correspondent à plusieurs sites web.

- Sur le serveur DNS nous allons ajouter à la zone de recherche direct le serveur DNS et les 2 sites (on va utiliser le CNAME).
- L’URL site1.<domaine> sera utilisée pour le site 1.
- L’URL site2.<domaine> sera utilisée pour le site 2.

![image-38.png](assets/image-38.png)

- Désactiver la configuration précédente avec la commande :
*a**2dissite**port_vhosts*

- Redémarrer le service apache2 :
*systemctl****reload**apache2*

- Créer un fichier de configuration avec la commande :
*nano**/**etc**/apache2/sites-**available**/**name_vhosts.conf*

- Ajouter ces lignes dans le fichier « /etc/apache2/sites-available/name_vhosts.conf » :

![image-39.png](assets/image-39.png)

- Dans le fichier « /var/www/vhosts/site1/index.html », ajouter la ligne :

![image-40.png](assets/image-40.png)

- Faites de même pour le site2 :

![image-41.png](assets/image-41.png)

Activer la configuration des hôtes virtuels avec la commande :

*a**2ensite**name_vhosts*

Redémarrer le service apache2 :

*systemctl****reload**apache2*

- Visualiser la configuration via le navigateur web :

![image-42.png](assets/image-42.png)

![image-43.png](assets/image-43.png)

## 9. Partage de Fichiers avec Samba

### 9.1 Installation de Samba

-Installez Samba et les utilitaires :

```bash
apt install samba samba-common-bin -y
```

- Répondez non aux questions DHCP et WINS si elles apparaissent.

### 9.2 Création des répertoires partagés

Créez un répertoire principal et deux sousdossiers pour élèves et enseignants :

```bash
mkdir -p /home/NAS/Partage_eleves
mkdir -p /home/NAS/Partage_enseignants
```

Appliquez les permissions de base (lecture/écriture pour le groupe) :

```bash
chmod 770 /home/NAS/Partage_eleves
chmod 770 /home/NAS/Partage_enseignants
```

- Création des utilisateurs et groupes
- Créez deux groupes Unix :

```
groupadd eleves
groupadd enseignants
```

- Ajoutez des utilisateurs dans chaque groupe :

```
useradd -m -g eleves eleve1
useradd -m -g eleves eleve2
useradd -m -g enseignants prof1
useradd -m -g enseignants prof2
```

- Assignez un mot de passe Samba à chacun :

```
smbpasswd -a eleve1
smbpasswd -a eleve2
smbpasswd -a prof1
smbpasswd -a prof2
```

- Configuration de Samba
- Sauvegardez la configuration par défaut :

```bash
cp /etc/samba/smb.conf /etc/samba/smb.conf.save
```

- Ouvrez /etc/samba/smb.conf
- Changez ses paramètres comme sur la photo :

![image-44.png](assets/image-44.png)

- Et ajoutez sous Share Definitions :

![image-45.png](assets/image-45.png)

- Appliquez les changements avec :

```bash
systemctl restart smbd.service
```

```
testparm
```

### 9.5 Finalisation sur Linux

Pour que les permissions Unix reflètent celles de Samba, ajustez la propriété et les droits :

```bash
chown -R root:eleves /home/NAS/Partage_eleves
chown -R root:enseignants /home/NAS/Partage_enseignants
chmod -R ug+rwx,o+rx-w /home/NAS
systemctl restart smbd.service
```

- Vérification depuis un client Windows
- Notez l’adresse IP du serveur (ip a).
- Dans l’explorateur Windows, saisissez l’UNC :

![image-46.png](assets/image-46.png)

- Testez l’accès avec eleve1, eleve2, prof1 ou prof2, créez/modifiez des fichiers pour vérifier les droits.

![image-47.png](assets/image-47.png)

![image-48.png](assets/image-48.png)

## 10. Serveur de Messagerie Postfix & Dovecot

Pour fournir un service de messagerie interne complet (SMTP, IMAP) et webmail, nous utiliserons Postfix et Dovecot, accompagnés de RainLoop pour l’accès web.

### 10.1 Pré-requis

VM Debian avec IP statique (ex. 10.10.10.4).

Nom de domaine interne (mail.<domaine>) pointant vers cette IP via DNS.

Paquets : postfix, mailutils, dovecot-core, dovecot-imapd, apache2, php, libapache2-mod-php.

### 10.2. Installation des paquets

```bash
apt update && apt upgrade -y
apt install postfix mailutils dovecot-core dovecot-imapd apache2 php libapache2-mod-php -y
```

- Pendant l’installation de Postfix :
Sélectionnez **Internet Site**.

- Entrez le nom de domaine **mail.<domaine>**.

![image-49.png](assets/image-49.png)

![image-50.png](assets/image-50.png)

### 10.3. Configuration de Postfix

- Sauvegardez la configuration par défaut :

```bash
cp /etc/postfix/main.cf /etc/postfix/main.cf.backup
```

- Éditez /etc/postfix/main.cf et ajustez comme sur la photo :

![image-51.png](assets/image-51.png)

- Redémarrez Postfix :

```bash
systemctl restart postfix
systemctl enable postfix
```

### 10.4 Vérification SMTP

- Testez l’envoi local :

![image-52.png](assets/image-52.png)

### 10.5. Configuration de Dovecot (IMAP)

- Activez l’écoute : dans /etc/dovecot/dovecot.conf, décommentez :

```
listen = *
```

- Authentification : dans /etc/dovecot/conf.d/10-auth.conf :

```
disable_plaintext_auth = no
auth_mechanisms = plain login
```

![image-53.png](assets/image-53.png)

![image-54.png](assets/image-54.png)

Emplacement des emails : dans /etc/dovecot/conf.d/10-mail.conf :

```
mail_location = maildir:~/Maildir
```

![image-55.png](assets/image-55.png)

- Permissions du service : dans /etc/dovecot/conf.d/10-master.conf :

![image-56.png](assets/image-56.png)

Redémarrez Dovecot :

```bash
systemctl restart dovecot
systemctl enable dovecot
```

### 10.6 Installation de RainLoop (Webmail)

Avant l’installation de RainLoop, assurez-vous d’avoir installé les dépendances nécessaires :

```bash
apt update && apt install apache2 php libapache2-mod-php php-curl php-xml php-mbstring -y
```

- Puis procédez à l’installation de RainLoop :

```bash
cd /var/www/html
rm index.html
curl -sL https://repository.rainloop.net/installer.php | php
```

- Accédez ensuite à : http://mail.<domaine>/?admin

```
**User** : admin / **Password** : <MOT_DE_PASSE>
```

- Ajoutez le domaine <domaine> avec serveur IMAP/SMTP 127.0.0.1.

![image-57.png](assets/image-57.png)

- Ensuite on se connecte à l’utilisateur que l’on a créé précédemment la Mounir par exemple.

![image-58.png](assets/image-58.png)

- Ensuite une fois connecté nous allons envoyer un mail afin de tester le bon fonctionnement du serveur mail :

![image-59.png](assets/image-59.png)

- Suite à l’envoi du mail nous pouvons voir que nous l’avons bien reçu

![image-60.png](assets/image-60.png)

## 11. Serveur de centralisation de logs (Rsyslog + MySQL + LogAnalyzer)

Pour agréger et analyser les journaux de vos machines Linux et Windows, nous mettons en place un serveur de logs centralisé basé sur rsyslog, MySQL/MariaDB et LogAnalyzer.

### 11.1 Topologie et rôle

- **Serveur****syslog** (Debian 12, IP 10.10.10.5) : réception et stockage des logs dans une base.
- **Serveur web** (même machine) : héberge l’interface LogAnalyzer.
- **Clients Linux et Windows** : envoient leurs logs vers le serveur syslog.

### 11.2 Installation du LAMP et de MySQL

- Mettez à jour et installez Apache, PHP, MariaDB :

```bash
apt update && apt upgrade -y
apt install apache2 php libapache2-mod-php php-mysql mariadb-server -y
```

- Sécurisez MariaDB :

```
mysql_secure_installation
```

![image-61.png](assets/image-61.png)

- Créez une base Syslog et un utilisateur rsyslog :

```bash
sudo mysql -u root -p
```

Puis, d’un seul tenant

- Créez une base Syslog et un utilisateur rsyslog :

```bash
apt install rsyslog rsyslog-mysql -y
```

![image-62.png](assets/image-62.png)

![image-63.png](assets/image-63.png)

![image-64.png](assets/image-64.png)

- Modification du fichier rsyslog.conf

![image-65.png](assets/image-65.png)

![image-66.png](assets/image-66.png)

- Redemarrer rsyslog

```bash
systemctl restart rsyslog
```

### 11.4. Installation de LogAnalyzer. Installation de LogAnalyzer

- Téléchargez la version stable :

```bash
cd /src
wget https://download.adiscon.com/loganalyzer/loganalyzer-4.1.13.tar.gz
```

```
tar -zxvf loganalyzer-4.1.13.tar.gz
```

```bash
mkdir -p /var/www/html/loganalyzer
cp -r loganalyzer-4.1.13/* /var/www/html/loganalyzer/
chown -R www-data:www-data /var/www/html/loganalyzer
```

- Dans le navigateur, accédez à

```
http://10.10.10.5/loganalyzer
```

![image-67.png](assets/image-67.png)

![image-68.png](assets/image-68.png)

- Suivez l’assistant :

![image-69.png](assets/image-69.png)

![image-70.png](assets/image-70.png)

![image-71.png](assets/image-71.png)

![image-72.png](assets/image-72.png)

![image-73.png](assets/image-73.png)

![image-74.png](assets/image-74.png)

![image-75.png](assets/image-75.png)

![image-76.png](assets/image-76.png)

![image-77.png](assets/image-77.png)

- Vous devriez avoir cela dans la page d’accueil :

![image-78.png](assets/image-78.png)

Si vous avez une erreur d’affichage verifiez bien les logs apache afin de voir d’où provient le souci :

```bash
sudo tail -f /var/log/apache2/error.log
```

Dans mon cas la colonne processid et checksum n’était pas présente je l’ai donc ajouté à la table manuellement :

*sudo****mysql**-u root -p -e "*

*USE**Syslog;*

*ALTER TABLE**SystemEvents*

*ADD COLUMN checksum**INT(**11) NULL DEFAULT NULL,*

*ADD COLUMN**processid****VARCHAR(**60) NULL DEFAULT NULL;*

- Ensuite sur un des serveur ( le SRV-MAIL par ex ) on va installer Rsyslog et modifier quelques fichiers :
- On va ensuite modifier le fichier suivant :

![image-79.png](assets/image-79.png)

![image-80.png](assets/image-80.png)

- Puis modifions ce fichier aussi :

![image-81.png](assets/image-81.png)

![image-82.png](assets/image-82.png)

- Ensuite on reboot le service rsyslog :

![image-83.png](assets/image-83.png)

- Le server web remonte bien ses logs :

![image-84.png](assets/image-84.png)

- Ensuite on va télécharger et installer rsyslog sur un Windows 10
- Puis, à installer sur le serveur ou le client Windows dont on souhaite remonter les logs :
- Test la connexion :

![image-85.png](assets/image-85.png)

- Il faut vérifier la réception des logs dans l’interface de LogAnalyzer.

![image-86.png](assets/image-86.png)

- Et les logs remontent sur Loganalyzer :

![image-87.png](assets/image-87.png)
