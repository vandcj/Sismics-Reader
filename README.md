# Sismics-Reader
Installer son propre agrégateur de flux RSS [ DEBIAN ]

# **Installation**

Dans un premier temps, nous allons installer Java qui est nécessaire pour faire tourner Sismics Reader.

    sudo apt update

    sudo apt install openjdk-8-jre

Nous allons ensuite récuperer le fichier .deb et l’installer.

    wget https://downloads.sourceforge.net/project/sismicsreader/release/1.4/reader-1.4.deb

puis

    sudo dpkg -i reader-1.4.deb

Une fois cela fait, le reader est disponible à cette adresse : http://adresseipduserveurlan:4001

Pour vous connecter la première fois, il faudra utiliser le login/MDP suivant : admin/admin.

Changer le mot de passe pour le compte administrateur, je vous déconseille de cocher l’UPnP et créez ensuite votre premier utilisateur. Vous pouvez ensuite vous connectez avec et soit importer vos sites via un fichier OPLM, soit les ajouter à la main.


# **Sécurisation de l'installation**

Avant d’ouvrir le site à l’extérieur, il est nécessaire de le sécuriser. Pour cela, nous allons changer l’utilisateur qui exécute l’application, il s’agit par défaut du root. Et nous allons changer le port par défaut pour accéder à la page web.

Dans un premier temps, créez un utilisateur avec un mot de passe fort.

    sudo adduser reader

Nous allons ensuite interdire l’accès au bash à cet utilisateur, ce qui empêchera quiconque de se connecter avec via un terminal, le seul but de ce dernier étant de faire tourner l’application.

    sudo nano /etc/passwd

Vous devriez trouver un ligne du genre :

    reader:x:1xxx:1xxx:reader,,,:/home/reader:/bin/bash

Il vous suffit de la modifier comme cela :

    reader:x:1xxx:1xxx:reader,,,:/home/reader:/bin/false

Nous allons ensuite modifier les droit sur le dossier de l’application pour autoriser notre utilisateur à faire tourner Sismics :

    cd /var/

    chown -R root:reader reader/

    chmod -R g+w reader/

Il faut maintenant aller dans le fichier de configuration de Sismics Reader et modifier la configuration pour coller à nos modifications.

    nano /etc/default/reader

Et modifier les deux arguments comme cela, mettez le port de votre choix, attention à ne pas en prendre un standardisé ou déjà utilisé ailleurs :

    READER_ARGS= »–port=xxxx –max-memory=150″

    READER_USER=reader

Il ne reste plus qu’à relancer l’application :

    service reader restart

Et voilà vous pouvez maintenant passez par l’adresse via le nouveau port :

    http://adresseipduserveurlan:xxxx
    
    
    
    
# **Mise en place vers l'extérieur**

Maintenant le plus gros, nous allons faire en sorte d’accéder de manière sécurisé à notre Sismics Reader via HTTPS.

Dans un premier temps, nous allons mettre votre nom de domain pour y accéder.

    sudo nano /etc/hosts

Il faudra ensuite rajouter une ligne tel que ça :

    127.0.1.1 serveur.domain.tld serveur
    127.0.0.1 localhost

Vous pouvez ensuite tester en accédant au reader via son nom :

    http://serveur.domain.tld:xxxx

Pensez à bien faire le rajout de ce nom d’hôte chez votre hébergeur de domaine en pointant cet nouvelle entrée DNS vers l’IP de votre serveur.

Une fois que vous validez cette étape, nous allons mettre en place un serveur apache pour permettre d’accéder à l’application sans devoir spécifier le port.

    sudo apt install apache2

Et nous allons activer tout les plugins nécessaire pour notre configuration :

    sudo a2enmod proxy_http && sudo service apache2 restart

Nous allons ensuite aller créer le fichier de configuration du site (⚠ ne pas oublier le .conf sinon apache ne le prendra pas en compte) :

    cd /etc/apache2/sites-available/

    sudo nano serveur.conf

Et il vaudra le remplir comme ceci :

    <VirtualHost *:80>
    ServerName serveur.domain.tld (⚠ identique à ce que vous avez mis dans le /etc/hosts)

    ProxyPreserveHost On
    ProxyRequests Off

    ProxyPass / http://127.0.0.1:xxxx/ (⚠ numéro du port spécifier dans la partie configuration)
    ProxyPassReverse / http//127.0.0.1:xxxx/ (⚠ numéro du port spécifier dans la partie configuration)
    </VirtualHost>

Il ne vous reste plus qu’à enregistrer le fichier et activer le site, puis redémarrer apache pour prendre en compte ce changement

sudo a2enssite serveur.conf

sudo apachectl restart

Vous pouvez maintenant tester l’accès au site sans devoir spécifier le port :

    http://serveur.domain.tld

