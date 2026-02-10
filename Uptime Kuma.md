# ⏱️ Uptime Kuma : Monitoring & Status Page

Uptime Kuma est un outil de surveillance "self-hosted" moderne et léger. Contrairement à Zabbix (orienté métriques serveur), Uptime Kuma se concentre sur la disponibilité des services (HTTP, TCP, DNS) et offre des pages de statut publiques élégantes pour visualiser la santé de votre infrastructure.

---

## 1. Prérequis

* **Système** : Une machine avec Docker et Docker Compose installés.
* **Ports** : Le port `3001` doit être disponible sur l'hôte.
* **Volume** : Un dossier persistant pour stocker l'historique de surveillance.

---

## 2. Installation

Nous utilisons Docker Compose pour un déploiement propre et maintenable.

### Étape 2.1 : Préparation de l'environnement

Créez le répertoire de travail :

```bash
mkdir -p ~/uptime-kuma
cd ~/uptime-kuma
```

### Étape 2.2 : Création du fichier Compose

Créez le fichier de configuration :

```bash
nano docker-compose.yml
```

Copiez-y le contenu suivant :

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./data:/app/data
      # Montage du socket Docker (Optionnel : permet de surveiller les conteneurs locaux)
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - "3001:3001"
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

### Étape 2.3 : Démarrage du Service

Lancez le conteneur en arrière-plan :

```bash
docker compose up -d
```

---

## 3. Configuration

L'outil est maintenant actif, il faut créer le compte administrateur et ajouter les premières sondes.

### Étape 3.1 : Création du compte Admin

Ouvrez votre navigateur et accédez à :
`http://IP_DE_VOTRE_SERVEUR:3001`

1.  **Langue** : Sélectionnez "Français".
2.  **Créer un utilisateur** : Définissez votre pseudo et un mot de passe robuste.
3.  Cliquez sur **Créer**.

### Étape 3.2 : Paramétrage Général

Une fois connecté :
1.  Allez dans **Paramètres** (en haut à droite) > **Général**.
2.  **Fuseau horaire** : Réglez sur "Europe/Paris" (automatique en général).
3.  **Apparence** : Activez le "Mode Sombre" pour le confort visuel (recommandé).

---

## 4. Vérification

Nous allons configurer une sonde de test pour prouver que le système fonctionne.

### Étape 4.1 : Ajouter une sonde (Monitor)

1.  Cliquez sur le bouton vert **+ Ajouter une sonde** (en haut à gauche).
2.  Remplissez les champs pour tester un site fiable (ex: Google) :
    * **Type de sonde** : `HTTP(s)`
    * **Nom** : `Google Public`
    * **URL** : `https://www.google.com`
    * **Intervalle** : `60` (secondes)
3.  Cliquez sur **Enregistrer**.

### Étape 4.2 : Validation du Monitoring

* Retournez sur le **Tableau de bord**.
* Vous devriez voir votre sonde "Google Public" passer au **Vert** (En ligne).
* À droite, le graphique de latence (Ping) doit commencer à se dessiner.
* **Succès** : Si vous coupez votre connexion internet (ou ciblez une fausse IP), la sonde passera au **Rouge** et Uptime Kuma est opérationnel.

---
*Guide réalisé par Paulo Rosa.*
