# Guide de Configuration d'un Serveur Nginx sur Ubtuntu

## Étape 1: Installation de Nginx

### Sur Debian/Ubuntu
1. **Mettre à jour le système:**
```bash
   sudo apt update
   sudo apt upgrade
```
2. **Installer Nginx**

```bash 
    sudo apt install nginx
```
3. **Vérifierl'installation:**
```bash
    nginx -v
```
### Sur CentOS/RHEL
1. **Mettre à jour le système:**
```bash
    sudo yum update
```
2. **Installer Nginx**

```bash 
    sudo yum install epel-release
    sudo yum install nginx

```
3. **Vérifierl'installation:**
```bash
    nginx -v
```

## Étape 2: Démarrer et activer Nginx
1. **Démarrer Nginx:**
```bash 
    sudo systemctl start nginx
```
2. **Activer Nginx au démarrage**
```bash 
    sudo systemctl enable nginx
```

## Étape 3: Configurer le pare-feu
### Sur Debian/Ubuntu avec UFW
1. **Autoriser le trafic HTTP et HTTPS:**
```bash
    sudo ufw allow 'Nginx Full'
```
###   Sur CentOS/RHEL avec firewalld
1. **Autoriser le trafic HTTP et HTTPS:**
```bash
    sudo firewall-cmd --permanent --zone=public --add-service=http
    sudo firewall-cmd --permanent --zone=public --add-service=https
    sudo firewall-cmd --reload
```

## Étape 4: Configuration de base de Nginx
1.**Fichiers de configuration:**

Le fichier principal de configuration est situé à **/etc/nginx/nginx.conf**.
Cependant, il est souvent plus pratique de gérer les configurations de sites individuels
dans le répertoire **/etc/nginx/sites-available/** et de créer des liens symboliques vers **/etc/nginx/sites-enabled/**.


2. **Créer un fichier de configuration pour votre site:**

```bash
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
```bash
    sudo ln -s /etc/nginx/sites-available/mon_site /etc/nginx/sites-enabled/
```
5. **Vérifier la configuration:**
```bash
    sudo nginx -t
```
6. **Redémarrer Nginx:**
```bash
    sudo systemctl reload nginx
```


## Étape 5: Déployer votre site web 

1. **Créer le répertoire de votre site:** 
```bash
    sudo mkdir -p /var/www/mon_site
```
2. **Définir les permissions:**
```bash
    sudo chown -R $USER:$USER /var/www/mon_site
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
```bash
   sudo apt update
   sudo apt upgrade
```
2. **Installer PHP-FPM**

```bash 
    sudo apt install php-fpm
```

### Sur CentOS/RHEL

1. **Mettre à jour le système:**
```bash
   sudo yum update
```
2. **Installer PHP-FPM**

```bash 
    sudo yum install php-fpm
```