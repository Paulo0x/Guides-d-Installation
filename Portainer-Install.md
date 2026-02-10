# 🚢 Portainer CE : Interface de Gestion Graphique pour Docker

Portainer Community Edition (CE) est l'interface de gestion (GUI) la plus populaire pour Docker. Elle permet de piloter vos conteneurs, images, réseaux et volumes via une interface web intuitive, sans avoir recours à la ligne de commande pour les tâches courantes.

---

## 1. Prérequis

* **Système** : Une machine avec Docker et Docker Compose déjà installés.
* **Ports** : Le port `9443` (HTTPS) doit être libre sur le serveur.
* **Privilèges** : Accès `sudo` ou appartenance au groupe `docker`.

---

## 2. Installation

Portainer s'installe lui-même comme un conteneur Docker. C'est la méthode la plus propre et la plus rapide.

### Étape 2.1 : Création du Volume

Nous devons créer un volume persistant pour stocker la base de données de Portainer (vos comptes utilisateurs, configurations, stacks). Sans cela, vous perdrez tout au redémarrage.

```bash
docker volume create portainer_data
```

### Étape 2.2 : Déploiement du Conteneur

Lancez la commande suivante pour télécharger l'image et démarrer le service.
*Note : Nous exposons le port 9443 pour l'accès web sécurisé.*

```bash
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Explication des options :
* `-d` : Mode détaché (tourne en arrière-plan).
* `--restart=always` : Redémarre automatiquement si le serveur reboot.
* `-v /var/run/docker.sock...` : Donne à Portainer le droit de piloter le Docker du serveur hôte.

---

## 3. Configuration

L'installation technique est finie, il faut maintenant initialiser l'accès administrateur.

### Étape 3.1 : Création du Compte Admin

1.  Ouvrez votre navigateur et accédez à : `https://IP_DE_VOTRE_SERVEUR:9443`
    * *Acceptez l'avertissement de sécurité SSL (certificat auto-signé).*
2.  **Username** : `admin`
3.  **Password** : Choisissez un mot de passe fort (12 caractères min).
4.  Cliquez sur **Create user**.

### Étape 3.2 : Connexion à l'environnement

L'assistant va vous demander quel environnement gérer.

1.  Cliquez sur **Get Started** (Portainer détecte automatiquement le socket local grâce au volume monté à l'étape 2.2).
2.  Vous arrivez sur le tableau de bord (Home).
3.  Cliquez sur la tuile **"local"** pour entrer dans la gestion de votre serveur.

---

## 4. Vérification

Validons que Portainer a bien le contrôle sur votre moteur Docker.

### Étape 4.1 : Vue d'ensemble

Dans le menu de gauche, cliquez sur **Containers**.
* Vous devriez voir la liste des conteneurs actifs (au minimum `portainer` lui-même).
* *Succès : Si vous voyez l'état "Running" en vert.*

### Étape 4.2 : Test de déploiement (Nginx)

Nous allons lancer un serveur web de test directement depuis l'interface pour prouver que tout fonctionne.

1.  Allez dans **App Templates** (menu de gauche).
2.  Cherchez **Nginx** dans la liste.
3.  Cliquez dessus.
4.  Donnez un nom (ex: `test-web`) et cliquez sur **Deploy the container**.
5.  Une fois déployé, cliquez sur le lien port `80:80` qui apparaît dans la liste des conteneurs.
6.  Si la page "Welcome to nginx!" s'ouvre, votre Portainer est parfaitement opérationnel pour administrer votre infrastructure.

---
*Guide réalisé par Paulo Rosa.*
