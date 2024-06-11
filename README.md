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

# Gide explicatif:  Directives, paramètres et variables FastCGI importants

### **Directives FastCGI communes**

Les directives servent à configurer un serveur FastCGI externe pour traiter les requêtes. En règle générale, on utilise des directives pour déterminer l'emplacement et les paramètres de ce serveur. Les configurations habituelles comprennent la définition des paramètres de connexion, l'ajout des paramètres nécessaires, la mise en place de caches pour améliorer les performances et la gestion des délais d'attente
 
- **fastcgi_index :** Précisez le fichier d'index à ajouter aux valeurs **$fastcgi_script_name** terminées par une barre oblique (/). Ceci est souvent utile si le paramètre SCRIPT_FILENAME est défini sur **$document_root$fastcgi_script_name** et que le bloc d'emplacement est configuré pour accepter les demandes avec des informations après le fichier.

- **fastcgi_pass :** La directive réelle qui oriente les requêtes dans le contexte actuel vers le backend. Cela indique l'endroit où le processeur FastCGI peut être atteint.

- **try_files :** Cela est fréquemment utilisé dans le cadre d'une routine de nettoyage de requête, pour vérifier l'existence du fichier demandé avant de le transmettre au processeur FastCGI.

- **include :** Habituellement, cela est utilisé pour inclure des détails de configuration communs et partagés dans plusieurs endroits.

- **fastcgi_param :** La directive de tableau peut être utilisée pour définir des paramètres sur des valeurs. Habituellement, cela est utilisé en parallèle avec des variables Nginx pour définir les paramètres FastCGI sur des valeurs spécifiques à la requête.

- **fastcgi_split_path_info :** Cette directive établit une expression régulière avec deux groupes capturés. La variable $fastcgi_script_name est utilisée comme valeur pour le premier groupe capturé. La variable **$fastcgi_path_info** est utilisée pour la valeur du deuxième groupe capturé. Ces deux éléments sont souvent utilisés pour analyser correctement la requête afin que le processeur sache quelles parties de la requête sont les fichiers à exécuter et quelles parties sont des informations supplémentaires à transmettre au script.

- **fastcgi_intercept_errors :** Cette directive précise si les erreurs reçues du serveur FastCGI doivent être prises en charge par Nginx ou transmises directement au client.

### **Variables communes utilisées avec FastCGI**

Avant d'aborder les paramètres que vous pourriez utiliser avec les passes FastCGI, il est pertinent de discuter de certaines variables Nginx courantes que nous utiliserons pour définir ces paramètres. Certaines sont définies par le module FastCGI de Nginx, mais la plupart proviennent du module Core.

- **$query_string** ou **$args :**les arguments énoncés dans la demande initiale du client.

- **$is_args :** sera égal à \ ? s'il y a des arguments dans la requête et sera défini sur une chaîne vide dans le cas contraire. Ceci est utile lors de la construction de paramètres qui peuvent ou non avoir des arguments.

- **$request_method :** indique la méthode de requête initiale du client, ce qui peut aider à déterminer si une opération doit être autorisée dans le contexte actuel.

- **$content_type :** Cela est spécifié dans l'en-tête de requête Content-Type. Cette information est indispensable pour le proxy si la requête de l'utilisateur est un POST, afin de gérer correctement le contenu qui suit.

- **$content_length :** Cela est déterminé par la valeur de l'en-tête Content-Length du client. Ces informations sont nécessaires pour toutes les requêtes POST des clients.

- **$fastcgi_script_name :** Cela contiendra le fichier de script à exécuter. Si la requête se termine par une barre oblique (/), la valeur de la directive **fastcgi_index** sera ajoutée à la fin. Si la directive **fastcgi_split_path_info** est utilisée, cette variable sera définie par le premier groupe capturé spécifié par cette directive. La valeur de cette variable doit indiquer le script réel à exécuter.

- **$request_filename :**  Cette variable contiendra le chemin d'accès au fichier demandé. Elle obtient cette valeur en combinant la racine du document actuel, en prenant en compte à la fois les directives root et alias, ainsi que la valeur de **$fastcgi_script_name.** Cela constitue une manière très flexible de définir le paramètre **SCRIPT_FILENAME.**

- **$request_uri :** La requête entière telle que reçue du client, incluant le script, toute information de chemin supplémentaire, ainsi que toutes les chaînes de requête.

