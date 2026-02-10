# 🚫 Pi-hole : Serveur DNS & Bloqueur de Publicités

Pi-hole est un "DNS Sinkhole" qui protège l'ensemble de votre réseau contre les publicités et les traceurs, sans avoir besoin d'installer de plugins sur chaque appareil. Ce guide couvre son déploiement via Docker et la résolution des conflits de port DNS sur Linux.

---

## 1. Prérequis

* **Système** : Une machine avec Docker et Docker Compose installés.
* **Réseau** : Le serveur doit avoir une adresse IP fixe.
* **Ports** : Le port `53` (DNS) est critique. Nous allons devoir le libérer sur l'hôte.

---

## 2. Préparation du Système (Port 53)

Sur la plupart des distributions modernes (Ubuntu/Debian), le port 53 est déjà occupé par `systemd-resolved`. Il faut le désactiver pour que Pi-hole puisse écouter les requêtes DNS.

### Étape 2.1 : Désactivation du Stub Listener

Exécutez les commandes suivantes :

```bash
# 1. Désactiver le résolveur système sur le port 53
sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf

# 2. Changer le lien symbolique du fichier resolv.conf
sudo sh -c 'rm /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf'

# 3. Redémarrer le service
sudo systemctl restart systemd-resolved
```

*Vérification : La commande `sudo lsof -i :53` ne doit plus rien retourner.*

---

## 3. Installation

Nous déployons Pi-hole via un conteneur Docker pour faciliter la mise à jour et la gestion.

### Étape 3.1 : Création du dossier

```bash
mkdir -p ~/pihole
cd ~/pihole
```

### Étape 3.2 : Création du fichier Compose

Créez le fichier de configuration :

```bash
nano docker-compose.yml
```

Copiez-y le contenu ci-dessous.
*Note : Nous mappons l'interface web sur le port 8080 pour ne pas entrer en conflit avec Nginx Proxy Manager (port 80).*

```yaml
version: "3"

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8080:80"    # Interface Web sur le port 8080
    environment:
      TZ: 'Europe/Paris'
      WEBPASSWORD: 'admin' # Changez ce mot de passe !
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

### Étape 3.3 : Démarrage

```bash
docker compose up -d
```

---

## 4. Configuration

Une fois le conteneur lancé, il faut configurer vos appareils pour utiliser ce nouveau DNS.

### Étape 4.1 : Accès à l'interface
Ouvrez votre navigateur :
`http://IP_DE_VOTRE_SERVEUR:8080/admin`

* Connectez-vous avec le mot de passe défini dans le fichier compose (ici `admin`).

### Étape 4.2 : Utilisation du DNS
Pour que le filtrage fonctionne, vous avez deux choix :
1.  **Méthode Globale (Recommandée)** : Changez le serveur DNS "Primaire" dans la configuration DHCP de votre Box Internet/Routeur en mettant l'IP de votre serveur Pi-hole.
2.  **Méthode Manuelle** : Changez les paramètres DNS de votre PC uniquement (dans les propriétés de la carte réseau).

---

## 5. Vérification

Validons que les publicités sont bien bloquées.

### Étape 5.1 : Test de résolution
Sur votre PC (après avoir configuré le DNS), ouvrez un terminal et tapez :

```bash
nslookup google.com
# Doit retourner la vraie IP de Google

nslookup doubleclick.net
# Doit retourner 0.0.0.0 (C'est la preuve que Pi-hole bloque la pub !)
```

### Étape 5.2 : Tableau de bord
Retournez sur l'interface web de Pi-hole. Le compteur "Queries Blocked" doit commencer à augmenter à mesure que vous naviguez sur Internet.

---
*Guide réalisé par Paulo Rosa.*
