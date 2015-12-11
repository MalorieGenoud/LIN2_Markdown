#   Configuration d'un hébergement de site Web

Avant de faire une quelconque installation, il faut disposer d'une machine Linux sous la distribution Debian 8.0.

Pour procéder aux différentes installations, il faut être connecté en **root**.

Dans un premier temps, nous allons modifier quelques lignes afin d'avoir les mêmes packages qui se synchronisent. On va éditer le fichier **sources.list** :

`   ~# nano /etc/apt/sources.list   `

Il faut commenter la dernière ligne en mettant un **#** au début et ajouter les lignes suivantes :

`   deb http://nginx.org/packages/debian/ jessie nginx   `

`   deb-src http://nginx.org/packages/debian/ jessie nginx   `

`   deb http://ftp.ch.debian.org/debian/ jessie main contrib non-free   `

`   deb-src http://ftp.ch.debian.org/debian/ jessie main contrib non-free   `

Il faut sauvegarder le fichier et lancer la commande suivante afin de faire la mise à jour de la liste ainsi que des packages :

`   ~# apt-get update   `

`   ~# apt-get upgrade   `

Ces commandes éviterons quelques soucis lors de l'installation des différents services à l'avenir.



##   Installer et configurer le serveur OpenSSH

Commençons par installer **OpenSSH** avec la commande suivante :

`   ~# apt-get install openssh-server   `

Maintenant on va faire une petite configuration du serveur car avec les dernière versions, il y a déjà une bonne configuration au niveau de la sécurité.

On va commencer par éditer le fichier **sshd_config** :

`   ~# nano /etc/ssh/sshd_config   `

On va modifier la ligne `   PermitRootLogin   without-password` et remplacer le paramètre `  without-password   ` par le paramètre `   no   ` afin d'éviter une quelconque connexion avec l'utilisateur **root**.

Une fois le fichier sauvegarder, on va rechercher la configuration ssh :

`   ~# /etc/init.d/ssh reload   `



##   Installer et configurer Nginx

Pour installer **Nginx** il faut utiliser la commande suivante : 

`   ~# apt-get install nginx   `

Maintenant on va démarrer le service : 

`   ~# service nginx start   `

On peut vérifier nos informations en pointant un moteur de recherche sur notre adresse IP, il devrait confirmer que **Nginx** a bien été installé. Pour trouver notre VPS IP adresse il faut taper la commande suivante : 

`   ~# ifconfig eth0 | grep inet | awk  '{ print $2 }'   `

Pour la configuration, on ouvre le host virtuel avec la commande suivante : 

`   ~# nano /etc/nginx/conf.d/default .conf  `

Il faut changer certaines lignes du fichier, notamment le nom du serveur qui est actuellement `   localhost   `. Il faut donc le changer par le votre, par exemple **cpnv.webapp**.
Voilà ce que vous devriez avoir à peu près : 

      [...]
      server {
      listen       80;
      server_name  cpnv.webapp;
  
      #charset koi8-r;
      #access_log  /var/log/nginx/log/host.access.log  main;
  
      location / {
          root   /usr/share/nginx/html;
          index index.php index.html index.htm index.nginx-debian.html;
      }
  
      #error_page  404              /404.html;
  
      # redirect server error pages to the static page /50x.html
      #
      error_page   500 502 503 504  /50x.html;
      location = /50x.html {
          root   /usr/share/nginx/html;
      }
  
      # proxy the PHP scripts to Apache listening on 127.0.0.1:80
      #
      #location ~ \.php$ {
      #    proxy_pass   http://127.0.0.1;
      #}
  
      # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
      #
      #location ~ \.php$ {
      #    root           html;
      #    fastcgi_pass   127.0.0.1:9000;
      #    fastcgi_index  index.php;
      #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
      #    include        fastcgi_params;
      #}
  
      # deny access to .htaccess files, if Apache's document root
      # concurs with nginx's one
      #
      #location ~ /\.ht {
      #    deny  all;
      #}
      }
      [...]



##   Installer et configurer PHP-FPM

Pour installer **PHP-FPM** il faut utiliser la commande suivante :

`   ~# apt-get install php5-fpm php5-mysql   `

Nous allons maintenant nous occuper de configurer **PHP-FPM**. Pour ce faire, il faut faire un petit changement dans le fichier **php.ini**. Voici la commande à taper :

`   ~# nano /etc/php5/fpm/php.ini   `