- **$fastcgi_path_info :** Cette variable contient des informations de chemin supplémentaires qui peuvent être présentes après le nom du script dans la requête. Elle peut parfois indiquer un autre emplacement que le script à exécuter doit connaître. La valeur de cette variable provient du deuxième groupe capturé par l'expression régulière utilisée avec la directive **fastcgi_split_path_info.**

- **$document_root :** Cette variable contient la valeur actuelle de la racine du document. Elle est définie en fonction des directives root ou alias.

- **$uri :** Cette variable contient l'URI actuel avec la normalisation appliquée. Puisque certaines directives de réécriture ou de redirection interne peuvent modifier l'URI, cette variable reflète ces changements.

### **Paramètres FastCGI communs**

Les paramètres FastCGI représentent des informations clé-valeur que nous voulons rendre disponibles au processeur FastCGI auquel nous envoyons la requête. Toutes les applications n'auront pas besoin des mêmes paramètres, donc il est souvent nécessaire de consulter la documentation de l'application.

Certains de ces paramètres sont indispensables pour que le processeur identifie correctement le script à exécuter. D'autres sont destinés au script lui-même, ce qui peut modifier son comportement s'il est configuré pour utiliser les paramètres définis.

- **QUERY_STRING  :** Ce paramètre doit être défini sur n'importe quelle chaîne de requête fournie par le client. Il s'agit généralement de paires clé-valeur fournies après un « ? » dans l'URI. Habituellement, ce paramètre est défini sur les variables **$query_string** ou **$args**, qui doivent toutes deux contenir les mêmes données.

- **REQUEST_METHOD :** Ce paramètre indique au processeur FastCGI quel type d'action a été demandé par le client. Il s'agit de l'un des rares paramètres à définir pour que la passe fonctionne correctement.

- **CONTENT_TYPE :** Si la méthode de requête est définie sur \POST, ce paramètre doit être défini. Il indique le type de contenu auquel le processeur FastCGI doit s'attendre. Il est généralement défini sur la variable **$content_type**, laquelle est déterminée en fonction des informations de la requête d'origine. 

- **CONTENT_LENGTH :** Si la méthode de requête est \POST, ce paramètre doit être défini. Cela indique la longueur du contenu. Il est généralement défini sur **$content_length**, une variable qui récupère sa valeur à partir des informations de la demande client d'origine.

- **SCRIPT_NAME :** Ce paramètre permet d'indiquer le nom du script principal qui sera exécuté. Il s'agit d'un paramètre extrêmement important pouvant être configuré de différentes manières selon vos besoins. Souvent, cela est défini sur **$fastcgi_script_name**, qui devrait être l'URI de la requête, l'URI de la requête avec le fastcgi_index ajouté s'il se termine par une barre oblique, ou le premier groupe capturé si vous utilisez **fastcgi_fix_path_info**.

- **SCRIPT_FILENAME :**  Ce paramètre spécifie l'emplacement réel sur le disque du script à exécuter. En raison de sa relation avec le paramètre **SCRIPT_NAME**, certains guides suggèrent d'utiliser **$document_root$fastcgi_script_name**. Une autre alternative qui présente de nombreux avantages consiste à utiliser **$request_filename**.

- **REQUEST_URI :** Il doit contenir l'URI de requête complet et non modifié, avec le script à exécuter, des informations de chemin supplémentaires et tous les arguments. Certaines applications préfèrent analyser ces informations elles-mêmes. Ce paramètre leur fournit les informations nécessaires pour le faire.

- **PATH_INFO :** Si **cgi.fix_pathinfo** est défini sur \1 dans le fichier de configuration PHP, ce paramètre contiendra toutes les informations de chemin supplémentaires ajoutées après le nom du script. Il est souvent utilisé pour définir un argument de fichier sur lequel le script doit agir. Cependant, définir **cgi.fix_pathinfo** sur \1 peut avoir des implications sur la sécurité si les requêtes de script ne sont pas filtrées par d'autres moyens (nous en discuterons plus tard). Parfois, cela est défini sur la variable **$fastcgi_path_info**, qui contient le deuxième groupe capturé par la directive **fastcgi_split_path_info**. D'autres fois, une variable temporaire devra être utilisée car cette valeur est parfois encombrée par d'autres traitements.

- **PATH_TRANSLATED :** Ce paramètre mappe les informations de chemin contenues dans **PATH_INFO** dans un chemin de système de fichiers réel. Habituellement, cela sera défini sur quelque chose comme **$document_root$fastcgi_path_info**, mais parfois la dernière variable doit être remplacée par la variable temporaire enregistrée comme indiqué précédemment. 


# Étape 5: Déployer l'Application FastCGI
