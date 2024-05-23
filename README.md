# Guide de Configuration d'un Serveur Nginx sur Ubtuntu

## Étape 1: Installation de Nginx

### Sur Debian/Ubuntu
1. **Mettre à jour le système:**
```console
   sudo apt update
```
```console
   sudo apt upgrade
```
2. **Installer Nginx**

```console 
    sudo apt install nginx
```
3. **Vérifierl'installation:**
```console
    nginx -v
```
### Sur CentOS/RHEL
1. **Mettre à jour le système:**
```console
    sudo yum update
```
2. **Installer Nginx**

```console 
    sudo yum install epel-release
```
```console 
    sudo yum install nginx
```
3. **Vérifierl'installation:**
```console
    nginx -v
```

## Étape 2: Démarrer et activer Nginx
1. **Démarrer Nginx:**
```console 
    sudo systemctl start nginx
```
2. **Activer Nginx au démarrage**
```console 
    sudo systemctl enable nginx
```

## Étape 3: Configurer le pare-feu
### Sur Debian/Ubuntu avec UFW
1. **Autoriser le trafic HTTP et HTTPS:**
```console
    sudo ufw allow 'Nginx Full'
```
###   Sur CentOS/RHEL avec firewalld
1. **Autoriser le trafic HTTP et HTTPS:**
```console
    sudo firewall-cmd --permanent --zone=public --add-service=http
```
```console
    sudo firewall-cmd --permanent --zone=public --add-service=https
```
```console
    sudo firewall-cmd --reload
```
## Étape 4: Configuration de base de Nginx
1.**Fichiers de configuration:**

Le fichier principal de configuration est situé à **/etc/nginx/nginx.conf**.
Cependant, il est souvent plus pratique de gérer les configurations de sites individuels
dans le répertoire **/etc/nginx/sites-available/** et de créer des liens symboliques vers **/etc/nginx/sites-enabled/**.


2. **Créer un fichier de configuration pour votre site:**

```console
    sudo nano /etc/nginx/sites-available/mon_site
```

3. **Exemple de configuration de base:**
```bash
    server {
        listen 80;
        server_name mon_site.com www.mon_site.com;

        root /var/www/mon_site;
        index index.html index.htm index.nginx-debian.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
```
4. **Activer la configuration:**
```console
    sudo ln -s /etc/nginx/sites-available/mon_site /etc/nginx/sites-enabled/
```
5. **Vérifier la configuration:**
```console
    sudo nginx -t
```
6. **Redémarrer Nginx:**
```console
    sudo systemctl reload nginx
```


## Étape 5: Déployer votre site web 

1. **Créer le répertoire de votre site:** 
```console
    sudo mkdir -p /var/www/mon_site
```
2. **Définir les permissions:**
```console
    sudo chown -R www-data:www-data /var/www/mon_site
```
```console
    sudo chmod -R 755 /var/www/mon_site
```
3. **Placer votre contenu dans le répertoire:**
```php
    echo "<html><h1>Bienvenue sur mon site</h1></html>" > /var/www/mon_site/index.html
```


# Guide de Configuration de FastCGI avec Nginx

## Étape 1: Installation des Paquets Nécessaires si PHP n'est pas installer sur votre serveur

### Sur Debian/Ubuntu

1. **Mettre à jour le système:**
```console
   sudo apt update
```
```console
   sudo apt upgrade
```
2. **Installer PHP-FPM**

```console 
    sudo apt install php-fpm
```

### Sur CentOS/RHEL

1. **Mettre à jour le système:**
```console
   sudo yum update
```
2. **Installer PHP-FPM**

```console 
    sudo yum install php-fpm
```


# Étape 2: Configurer PHP-FPM
1. **Modifier le fichier de configuration de PHP-FPM:**
Le fichier de configuration est généralement situé à /etc/php/8.1/fpm/php-fpm.conf (sur Debian/Ubuntu) ou /etc/php-fpm.d/www.conf (sur CentOS/RHEL).  
Assurez-vous que listen est configuré pour utiliser un socket Unix ou un port TCP. Par exemple:

```ini
    listen = /run/php/php8.1-fpm.sock
```

Soyez conscient que Sesi est pour PHP8.1. Le nom peut différer en fonction de la version PHP que vous utilisez.

2. **Démarrer et activer PHP-FPM:**
```console
    sudo systemctl start php8.1-fpm
```
```console
    sudo systemctl enable php8.1-fpm
```

# Étape 3: Configurer Nginx pour Utiliser FastCGI

1. **Créer un fichier de configuration pour votre site:** 

```console
    sudo nano /etc/nginx/sites-available/mon_site
```

2. **Ajouter la configuration FastCGI:**
Voici un exemple de configuration pour servir des fichiers PHP:
Dans les dossiers de votre site, créez un dossier api qui contiendra le code à appeler.

```ini
    server {
        listen 80;
        server_name www.mon_site.com;

        root /var/www/mon_site;
        index index.php index.html index.htm;

        location / {
            try_files $uri $uri/ =404;
        }

        location /api/ {
            try_files $uri $uri/ /index.php?$query_string;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_index index.cgi;
            include fastcgi_params;
            fastcgi_pass unix:/var/run/fcgiwrap.socket;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php8.1-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.ht {
            deny all;
        }
    }

```
3. **Activer la configuration:**
```console
    sudo ln -s /etc/nginx/sites-available/mon_site /etc/nginx/sites-enabled/
```

4. **Vérifier la configuration de Nginx:**
```console
    sudo nginx -t
```

5. **Redémarrer Nginx:**
```console
    sudo systemctl reload nginx
```

# Étape 5: Déployer l'Application FastCGI