Une fois dans le fichier il faut rechercher la ligne `   cgi.fix_pathinfo=1   `. Il faut changer cette ligne en `   cgi.fix_pathinfo=0   `. 
On change la valeur à **0** car cela permet d'avoir une meilleur sécurité au niveau de l'interpréteur PHP et change sa manière dont il va traiter les données qu'il recevra du serveur **Nginx**.

Il y a une dernière vérification à faire avant de passer à la suite. Utilisez la commande suivante :

`   ~# nano /etc/php5/fpm/pool.d/www.conf   `

Vous devez chercher et vérifier si la ligne `   listen = 127.0.0.1:9000   `  existe. 
Normalement sur les dernières versions de **PHP-FPM** la ligne est écrite ainsi : 

`   listen = /var/run/php5-fpm.sock   `

Ceci permettra de faire en sorte qu'il ne va pas "écouter" sur le réseau local mais sur son propre réseau.

Afin que **PHP-FPM** prenne en charge toutes les modifications faites précédemment, nous allons le redémarrer. Voici la commande à taper :

`   ~# service php5-fpm restart   `

Maintenant que tout a été configuré, nous pouvons créer un fichier d'informations. Voici la commande à taper :

`   ~# touch /usr/share/ngnix/html/info.php   `   

`   ~# nano /usr/share/nginx/html/info.php   `

Il est possible qu'à la place du dossier `   html   ` vous ayez un dossier nommer `   www   `. Mais le principe reste le même.

Écrivez la ligne suivante dans ce fichier :

`   <?php   phpinfo()   ?>   `

Redémarrez le serveur **Nginx** avec la commande suivante :

`   ~# service nginx restart   `



##   Installer et configurer MariaDB

Avant d'installer **MariaDB**, il faut installer le package **python-software-properties** afin d'assurer le bon fonctionnement de la base de données :

`   ~# apt-get install python-software-properties   `

Nous n'allons pas installer de clé **GPG** car ceci n'a pas de réel impact sur l'installation, mais au cas où voici un lien proposant diverses clé **GPG** :

