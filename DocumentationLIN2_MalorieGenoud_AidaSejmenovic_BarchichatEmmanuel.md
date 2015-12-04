#   Configuration d'un hébergement de site Web

Avant de faire une quelconque installation, il faut disposer d'une machine Linux sous la distribution Debian 8.0.

Pour procéder aux différentes installations, il faut être connecté en **root**.

Il faut maintenant modifier le **umask** afin que l'on retrouve les droits suivants pour les dossiers et les fichiers pour les futurs utilisateurs:

dossiers : `  rwx --- ---   `

fichiers : `   rw- --- --- `

Ceci permettra de faire en sorte qu'un utilisateur n'ait pas accès ax dossiers et fichiers d'un autre utilisateur.

Voici la commande à taper pour changer le **umask**:

`   ~# umask 077   `


##   Installer et configurer le serveur OpenSSH

Pour installer le serveur **OpenSSH** il faut utiliser la commande suivante:

`   ~# apt-get install openssh-server   `

Si le système demande un disque **jessie** , depuis root éditez le fichier suivant:

`   ~# /etc/apt/sources.list   `

Il faut commenter la dernière ligne en mettant un # au début et ajouter les deux lignes suivantes:

`   deb http://nginx.org/packages/debian/ jessie nginx   `

`   deb-src http://nginx.org/packages/debian/ jessie nginx   `

Il faut sauvegarder le fichier et lancer la commande suivante:

`   ~# apt-get update   `

Cette commande va permettre de synchroniser les packages pour éviter d'avoir des soucis lors d'une installation.

Vous pouvez maintenant faire l'installation de **OpenSSH** avec la commande suivante:

`   ~# apt-get install openssh-server   `



##   Installer et configurer Nginx

Pour installer **Nginx** il faut utiliser la commande suivante: 

`   ~# apt-get install nginx   `

Maintenant on va démarrer le service: 

`   ~# service nginx start   `

On peut vérifier nos informations en pointant un moteur de recherche sur notre adresse IP, il devrait confirmer que nginx a bien été installé. Pour trouver notre VPS IP adresse : 

`   ~# ifconfig eth0 | grep inet | awk '{ print $2 }'   `

On ouvre le host virtuel avec la comande suivante: 

`   ~# sudo nano /etc/nginx/sites-available/default   `

Il faut changer certaines lignes du fichier, notamment le nom de votre serveur, par exemple **cpnv.webapp**.
Voilà ce que vous devriez avoir à peu près : 
    
    [...]
    server {
            listen   80;
            
            root /usr/share/nginx/www;
            index index.php index.html index.htm;

            server_name cpnv.webapp;

            location / {
                   try_files $uri $uri/ /index.html;
            }

            error_page 404 /404.html;

            error_page 500 502 503 504 /50x.html;
            location = /50x.html {
                   root /usr/share/nginx/www;
            }

            # pass the PHP scripts to FastCGI server listening on /var/run/php5-fpm.sock
            location ~ \.php${
                    try_files $uri =404;
                    fastcgi_pass unix:/var/run/php5-fpm.sock;
                    fastcgi_index index.php;
                    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                    include fastcgi_params;  
             }
    }
    [...]



##   Installer et configurer PHP-FPM

Pour installer **PHP-FPM** il faut utiliser la commande suivante:

`   ~# apt-get install php5-fpm php5-mysql   `

Nous allons maintenant nous occuper de configurer **PHP-FPM**. Pour ce faire, il faut faire un petit changement dans pe fichier **php.ini**. Voici la commande à taper:

`   ~# nano /etc/php5/fpm/php.ini   `

Une fois dans le fichier il faut rechercher la ligne `   cgi.fix_pathinfo=1   `. Il faut changer cette ligne en `   cgi.fix_pathinfo=0   `. 
On change la valeur à **0** car cela permet d'avoir une meilleur sécurité au niveau de l'interpréteur PHP et change sa manière dont il va traiter les données qu'il recevra du serveur **NGNIX**.

Il y a une dernière vérification à faire avant de passer à la suite. Utilisez la commande suivante:

`   ~# nano /etc/php5/fpm/pool.d/www.conf   `

Vous devez chercher et vérifier si la ligne `   listen = 127.0.0.1:9000   `  existe. 
Normalement sur les dernières versions de **PHP-FPM** la ligne est écrite ainsi: 

`   listen = /var/run/php5-fpm.sock   `

Ceci permettra de faire en sorte qu'il ne va pas "écouter" sur le réseau local mais sur son propre réseau.

Afin que **PHP-FPM** prenne en charge toutes les modifications faites précédemment, nous allons le redémarrer. Voici la commande à taper:

`   ~# service php5-fpm restart   `

Maintenant que tout a été configuré, nous pouvons créer un fichier d'informations. Voici la commande à taper:

`   ~# touch /usr/share(ngnix/www/info.php   `   

`   ~# nano /usr/share/nginx/www/info.php   `

Écrivez la ligne suivante dans ce fichier:

`   <?php   phpinfo()   ?>   `

Redémarrez le serveur **NGINX** avec la commande suivante:

`   ~# service nginx restart   `



##   Installer et configurer MariaDB


Avant d'installer **MariaDB**, il faut installer le package **"python-software-properties"**:

`   ~# apt-get install python-software-properties   `

Nous n'allons pas installer de clé **GPG**, mais au cas où voici un lien proposant diverses clé **GPG**:

[Clé GPG](http://korben.info/keyserver-ubuntu-com-inaccessible-que-faire.html )

Afin d'éviter qu'une erreur se fasse lors de l'ajout du répertoire officiel de **MariaDB**, voici la commande à taper:

`   ~# apt-get install software-properties-common   `

Maintenant, vous pouvez procéder à l'installation du répertoire:

`   add-apt-repository 'deb http://mariadb.biz.net.id//repo/10.1/debian sid main'   `

Nous allons faire une nouvelle mise à jour des packages avant d'installer la base de données:

`   ~# apt-get update   `

Voici la commande pour installer la base de données:

`   ~# apt-get install mariadb-server mariadb-client   `

Si l'installation se passe bien, vous pouvez vérifier la vesrion de **MariaDB**:

`   ~# mysql -V   `

Voici ce que vous devriez obtenir:

`   mysql  Ver 15.1 Distrib 5.5.38-MariaDB, for debian-linux-gnu (x86_64) using readline 5.1   `

Il faut se connecter à la base de données en tant que **root** afin de pouvoir créer un utilisateur

`   ~# mysql -u root -p   `

Commande pour créer un utilisateur:

`  MariaDB [(none)]> CREATE USER 'tux' IDENTIFIED BY 'tuxounet';   `

Maintenant quitter la base de données et connectez-vous avec l'utilisateur créé:

`  MariaDB [(none)] exit   `

`   ~# mysql -u tux -p   `








## Création du répertoire pour les utilisateurs

Il faut créer un répertoire quiregroupera tous les sites Web des utilisateurs. Il faut utiliser la commande suivante:

`   mkdir /home/Applications   `

Ce répertoire va répertorier les webapp. Voici un exemple de webapp:

`   mkdir /home/Applications/tuxwebapp   `

Maintenant on va attribuer le répertoire à l'utilisateur avec la commande suivante:

`   ~# chown tux /home/Applications/tuxwebapp   `



##   Source d'installation et de configuration


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


