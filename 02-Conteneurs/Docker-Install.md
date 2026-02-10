# 🐳 Docker & Docker Compose : Installation et Configuration

Docker est la plateforme standard pour le déploiement d'applications en conteneurs. Ce guide détaille l'installation de la version "Community Edition" (CE) depuis les dépôts officiels, ainsi que l'installation du plugin Docker Compose sur Debian 12.

---

## 1. Prérequis

* **Système** : Une machine sous Debian 11/12 ou Ubuntu 22.04/24.04.
* **Architecture** : Processeur 64-bits (x86_64/amd64) ou ARM64.
* **Privilèges** : Accès `root` ou utilisateur avec droits `sudo`.
* **Nettoyage** : S'assurer qu'aucune ancienne version de Docker n'est installée (`docker.io`, `docker-doc`, etc.).

---

## 2. Installation

Nous n'utilisons pas les paquets par défaut de Debian (souvent obsolètes), mais le dépôt officiel Docker pour avoir la dernière version stable.

### Étape 2.1 : Installation des dépendances
Mettez à jour l'index et installez les outils nécessaires pour gérer les dépôts HTTPS :

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg -y
```

### Étape 2.2 : Ajout de la clé GPG officielle
Cette étape sécurise les téléchargements depuis les serveurs de Docker.

```bash
# Création du dossier pour les clés (si inexistant)
sudo install -m 0755 -d /etc/apt/keyrings

# Téléchargement de la clé
curl -fsSL [https://download.docker.com/linux/debian/gpg](https://download.docker.com/linux/debian/gpg) | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Attribution des droits de lecture
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### Étape 2.3 : Ajout du dépôt Docker
Nous ajoutons le lien vers le dépôt correspondant à votre version de Linux :

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] [https://download.docker.com/linux/debian](https://download.docker.com/linux/debian) \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Étape 2.4 : Installation des paquets
Mettez à jour la liste des paquets (pour inclure le nouveau dépôt) et installez Docker Engine + Compose :

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

---

## 3. Configuration

Par défaut, seul l'utilisateur `root` peut lancer des conteneurs. Pour un usage quotidien, il faut autoriser votre utilisateur standard.

### Étape 3.1 : Gestion des droits (Post-installation)
Ajoutez votre utilisateur actuel au groupe `docker` :

```bash
sudo usermod -aG docker $USER
```

**⚠️ Important :** Pour que cette modification prenne effet, vous devez vous **déconnecter** et vous reconnecter à votre session SSH (ou taper `newgrp docker`).

### Étape 3.2 : Activation au démarrage
Assurez-vous que Docker se lance automatiquement au reboot du serveur :

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

---

## 4. Vérification

Validons que le moteur de conteneur et l'orchestrateur fonctionnent correctement.

### Étape 4.1 : Versions installées
Vérifiez que les commandes répondent :

```bash
docker --version
# Doit afficher : Docker version 24.x.x (ou plus récent)

docker compose version
# Doit afficher : Docker Compose version v2.x.x
```

### Étape 4.2 : Le test ultime "Hello World"
Lancez un petit conteneur de test qui va télécharger une image, l'exécuter et s'arrêter :

```bash
docker run hello-world
```

* **Succès** : Si vous voyez le message :
> *Hello from Docker!*
> *This message shows that your installation appears to be working correctly.*

Votre serveur est prêt à déployer des stacks complètes.

---
*Guide réalisé par Paulo Rosa.*
