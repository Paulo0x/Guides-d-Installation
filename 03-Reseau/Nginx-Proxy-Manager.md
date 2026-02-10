# 🌐 Nginx Proxy Manager : Reverse Proxy & Gestion SSL

Nginx Proxy Manager est une solution complète de "Reverse Proxy" basée sur Nginx. Elle permet de diriger le trafic entrant (HTTP/HTTPS) vers vos différents conteneurs ou serveurs internes, tout en gérant automatiquement la création et le renouvellement des certificats SSL (Let's Encrypt) pour sécuriser vos échanges.

---

## 1. Prérequis

* **Système** : Une machine avec Docker et Docker Compose installés.
* **Ports Disponibles** : Les ports `80` (HTTP), `443` (HTTPS) et `81` (Administration) doivent être libres sur l'hôte.
* **DNS (Optionnel)** : Avoir un nom de domaine pointant vers l'IP du serveur est recommandé pour tester la génération SSL.

---

## 2. Installation

Nous allons déployer l'application via une stack Docker Compose incluant le service principal et sa base de données.

### Étape 2.1 : Préparation de l'environnement

Créez un dossier dédié pour stocker la configuration et la base de données :

```bash
mkdir -p ~/nginx-proxy
cd ~/nginx-proxy
```

### Étape 2.2 : Création du fichier Compose

Créez le fichier de définition du service :

```bash
nano docker-compose.yml
```

Copiez-y le contenu suivant (ceci définit l'application et une base de données MySQL robuste) :

```yaml
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'      # Port HTTP public
      - '81:81'      # Port d'administration Web
      - '443:443'    # Port HTTPS public
    environment:
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "npm"
      DB_MYSQL_PASSWORD: "npm_password"
      DB_MYSQL_NAME: "npm"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - db

  db:
    image: 'jc21/mariadb-aria:latest'
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'npm_root_password'
      MYSQL_DATABASE: 'npm'
      MYSQL_USER: 'npm'
      MYSQL_PASSWORD: 'npm_password'
    volumes:
      - ./mysql:/var/lib/mysql
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

### Étape 2.3 : Démarrage de la Stack

Lancez le téléchargement des images et le démarrage des conteneurs en mode détaché :

```bash
docker compose up -d
```

---

## 3. Configuration

L'initialisation se fait via l'interface web dédiée sur le port 81.

### Étape 3.1 : Premier accès

Ouvrez votre navigateur et accédez à l'adresse :
`http://IP_DE_VOTRE_SERVEUR:81`

### Étape 3.2 : Authentification par défaut

Utilisez les identifiants initiaux (fixés par l'éditeur) :

* **Email** : `admin@example.com`
* **Password** : `changeme`

### Étape 3.3 : Sécurisation du compte Admin

Dès la première connexion, une fenêtre contextuelle vous obligera à :
1.  Modifier l'email administrateur (mettez le vôtre).
2.  Modifier le mot de passe (choisissez un mot de passe complexe).

Ceci est impératif car le port 81 donne le contrôle total sur le routage de votre réseau.

---

## 4. Vérification

Validons que le proxy est actif et capable de gérer des requêtes.

### Étape 4.1 : Vérification des Conteneurs

Dans votre terminal, assurez-vous que les deux services sont "Up" :

```bash
docker compose ps
```
*Le statut doit être "Up" pour `nginx-proxy-app-1` et `nginx-proxy-db-1`.*

### Étape 4.2 : Test fonctionnel

Depuis un navigateur, tentez d'accéder au port 80 de votre serveur :
`http://IP_DE_VOTRE_SERVEUR`

* **Succès** : Vous devez voir une page par défaut affichant **"Congratulations! You've just successfully created your first Proxy Host"**. Cela signifie que Nginx écoute bien sur le port 80 et attend vos instructions de routage.

---
*Guide réalisé par Paulo Rosa.*
