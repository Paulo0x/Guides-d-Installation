# 🐧 LAMP Stack : Le Serveur Web Fondamental

L'acronyme LAMP (Linux, Apache, MariaDB, PHP) désigne l'architecture historique du web. Savoir installer et configurer ces composants "en dur" sur le système (sans Docker) est une compétence fondamentale pour comprendre comment les serveurs web traitent les requêtes, gèrent les permissions et connectent les bases de données.

---

## 1. Prérequis

* **Système** : Un serveur Debian 12 ou Ubuntu 22.04 vierge.
* **Réseau** : Le port `80` (HTTP) doit être ouvert.
* **Privilèges** : Accès `root` ou `sudo`.

---

## 2. Installation des Composants

Nous allons installer les trois briques logicielles une par une.

### Étape 2.1 : Le Serveur Web (Apache2)

Apache est le serveur HTTP le plus utilisé au monde.

```bash
sudo apt update
sudo apt install apache2 -y

# Vérification du statut
sudo systemctl status apache2
```
*Si le service est "active (running)", ouvrez l'IP de votre serveur dans un navigateur. Vous devriez voir la page "Apache2 Default Page".*

### Étape 2.2 : La Base de Données (MariaDB)

MariaDB est le fork communautaire et libre de MySQL.

```bash
sudo apt install mariadb-server -y

# Sécurisation de l'installation (Script interactif)
sudo mysql_secure_installation
```
*Répondez aux questions :*
* *Switch to unix_socket authentication?* **Y**
* *Change the root password?* **Y** (Définissez un mot de passe root SQL solide)
* *Remove anonymous users?* **Y**
* *Disallow root login remotely?* **Y**
* *Remove test database?* **Y**
* *Reload privilege tables now?* **Y**

### Étape 2.3 : Le Langage de Script (PHP)

PHP permet de générer des pages dynamiques. Nous installons le moteur et le module de liaison pour Apache.

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

---

## 3. Configuration (VirtualHost)

La bonne pratique n'est pas d'utiliser le dossier par défaut, mais de créer un "VirtualHost" pour chaque site. Cela permet d'héberger plusieurs sites sur le même serveur.

### Étape 3.1 : Création du dossier du site

```bash
# Création du répertoire (remplacez 'mon-site' par le nom de votre projet)
sudo mkdir -p /var/www/mon-site

# Attribution des droits à l'utilisateur web (www-data) et à votre utilisateur actuel
sudo chown -R $USER:www-data /var/www/mon-site
sudo chmod -R 775 /var/www/mon-site
```

### Étape 3.2 : Création du fichier de configuration Apache

```bash
sudo nano /etc/apache2/sites-available/mon-site.conf
```

Collez la configuration suivante :

```apache
<VirtualHost *:80>
    # L'email de l'admin (visible en cas d'erreur 500)
    ServerAdmin webmaster@localhost
    
    # Le chemin vers vos fichiers
    DocumentRoot /var/www/mon-site

    # Configuration des logs
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    
    # Autorisations pour le dossier
    <Directory /var/www/mon-site>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Étape 3.3 : Activation du site

Apache utilise un système de liens symboliques pour activer (`a2ensite`) ou désactiver (`a2dissite`) les sites.

```bash
# Désactiver le site par défaut
sudo a2dissite 000-default.conf

# Activer votre nouveau site
sudo a2ensite mon-site.conf

# Vérifier la syntaxe
sudo apache2ctl configtest
# Doit retourner : Syntax OK

# Redémarrer Apache pour appliquer
sudo systemctl reload apache2
```

---

## 4. Vérification

Nous allons créer une page PHP de test pour confirmer que tout fonctionne ensemble.

### Étape 4.1 : Création du fichier test

```bash
nano /var/www/mon-site/index.php
```

Collez ce code PHP simple :

```php
<?php
phpinfo();
?>
```

### Étape 4.2 : Test final

1.  Ouvrez votre navigateur.
2.  Allez sur `http://IP_DE_VOTRE_SERVEUR`.
3.  **Succès** : Vous devez voir une page violette et grise avec le logo PHP et toutes les informations de version (Core, Date, Modules...).
    * Cela prouve qu'Apache tourne (il a servi la page).
    * Cela prouve que PHP fonctionne (le code a été interprété).

*Note de sécurité : Une fois le test validé, supprimez ce fichier (`rm /var/www/mon-site/index.php`) car il donne trop d'infos aux pirates.*

---
*Guide réalisé par Paulo Rosa.*
