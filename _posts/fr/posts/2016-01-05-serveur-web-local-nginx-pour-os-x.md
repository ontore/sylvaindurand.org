---
title: Serveur web local<br/> <em>Nginx</em> pour OS X
lang: fr
---

Cherchant une solution satisfaisante pour créer un serveur web local permettant de programmer sous OS X avec PHP et MySQL, j'ai été déçu de constater que les solutions «~clés en main~» étaient loin d'égaler les WAMP  qui peuvent exister sous Windows.

C'était oublier qu'OS X est un système Unix, et que contrairement à Windows, il est parfaitement possible d'y créer un serveur local sur mesure avec quelques paquets.

Nous allons voir ici comment installer *Nginx*, PHP-FPM et *MariaDB* (MySQL) sous OS X *El Capitan* grâce au gestionnaire de paquets *Homebrew*.

## *Homebrew*

*HomeBrew* est un gestionnaire de paquets pour OS X, qui permet d'installer facilement les différentes applications Unix.

Pour l'installer, il suffit de saisir dans un terminal :

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Si vous ne les avez pas déjà, OS X vous proposera d'installer préalablement les outils en ligne de commande *Xcode*.

Après installation, la commande suivante permet, le cas échéant, de donner les indications permettant de parfaire l'installation :

```bash
brew doctor
```

On met ensuite à jour tous les paquets avec :

```bash
brew update
brew upgrade
```

## *Nginx*

Bien qu'une version d'*Apache* soit nativement fournie avec OS X, nous nous proposons ici d'installer *Nginx*, particulièrement léger et facilement configurable. Son installation se fait avec :

```bash
brew install nginx
```

Les commandes suivantes permettent de lancer automatiquement *Nginx* au démarrage :

```bash
sudo cp -v /usr/local/opt/nginx/*.plist /Library/LaunchDaemons/
sudo chown root:wheel /Library/LaunchDaemons/homebrew.mxcl.nginx.plist
```

Nous souhaitons pouvoir stocker notre site internet dans le dossier de notre choix, et y accéder à l'URL `http://localhost/`.

Pour cela, on édite le fichier de configuration :

```bash
nano /usr/local/etc/nginx/nginx.conf
```

Pour commencer, on édite la ligne commençant par `#user` pour donner à *Nginx* la permission d'accéder à nos fichiers, pour la modifier en la ligne suivante, où `<user>` est votre nom d'utilisateur :

```nginx
user <user> staff;
```

Pour utiliser le port 80, on change la ligne commençant par `listen` en :

```nginx
listen 80;
```

Enfin, on indique le dossier où vous souhaitez stocker vos sites grâce à la variable `root` :

```nginx
root <chemin/vers/votre/site>;
```

On peut désormais lancer *Nginx* avec :

```bash
sudo nginx
```

## PHP

Pour utiliser PHP avec *Nginx*, nous allons utiliser PHP-FPM. Pour installer PHP 5.6, on lance les commandes suivantes :

```bash
brew tap homebrew/php
brew install php56 --with-fpm --without-apache
```

Une fois installé, on utilise les commandes suivantes pour lancer PHP-FPM au démarrage :

```bash
sudo cp -v /usr/local/opt/php56/*.plist /Library/LaunchDaemons/
sudo chown root:wheel /Library/LaunchDaemons/homebrew.mxcl.php56.plist
```

Pour lancer manuellement PHP-FPM, on utilise la commande :

```bash
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.php56.plist
```

On peut s'assurer que le service tourne bien en regardant si la commande suivante produit bien un résultat :

```bash
lsof -Pni4 | grep LISTEN | grep php
```

Enfin, on édite à nouveau le fichier de configuration :

```bash
nano /usr/local/etc/nginx/nginx.conf
```

On modifie la ligne commençant par `index` par :

```nginx
index index.php;
```

Enfin, on ajoute dans la rubrique `server` les lignes suivantes pour faire fonctionner PHP pour tous les fichiers ayant l'extension `.php` :

```nginx
location ~ \.php$ {
    root  <chemin/vers/votre/site>;
    try_files $uri =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass 127.0.0.1:9000;
    include  fastcgi.conf;
}
```

On redémarre *Nginx* pour activer les changements :

```bash
sudo nginx -s reload
```

## MySQL

Plutôt que d'installer la version de *MySQL* réalisée par Oracle, nous allons installer le *fork* libre MariaDB avec les commandes suivantes :

```bash
brew install mariadb
unset TMPDIR
mysql_install_db --user=mysql --basedir=$(brew --prefix mariadb)
PATH="/usr/local/bin:/usr/local/sbin:$PATH"
```

Les commandes suivantes permettent de lancer automatiquement *Nginx* au démarrage :

```bash
sudo mysql.server start
```

Les lignes suivantes permettent de démarrer le serveur au démarrage :

```bash
sudo cp -v /usr/local/opt/mariadb/*.plist /Library/LaunchDaemons/
sudo chown root:wheel /Library/LaunchDaemons/homebrew.mxcl.mariadb.plist
```

Enfin, on finalise l'installation en choisissant un mot de passe `root` pour MySQL :

```bash
mysql_secure_installation
```