[Clé GPG](http://korben.info/keyserver-ubuntu-com-inaccessible-que-faire.html )

Afin d'éviter qu'une erreur se fasse lors de l'ajout du répertoire officiel de **MariaDB**, voici la commande à taper :

`   ~# apt-get install software-properties-common   `

Maintenant, vous pouvez procéder à l'installation du répertoire qui va rechercher les différents package de **MariaDB** :

`   add-apt-repository 'deb http://mariadb.biz.net.id//repo/10.1/debian jessie main'   `

Nous allons faire une nouvelle mise à jour de la liste et des packages avant d'installer la base de données :

`   ~# apt-get update   `

`   ~# apt-get upgrade  `

Voici la commande pour installer la base de données du côté serveur et du côté client :

`   ~# apt-get install mariadb-server mariadb-client   `

À la suite de ça une fenêtre va s'ouvrir et demander l'ajout d'un mot de passe. Il est important de se souvenir de ce mot de passe car il sera réutilisé plus tard pour la connexion à la base de données.

Exemple de mot de passe : `   D08x97H$   `

Si l'installation se passe bien, vous pouvez vérifier la version de **MariaDB** afin d'être sûr que vous disposiez de la dernière version :

`   ~# mysql -V   `

Voici ce que vous devriez obtenir :

`   mysql  Ver 15.1 Distrib 10.0.22-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2   `

###   Utilisation des diverses commandes dans MariaDB

Nous allons apprendre à utiliser les commandes de **MariaDB**.

Tout d'abord, il faut se connecter à la base de données en tant que **root** afin de pouvoir créer un utilisateur :

`   ~# mysql -u root -p   `

Une fois cette commande taper, un mot de passe va être demandé. 

**ATTENTION LE MOT DE PASSE N'EST PAS LE MOT DE PASSE DE L'UTILISATEUR ROOT MAIS CELUI QUI A ÉTÉ MIS LORS DE L'INSTALLATION DE MARIADB**

Commande pour créer un utilisateur :

`  MariaDB [(none)]> CREATE USER 'tux' IDENTIFIED BY 'tuxounet';   `

Maintenant quittez la base de données et connectez-vous avec l'utilisateur créé :

`  MariaDB [(none)] exit   `

`   ~# mysql -u tux -p   `

Une fois connectez avec l'utilisateur créer précédemment, on va afficher les différentes bases de données actuelles :

`   MariaDB[(none)]> SHOW DATABASES;   `

Voici le résultat obtenu:

     +--------------------+ 
     | Database           | 
     +--------------------+ 
     | information_schema | 
     | mysql              | 
     | performance_schema | 
     +--------------------+ 
     3 rows in set (0.04 sec)

On va maintenant travailler sur la base de données **mysql** :

`   MariaDB[(none)]> USES mysql;   `

Voici le résultat obtenu :
    
     Reading table information for completion of table and column names
     You can turn off this feature to get a quicker startup with -A 
    
     Database changed 
     MariaDB [mysql]>

Pour afficher les tables de la bases de données, utilisez la commande suivante :

`   MariaDB [mysql]> SHOW TABLES;    `

Voici le résultat obtenu :

     | Tables_in_mysql           | 
     +---------------------------+ 
     | columns_priv              | 
     | db                        | 
     | event                     | 
     | func                      | 
     | general_log               | 
     | help_category             | 
     | help_keyword              | 
     | help_relation             | 
     | help_topic                | 
     .....
     24 rows in set (0.00 sec)
     
Pour afficher toutes les colonnes de la table **user** par exemple de la base de données **mysql**, voici une des deux commandes que l'on peut utiliser:

`   MariaDB [mysql]> SHOW COLUMNS FROM user;   `

ou 

`   MariaDB[mysql]> DESCRIBE user;   `

Voici le résultat obtenu avec l'une de ces commandes :

     +------------------------+-----------------------------------+------+-----+---------+-------+ 
     | Field                  | Type                              | Null | Key | Default | Extra | 
     +------------------------+-----------------------------------+------+-----+---------+-------+ 
     | Host                   | char(60)                          | NO   | PRI |         |       | 
     | User                   | char(16)                          | NO   | PRI |         |       | 
     | Password               | char(41)                          | NO   |     |         |       | 
     | Select_priv            | enum('N','Y')                     | NO   |     | N       |       | 
     | Insert_priv            | enum('N','Y')                     | NO   |     | N       |       | 
     | Update_priv            | enum('N','Y')                     | NO   |     | N       |       | 
     | Delete_priv            | enum('N','Y')                     | NO   |     | N       |       | 
     | Create_priv            | enum('N','Y')                     | NO   |     | N       |       | 
     | Drop_priv              | enum('N','Y')                     | NO   |     | N       |       | 
     .......
     42 rows in set (0.01 sec)

Pour afficher le status du seveur, voici la commande à utiliser :

`  MariaDB [mysql]> SHOW STATUS;    `

Voici le résultat obtenu :

     +------------------------------------------+----------------------+ 
     | Variable_name                            | Value                | 
     +------------------------------------------+----------------------+ 
     | Aborted_clients                          | 0                    | 
     | Aborted_connects                         | 0                    | 
     | Access_denied_errors                     | 0                    | 
     | Aria_pagecache_blocks_not_flushed        | 0                    | 
     | Aria_pagecache_blocks_unused             | 15737                | 
     | Aria_pagecache_blocks_used               | 2                    | 
     | Aria_pagecache_read_requests             | 176                  | 
     | Aria_pagecache_reads                     | 4                    | 
     | Aria_pagecache_write_requests            | 8                    | 
     ....
     419 rows in set (0.00 sec)
     
Voici la commande à faire pour créer une base de données :

`   MariaDB [mysql]> SHOW CREATE DATABASE db;   `

Voici le résultat obtenu :

     +----------+------------------------------------------------------------------+ 
     | Database | Create Database                                                  | 
     +----------+------------------------------------------------------------------+ 
     | db   | CREATE DATABASE `db` /*!40100 DEFAULT CHARACTER SET latin1 */ | 
     +----------+------------------------------------------------------------------+ 
     1 row in set (0.00 sec)
     
On va maintenant créer une table **user** dans cette base de données :

`   MariaDB [mysql]> SHOW CREATE TABLE user;   `

Voici le résultat obtenu :

      +
      | Table | Create Table
      +-------
      | user  | CREATE TABLE `user` ( 
         `Host` char(60) COLLATE utf8_bin NOT NULL DEFAULT '', 
         `User` char(16) COLLATE utf8_bin NOT NULL DEFAULT '', 
         `Password` char(41) CHARACTER SET latin1 COLLATE latin1_bin NOT NULL DEFAULT '', 
         `Select_priv` enum('N','Y') CHARACTER SET utf8 NOT NULL DEFAULT 'N', 
         `Insert_priv` enum('N','Y') CHARACTER SET utf8 NOT NULL DEFAULT 'N', 
      ....

Tapez la commande suivante pour voir les droits sur un ou tous les utilisateurs :

`   MariaDB [mysql]> SHOW GRANTS;   `

Voici le résultat obtenu :

     +----------------------------------------------------------------------------------------------------------------------------------------+ 
     | Grants for root@localhost                                                                                                              | 
     +----------------------------------------------------------------------------------------------------------------------------------------+ 
     | GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*698vsgfkemhvjh7txyD863DFF63A6bdfj8349659232234bs3bk5DC1412A' WITH GRANT OPTION | 
     | GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION                                                                           | 
     +----------------------------------------------------------------------------------------------------------------------------------------+ 
     2 rows in set (0.00 sec)

Tapez la commande suivante pour voir les différents messages d’attention de mariadb serveur :

`   MariaDB [mysql]> SHOW WARNINGS;   `

Voici le résultat obtenu :
     +--------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
     | Level | Code |Message                                                                                                                                                      | 
     +-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
     | Error | 1064 | You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'ON mysql' at line 1 | 
     +-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
     1 row in set (0.00 sec)
     
Tapez la commande suivante pour voir les  erreurs sur le serveur **MariaDB** :

`   MariaDB [mysql]> SHOW ERRORS;   ` 

Voici le résultat obtenu :

     +-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
     | Level | Code | Message                                                                                                                                                      | 
     +-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
     | Error | 1064 | You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'ON mysql' at line 1 | 
     +-------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------------+ 
     1 row in set (0.00 sec)




##   Création du répertoire pour les utilisateurs et gestions des droits

Il faut créer un répertoire qui regroupera tous les sites Web des utilisateurs. Il faut utiliser la commande suivante :

`   mkdir /home/Applications   `

Ce répertoire va répertorier les webapp. Voici un exemple de webapp :

`   mkdir /home/Applications/tuxwebapp   `

Maintenant on va attribuer le répertoire à l'utilisateur avec la commande suivante :

`   ~# chown tux /home/Applications/tuxwebapp   `

Il faut maintenant modifier le **umask** afin que l'on retrouve les droits suivants pour les dossiers et les fichiers pour les futurs utilisateurs :

dossiers : `  rwx --- ---   `

fichiers : `   rw- --- --- `

Ceci permettra de faire en sorte qu'un utilisateur n'ait pas accès aux dossiers et fichiers d'un autre utilisateur.

Voici la commande à taper pour changer le **umask** :

`   ~# umask 077   `

Voici les différentes étapes pour la création d'un utilisateur :

     ~# adduser tux
      Adding user `tux' ...
      Adding new group `tux' (1001) ...
      Adding new user `tux' (1001) with group `tux' ...
      Creating home directory `/home/tux' ...
      Copying files from `/etc/skel' ...
      Entrez le nouveau mot de passe UNIX : (nouveau mot de passe)
      Retapez le nouveau mot de passe UNIX : (mot de passe répété)
      passwd : le mot de passe a été mis à jour avec succès
      Changing the user information for tux
      Enter the new value, or press ENTER for the default
      Full Name []: tux
      Room Number []: 
      Work Phone []: 
      Home Phone []: 
      Other []: 
      Is the information correct? [Y/n] y
      administrateur@ordinateur:~$ _


##   Source d'installation et de configuration



## Liens pour l'installation des packages

[Lien 1](http://linuxconfig.org/debian-apt-get-jessie-sources-list)
[Lien 2](http://nginx.org/en/linux_packages.html)



###   OpenSSH

[Lien 1](https://openclassrooms.com/courses/installation-et-utilisation-d-un-serveur-ssh-sous-debian-etch)

[Lien 2](http://doc.ubuntu-fr.org/tutoriel/reverse_ssh)

[Lien 3](http://www.linuxtricks.fr/wiki/ssh-installer-et-configurer-un-serveur-ssh#paragraph_installation-du-serveur-openssh)

[Lien 4](http://doc.ubuntu-fr.org/ssh) 



###   Ngnix

[Lien 1](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-debian-7)



###   PHP-FPM

[Lien 1](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-debian-7) 



###   MariaDB

[Lien 1](https://mariadb.com/kb/en/mariadb/configuring-mariadb-for-remote-client-access/)