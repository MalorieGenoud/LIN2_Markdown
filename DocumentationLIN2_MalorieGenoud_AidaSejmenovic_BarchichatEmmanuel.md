**Auteurs:** Barchichat Emmanuel, Genoud Malorie, Sejmenovic Aida

**Date:** 08.01.2016

**Version Debian utilisée:** 8

# Configuration d'un hébergement de site Web

**!!! TOUTE L'INSTALLATION A ÉTÉ FAITE SUR DEBIAN 8 EN "FROM SCRATCH" !!!**

## Ressources utilisées et les points faire avant l'installation

Tout d'abord si vous ne possédez pas d'image ISO de Debian 8, voici un lien pour télécharger l'image ISO:

[Télécharger Debian 8](https://www.debian.org/CD/http-ftp/#stable)

Une fois que vous avez cliqué sur le lien, choisissez dans la partie **CD** le lien **amd64** puis choisissez un des liens d'images ISO disponibles

**Mémoire:** 2GB

**Processeurs:** 1

**Disque dur:** 60 GB

Pour procéder aux différentes installations, il faut être connecté en **root**.

Dans un premier temps, nous allons modifier quelques lignes afin d'avoir les mêmes dépots qui se synchronisent. On va éditer le fichier **sources.list** :

`~# nano /etc/apt/sources.list`

Il faut commenter la dernière ligne en mettant un **#** au début et ajouter les lignes suivantes :

`deb http://nginx.org/packages/debian/ jessie nginx`

`deb-src http://nginx.org/packages/debian/ jessie nginx`

`deb http://ftp.ch.debian.org/debian/ jessie main contrib non-free`

`deb-src http://ftp.ch.debian.org/debian/ jessie main contrib non-free`

`deb http://mariadb.biz.net.id//repo/10.1/debian jessie main`

Il faut sauvegarder le fichier et lancer la commande suivante afin de faire la mise à jour des dépôts :

`~# apt-get update`

`~# apt-get upgrade`

Ces commandes éviterons quelques soucis lors de l'installation des différents services à l'avenir.

Afin de faire en sorte que chacun des utilisateurs qui seront créés ne puissent pas accéder aux répertoires des autres utilisateurs, il faut modifier le umask de façon permanente. Voici la commande à faire:

`~# nano ~/.bashrc`

Il faut ensuite changer la valeur par défaut du umask par `077`


## Installer et configurer le serveur OpenSSH

Commençons par installer **OpenSSH** avec la commande suivante :

`~# apt-get install openssh-server`

Maintenant on va faire une petite configuration du serveur car avec les dernières versions, il y a déjà une bonne configuration au niveau de la sécurité.

On va commencer par éditer le fichier **sshd_config** :

`~# nano /etc/ssh/sshd_config`

On va modifier la ligne `PermitRootLogin   without-password` et remplacer le paramètre `without-password` par le paramètre `no` afin d'éviter une quelconque connexion avec l'utilisateur **root**.

Une fois le fichier sauvegardé, on va recharger la configuration ssh :

`~# /etc/init.d/ssh reload`


## Installer et configurer Nginx

Pour installer **Nginx** il faut utiliser la commande suivante :

`~# apt-get install nginx`

Il est possible qu'il vous demande s'il faut installer les paquets sans vérification. Vous pouvez sans autre lui dire "oui".

Maintenant on va démarrer le service :

`~# service nginx start`

On peut vérifier nos informations en pointant un moteur de recherche sur notre adresse IP, il devrait confirmer que **Nginx** a bien été installé. Pour trouver notre VPS IP adresse il faut taper la commande suivante :

`~# ifconfig eth0 | grep inet | awk  '{ print $2 }'`

Pour la configuration, on ouvre le host virtuel avec la commande suivante :

`~# nano /etc/nginx/conf.d/default.conf`

Il faut changer certaines lignes du fichier, notamment le nom de votre serveur qui est dans l'exemple ci-dessous `NameServer`.
Pour savoir votre nom de serveur et de hôte si besoin tapez la commande suivante :

`~# nano /etc/hosts`

Voilà ce que vous devriez avoir une fois les lignes changées :

      [...]
      server {
      listen       80;
      server_name  NameServer;

      location / {
          root   /usr/share/nginx/html;
          index index.php index.html index.htm index.nginx-debian.html;
      }
     }
      [...]

Il reste une dernière étape à faire. Il faut maintenant ajouter **Nginx** au groupe des utilisateurs qui seront créer plus tard. Voici la commande:

`~# chown -R www-data:NameGroup /var/log/nginx && chmod g+s /var/log/nginx`


## Installer et configurer PHP-FPM

Pour installer **PHP-FPM** il faut utiliser la commande suivante :

`~# apt-get install php5-fpm php5-mysql`

Nous allons maintenant nous occuper de configurer **PHP-FPM**. Pour ce faire, il faut faire un petit changement dans le fichier **php.ini**. Voici la commande à taper :

`~# nano /etc/php5/fpm/php.ini`

Vous devez chercher et vérifier que la ligne suivante est écrite ainsi `cgi.fix_pathinfo=1`. La valeur à **1** permet d'avoir une meilleur sécurité au niveau de l'interpréteur PHP et change sa manière dont il va traiter les données qu'il recevra du serveur **Nginx**. Par contre si vous avez cette ligne-ci `cgi.fix_pathinfo=0`, changez la valeur **0** par **1**.

Il y a une dernière vérification à faire avant de passer à la suite. Utilisez la commande suivante :

`~# nano /etc/php5/fpm/pool.d/www.conf`

Vous devez chercher et vérifier si la ligne suivante est écrite ainsi `listen = /var/run/php5-fpm.sock`. Ceci permettra de faire en sorte qu'il ne va pas "écouter" sur le réseau local mais sur un socket Unix. Si c'est le cas, ne changez pas la ligne. Par contre si ce n'est pas le cas, vous devriez avoir cette ligne-ci à la place `listen = 127.0.0.1:9000`, remplacez-la par `listen = /var/run/php5-fpm.sock`.

Afin que **PHP-FPM** prenne en charge toutes les modifications faites précédemment, nous allons le redémarrer. Voici la commande à taper :

`~# service php5-fpm restart`

Maintenant que tout a été configuré, nous pouvons créer un fichier d'informations. Voici la commande à taper :

`~# cd /usr/share/ngnix/html`

`~# touch info.php`   

`~# nano info.php`

Il est possible qu'à la place du dossier `html` vous ayez un dossier nommer `www`. Mais le principe reste le même.

Écrivez la ligne suivante dans ce fichier :

`<?php   phpinfo()   ?>`

Redémarrez le serveur **Nginx** avec la commande suivante :

`~# service nginx restart`


## Installer et configurer MariaDB

Dans cette partie, il est possible d'installer des clés **GPG** mais ceci n'a pas de réel impact sur l'installation, mais au cas où voici un lien proposant diverses clés **GPG** :

[Clé GPG](http://korben.info/keyserver-ubuntu-com-inaccessible-que-faire.html )

Afin d'être sûr d'avoir les dernières mises à jour des dépôts, nous allons faire une nouvelle mise à jour :

`~# apt-get update`

`~# apt-get upgrade`

Voici la commande pour installer la base de données du côté serveur et du côté client :

`~# apt-get install mariadb-server mariadb-client`

À la suite de ça une fenêtre va s'ouvrir et demander l'ajout d'un mot de passe. Il est important de se souvenir de ce mot de passe car il sera réutilisé plus tard pour la connexion à la base de données.

Exemple de mot de passe : `D08x97H$`

Si l'installation se passe bien, vous pouvez vérifier la version de **MariaDB** afin d'être sûr que vous disposiez de la dernière version :

`~# mysql -V`

Voici ce que vous devriez obtenir :

`mysql  Ver 15.1 Distrib 10.0.22-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2`


### Utilisation des diverses commandes dans MariaDB

Dans cette partie, il est possible d'installer des clés **GPG** mais ceci n'a pas de réel impact sur l'installation, mais au cas où voici un lien proposant diverses clés **GPG** :

[Clé GPG](http://korben.info/keyserver-ubuntu-com-inaccessible-que-faire.html )

Afin d'être sûr d'avoir les dernières mises à jour des dépôts, nous allons faire une nouvelle mise à jour :

`~# apt-get update`

`~# apt-get upgrade`

Voici la commande pour installer la base de données du côté serveur et du côté client :

`~# apt-get install mariadb-server mariadb-client`

À la suite de ça une fenêtre va s'ouvrir et demander l'ajout d'un mot de passe. Il est important de se souvenir de ce mot de passe car il sera réutilisé plus tard pour la connexion à la base de données.

Exemple de mot de passe : `D08x97H$`

Si l'installation se passe bien, vous pouvez vérifier la version de **MariaDB** afin d'être sûr que vous disposiez de la dernière version :

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

Une fois connecté avec l'utilisateur créer précédemment, on va afficher les différentes bases de données actuelles :

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


## Création des différents utilisateurs

### Utilisateur standard

Il faut créer un répertoire qui regroupera tous les sites Web des utilisateurs. Il faut utiliser la commande suivante :

`mkdir /home/Applications`

Voici la commande à utiliser pour la création d'un utilisateur :

`~# adduser NameUser`

On va maintenant créer un groupe commun à tous les utilisateurs et les ajouter dans ce groupe. Voici les commandes:

`~# groupadd -g NameGroup`

`~# usermod -g NameGroup NameUser`

Dans le répertoire `mkdir /home/Applications/` nous allons créer le répertoire Webapp pour l'utilisateur créé:

`mkdir /home/Applications/NameUser_Webapp`

Maintenant on va l'attribuer à l'utilisateur avec la commande suivante :

`~# chown NameUser /home/Applications/NameUser_Webapp`

### Utilisateur PHP-FPM


### Utilisateur MariaDB

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
