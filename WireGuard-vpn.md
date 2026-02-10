# 🔒 WireGuard (WG-Easy) : VPN Nouvelle Génération

WireGuard est un protocole VPN moderne, extrêmement rapide et plus simple à auditer qu'OpenVPN. Cette installation utilise l'implémentation "WG-Easy" qui ajoute une interface de gestion Web, permettant de créer des configurations clients et des QR Codes en quelques clics pour sécuriser vos accès distants.

---

## 1. Prérequis

* **Système** : Une machine avec Docker et Docker Compose installés.
* **Réseau** :
    * Le port `51820` (UDP) doit être ouvert sur le pare-feu du serveur.
    * Le port `51821` (TCP) sera utilisé pour l'interface web d'administration.
* **Accès Externe** : Vous devez connaître votre **Adresse IP Publique** (ou avoir un nom de domaine dynamique type DDNS configuré).

---

## 2. Installation

Nous utilisons Docker Compose pour déployer le serveur VPN et son interface de gestion simultanément.

### Étape 2.1 : Préparation du dossier

Créez un répertoire dédié pour stocker les configurations clients :

```bash
mkdir -p ~/wireguard
cd ~/wireguard
```

### Étape 2.2 : Création du fichier Compose

Créez le fichier de configuration :

```bash
nano docker-compose.yml
```

Copiez-y le contenu ci-dessous.
**⚠️ IMPORTANT :** Remplacez `IP_PUBLIQUE_OU_DOMAINE` par votre véritable IP publique (celle de votre box) ou votre nom de domaine.

```yaml
version: "3.8"
services:
  wg-easy:
    environment:
      # L'adresse externe pour se connecter (Votre IP Publique)
      - WG_HOST=IP_PUBLIQUE_OU_DOMAINE
      
      # Mot de passe pour l'interface web (Changez-le !)
      - PASSWORD=votre_mot_de_passe_admin
      
      # Configuration Réseau (Ne pas toucher sauf besoin spécifique)
      - WG_PORT=51820
      - WG_DEFAULT_ADDRESS=10.8.0.x
      - WG_DEFAULT_DNS=1.1.1.1
      - WG_ALLOWED_IPS=0.0.0.0/0
      - WG_MTU=1420

    image: ghcr.io/wg-easy/wg-easy
    container_name: wireguard
    volumes:
      - ./config:/etc/wireguard
    ports:
      - "51820:51820/udp" # Tunnel VPN
      - "51821:51821/tcp" # Interface Web
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

### Étape 2.3 : Démarrage du Service

Lancez le conteneur :

```bash
docker compose up -d
```

---

## 3. Configuration

Tout se gère désormais depuis l'interface graphique.

### Étape 3.1 : Accès à l'interface
Depuis votre navigateur (sur le réseau local), accédez à :
`http://IP_LOCALE_DU_SERVEUR:51821`

* Connectez-vous avec le mot de passe défini dans le fichier `docker-compose.yml`.

### Étape 3.2 : Création d'un client
1.  Cliquez sur le bouton rouge **+ New Client**.
2.  Donnez un nom (ex: `iPhone-Paulo` ou `PC-Portable`).
3.  Cliquez sur **Create**.

### Étape 3.3 : Connexion d'un appareil
* **Sur Mobile (iOS/Android)** : Installez l'application officielle "WireGuard". Cliquez sur l'icône **QR Code** dans l'interface web et scannez-le avec l'application. Activez le tunnel.
* **Sur PC (Windows/Mac/Linux)** : Cliquez sur l'icône **Télécharger** (Download) pour récupérer le fichier `.conf`. Importez ce fichier dans le client WireGuard officiel.

---

## 4. Vérification

Validons que le tunnel VPN fonctionne correctement et chiffre votre trafic.

### Étape 4.1 : Test d'IP
1.  **Sans le VPN** : Allez sur un site comme `mon-ip.io` avec votre téléphone (en 4G/5G). Notez votre IP.
2.  **Activez le VPN** WireGuard sur votre téléphone.
3.  Rafraîchissez la page `mon-ip.io`.
4.  **Succès** : L'IP affichée doit maintenant être celle de votre serveur (votre Box Internet) et non plus celle de votre opérateur mobile.

### Étape 4.2 : Accès aux ressources locales
Essayez d'accéder à l'interface de votre Portainer ou Proxmox (`http://192.168.x.x...`) depuis votre téléphone en 4G avec le VPN actif.
* **Succès** : Si la page s'ouvre, vous avez un accès sécurisé à votre infrastructure depuis n'importe où dans le monde.

---
*Guide réalisé par Paulo Rosa.*
