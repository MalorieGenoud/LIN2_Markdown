**Auteurs:** Barchichat Emmanuel, Genoud Malorie, Sejmenovic Aida

**Date:** 08.01.2016

**Version Debian utilisée:** 8

# Configuration d'un hébergement de site Web

**!!! TOUTE L'INSTALLATION A ÉTÉ FAITE SUR DEBIAN 8 EN "FROM SCRATCH" !!!**

## Installation de la machine virtuelle

Tout d'abord si vous ne possédez pas d'image ISO de Debian 8, voici un lien pour télécharger l'image ISO:

[Télécharger Debian 8](https://www.debian.org/CD/http-ftp/#stable)

Une fois que vous avez cliqué sur le lien, choisissez dans la partie **CD** le lien **amd64** puis choisissez un des liens d'images ISO disponibles.

Pour installer la machine virtuelle, cliquez sur "File" et choisissez "New virtual machine" et suivez les différentes étapes pour la configuration de bases des ressources:

**Processeurs:** 1

**Mémoire:** 2GB

**Réseau:** NAT

**SCSI Controller:** LSI Logic (recommandé)

**Type de disque virtuel:** SCSI (recommandé)

**Disque dur:** 60 GB

Une fois la configuration faite, vous pouvez démarrer la machine virtuelle. Voici les étapes:

**N'INSTALLEZ PAS D'INTERFACE GRAPHIQUE! FAÎTES UNE SIMPLE INSTALLATION!**

**Langue** À choix

**Situation géographique:** Â choix

**Configuration clavier:** À choix

**Nom de machine:** À choix

**Domaine:** À choix

**Mot de passe root:** À choix

**Nom du nouvel utilisateur et mot de passe:** À choix

**Partitionner les disques:** Assisté - utiliser un disque entier / Partitions /home, /var et /tmp séparées / Valider le partitionnement

**Configurer l'outil de gestion des paquets:** Non / Non

**Configuration de popularity-contest:** Non

**Sélection des logiciels:** Décocher "environnement de bureau Debian" à l'aide de la touche "espace"

**Installer le programme de démarrage GRUB sur un disque dur:** Oui / Choisir "/dev/sda"

## Quelques réglages à faire avant l'installation des différents services

Pour procéder aux différents réglages, il faut être connecté en **root**.

Dans un premier temps, nous allons modifier quelques lignes dans le fichier **sources.list** afin d'avoir les mêmes dépots qui se synchronisent:

`~# nano /etc/apt/sources.list`

Il faut commenter la `deb cdrom:[...]` en mettant **#** au début et ajouter les lignes suivantes :

`deb http://nginx.org/packages/debian/ jessie nginx`

`deb-src http://nginx.org/packages/debian/ jessie nginx`

`deb http://ftp.ch.debian.org/debian/ jessie main contrib non-free`

`deb-src http://ftp.ch.debian.org/debian/ jessie main contrib non-free`

`deb http://mariadb.biz.net.id//repo/10.1/debian jessie main`

Sauvegardez le fichier et lancer la commande suivante afin de faire la mise à jour des dépôts :

`~# apt-get update`

`~# apt-get upgrade`

Ces commandes permettent d'éviter quelques problèmes d'installations des services par la suite.

Il est aussi possible que lors de la fin de la mise à jour, vous ayez une voir deux lignes vous disant qu'il manque des clés. Vous pouvez passer outre de cette information car ceci n'a pas d'impact sur la suite mais si vous souhaitez installer les clés, voici un lien proposant diverses clés **GPG** :

[Clé GPG](http://korben.info/keyserver-ubuntu-com-inaccessible-que-faire.html )

Afin de faire en sorte que chacun des utilisateurs qui seront créés ne puissent pas accéder aux répertoires des autres utilisateurs, il faut modifier le umask de façon permanente. Il faut donc éditer le fichier **.bashrc**:

`~# nano ~/.bashrc`

Changez la valeur par défaut du umask par `077` car ceci permettra qu'un utilisateur aura accès seulement à son propre répertoire.


## Installer et configurer le serveur OpenSSH

Commençons par installer **OpenSSH** avec la commande suivante:

`~# apt-get install openssh-server`

Il n'y que quelques modifications à faire car avec les dernières versions il y a déjà une bonne configuration au niveau de la sécurité.

Commencez par éditer le fichier **sshd_config**:

`~# nano /etc/ssh/sshd_config`

Modifiez la ligne `PermitRootLogin   without-password` et remplacez le paramètre `without-password` par le paramètre `no` afin d'éviter une quelconque connexion avec l'utilisateur **root**.

Une fois le fichier sauvegardé, rechargez la configuration ssh:

`~# /etc/init.d/ssh reload`


## Installer et configurer Nginx

Pour installer **Nginx** il faut utiliser la commande suivante:

`~# apt-get install nginx`

Maintenant démarrez le service :

`~# service nginx start`

Il est possible de vérifier les informations en pointant le moteur de recherche sur notre adresse IP, il devrait confirmer que **Nginx** a bien été installé. Pour trouver votre VPS IP adresse il faut taper la commande suivante :

`~# ifconfig eth0 | grep inet | awk  '{ print $2 }'`

Pour la configuration, ouvrez le host virtuel avec la commande suivante :

`~# nano /etc/nginx/conf.d/default.conf`

Il faut changer certaines lignes du fichier, notamment le nom de votre serveur qui est dans l'exemple ci-dessous `NameServer`.
Pour savoir votre nom de serveur et d'hôte si besoin tapez la commande suivante :

`~# nano /etc/hosts`

Voilà le résulat une fois les lignes changées :

      [...]
      server {
      listen 80;
      server_name  NameServer;

      location / {
          root   /usr/share/nginx/html;
          index index.php index.html index.htm index.nginx-debian.html;
        }
      }

      [...]

      location ~ \.php$ {
             proxy_pass   http://127.0.0.1;
             fastcgi_pass unix:/var/run/php5-fpm.sock;
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name$;
             include fastcgi_params;
      }
      [...]


Il reste une dernière étape à faire. Ajoutez **Nginx** au groupe des utilisateurs qui seront créer plus tard. Voici la commande:

`~# chown -R www-data:NameGroup /var/log/nginx && chmod g+s /var/log/nginx`


## Installer et configurer PHP-FPM

Pour installer **PHP-FPM** il faut utiliser la commande suivante:

`~# apt-get install php5-fpm php5-mysql`

Éditez le fichier **php.ini**:

`~# nano /etc/php5/fpm/php.ini`

Vous devez chercher et vérifier que la ligne suivante est écrite ainsi `cgi.fix_pathinfo=1`. La valeur à **1** permet d'avoir une meilleure sécurité au niveau de l'interpréteur PHP et change sa manière dont il va traiter les données qu'il recevra du serveur **Nginx**. Par contre si vous avez cette ligne-ci `cgi.fix_pathinfo=0`, changez la valeur **0** par **1**.

Il y a une autre vérification à faire avant de passer à la suite. Utilisez la commande suivante:

`~# nano /etc/php5/fpm/pool.d/www.conf`

Vous devez chercher et vérifier si la ligne suivante est écrite ainsi `listen = /var/run/php5-fpm.sock`. Ceci permettra de faire en sorte qu'il ne va pas "écouter" sur le réseau local mais sur un socket Unix. Si c'est le cas, ne changez pas la ligne. Par contre si ce n'est pas le cas, vous devriez avoir cette ligne-ci à la place `listen = 127.0.0.1:9000`, remplacez-la par `listen = /var/run/php5-fpm.sock`.

Afin que **PHP-FPM** prenne en charge toutes les modifications faites précédemment, redémarrez-le:

`~# service php5-fpm restart`

Maintenant que tout a été configuré, vous pouvez créer un fichier d'informations:

`~# cd /usr/share/ngnix/html`

`~# touch info.php`   

`~# nano info.php`

Il est possible qu'à la place du dossier `html` vous ayez un dossier nommer `www` car tout dépend de la version de **Nginx** qui a été chargée. Mais le principe reste le même.

Écrivez la ligne suivante dans ce fichier :

`<?php   phpinfo()   ?>`

Redémarrez le serveur **Nginx**:

`~# service nginx restart`


## Installer et configurer MariaDB

Pour installer **MariaDB**, utilisez la commande suivante:

`~# apt-get install mariadb-server mariadb-client`

Par la suite une fenêtre va s'ouvrir en vous demandant d'insérer un mot de passe. Il est important que vous vous souveniez de ce mot de passe car il vous sera utile pour vous connctez à **MariaDB**.

Exemple de mot de passe : `D08x97H$`

Si l'installation se passe bien, vous pouvez vérifier la version de **MariaDB** afin d'être sûr que vous disposiez de la dernière version qui est la 10.0 :

`~# mysql -V`

Voici ce que vous devriez obtenir :

`mysql  Ver 15.1 Distrib 10.0.22-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2`

###   Utilisation des diverses commandes dans MariaDB

Nous allons apprendre à utiliser quelques commandes de **MariaDB**.

Tout d'abord, il faut se connecter à la base de données en tant que **root** afin de pouvoir créer un utilisateur :

`~# mysql -u root -p`

Une fois cette commande tapée, un mot de passe va être demandé.

**ATTENTION LE MOT DE PASSE N'EST PAS LE MOT DE PASSE DE L'UTILISATEUR ROOT, MAIS CELUI QUI A ÉTÉ MIS LORS DE L'INSTALLATION DE MARIADB**

Pour chaque utilisateur qui sera créé par la suite, chacun aura accès seulement à sa propre base de données.

Commande pour créer un utilisateur :

`MariaDB [(none)]> CREATE USER 'NameUser' IDENTIFIED BY 'NameUser';`

Maintenant quittez la base de données et connectez-vous avec l'utilisateur créé :

`MariaDB [(none)] exit`

`~# mysql -u NameUser -p`

Une fois connecté avec l'utilisateur créer précédemment, vous piuvez afficher les différentes bases de données actuelles :

`MariaDB[(none)]> SHOW DATABASES;`

Voici le résultat obtenu:

     +--------------------+
     | Database           |
     +--------------------+
     | information_schema |
     | mysql              |
     | performance_schema |
     +--------------------+
     3 rows in set (0.04 sec)

Voici la commande à faire pour créer une base de données :

`MariaDB [mysql]> CREATE DATABASE NameDataBase;`

Pour créer une table **user** dans cette base de données, on utilisera la commande suivante :

`MariaDB [mysql]> CREATE TABLE user;`

Tapez la commande suivante pour voir les droits sur un ou tous les utilisateurs :

`MariaDB [mysql]> SHOW GRANTS;`


# Configuration des utilisateurs pour les différents services

## Utilisateur standard

Créez un répertoire qui regroupera les dossiers/fichiers des utilisateurs:

`mkdir /home/Applications`

VCréez un nouvel utilisateur :

`~# adduser NameUser`

Créez un groupe commun à tous les utilisateurs et ajoutez les dans ce groupe. Voici les commandes:

`~# groupadd -g NameGroup`

`~# usermod -g NameGroup NameUser`

Dans le répertoire `mkdir /home/Applications/` créez le répertoire Webapp pour l'utilisateur créé:

`mkdir /home/Applications/NameUser_Webapp`

Maintenant attribuez l'utilisateur à son répertoire avec la commande suivante :

`~# chown NameUser /home/Applications/NameUser_Webapp`


## Utilisateur Nginx

Il est nécessaire que pour chacun des utilisateurs qui seront créés, que chacun possède sont propre fichier de configuration. Modifiez le fichier `/etc/nginx/conf.d/default.conf` par le nom de l'utilisateur `NameUser.conf`

    [...]
    server {
    listen 80;
    root /usr/share/nginx/conf.d/NameUser;
    server_name NameUser;
    location / {
            root   /usr/share/nginx/html;
            index index.php index.html index.htm index.nginx-debian.html;
      }
    location ~ \.php$ {
            try_files $uri =404;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
            fastcgi_pass unix:/var/run/NameUser.sock;
      }
    }
    [...]

Il faut maintenant redémarrer le service:

`~# /etc/init.d/nginx restart`

Créez un répertoire pour chacun des utilisateurs dans `/etc/nginx/conf.d/` qui regroupera leur site Web. Il n'est pas utile de retirez les droits pour les "others" car normalement le umask retire déjà les droits mais si vous voulez être sûr que les "others" n'ont pas le droits vous pouvez exéctutez les commandes suivantes:

`~# chmod 770 /usr/share/nginx/conf.d/*`

Vous pouvez maintenant créer le reépertoire pour chacun des utilisateurs et attribuer le dossier à l'utilisateur:

`~# mkdir /etc/nginx/conf.d/NameUserRecord`

`~# chown NameUser NameUserRecord`


## Utilisateur PHP-FPM

Il est nécessaire que pour chacun des utilisateurs qui seront créés, que chacun possède sa propre configuration. Copiez le fichier `/etc/php5/fpm/php.ini` et nommez le avec le nom de l'utilisateur `NameUser.ini`

Modifiez les informations suivantes:

    [www] -> [NameUser]
    user = Nameuser
    group = NameGroup
    [...]
    listen = /var/run/NameUser.sock
    [..]
    listen.owner = NameUser
    listen.group = NameUser

Redémarrez le service:

`~# /etc/init.d/php5-fpm restart`


## Utilisateur MariaDB

Pour chaque utilisateur qui sera créé par la suite, chacun aura accès seulement à sa propre base de données.

Commande pour créer un utilisateur :

`MariaDB [(none)]> CREATE USER 'NameUser' IDENTIFIED BY 'NameUser';`

Maintenant quittez la base de données et connectez-vous avec l'utilisateur créé :

`MariaDB [(none)] exit`

`~# mysql -u NameUser -p`


# Source d'installation et de configuration

## Liens pour l'installation des packages

[Lien 1](http://linuxconfig.org/debian-apt-get-jessie-sources-list)

[Lien 2](http://nginx.org/en/linux_packages.html)


### OpenSSH

[Lien 1](https://openclassrooms.com/courses/installation-et-utilisation-d-un-serveur-ssh-sous-debian-etch)

[Lien 2](http://doc.ubuntu-fr.org/tutoriel/reverse_ssh)

[Lien 3](http://www.linuxtricks.fr/wiki/ssh-installer-et-configurer-un-serveur-ssh#paragraph_installation-du-serveur-openssh)

[Lien 4](http://doc.ubuntu-fr.org/ssh)


### Ngnix

[Lien 1](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-debian-7)


### PHP-FPM

[Lien 1](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-debian-7)


### MariaDB

[Lien 1](https://mariadb.com/kb/en/mariadb/configuring-mariadb-for-remote-client-access/)
