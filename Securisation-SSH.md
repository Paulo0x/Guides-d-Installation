# 🔐 Sécurisation SSH : Authentification par Clés

L'authentification par mot de passe est le vecteur d'attaque n°1 sur les serveurs exposés à Internet (Brute Force). Ce guide détaille la mise en place d'une authentification robuste par paire de clés cryptographiques (Ed25519) et la désactivation totale des mots de passe.

---

## 1. Prérequis

Avant de débuter cette procédure, assurez-vous de disposer des éléments suivants :

* **Serveur** : Une machine Linux (Debian, Ubuntu, CentOS...) avec le service SSH actif.
* **Client** : Votre poste de travail (Windows avec PowerShell, Linux ou macOS).
* **Privilèges** : Un accès `root` ou un utilisateur avec des droits `sudo` sur le serveur.
* **Accès initial** : Une connexion SSH fonctionnelle par mot de passe (pour la mise en place).

---

## 2. Installation

Cette étape consiste à générer la paire de clés sur le poste client et à "installer" la clé publique sur le serveur.

### Étape 2.1 : Génération des clés (Sur le poste client)

Ouvrez un terminal sur votre ordinateur local (**pas sur le serveur**) et exécutez la commande suivante pour générer une clé moderne (Ed25519) :

```bash
ssh-keygen -t ed25519 -C "admin@mon-infra"
```

1.  Appuyez sur **Entrée** pour valider l'emplacement par défaut.
2.  **Recommandé** : Saisissez une passphrase complexe pour chiffrer la clé privée sur votre disque.

### Étape 2.2 : Transfert de la clé (Vers le serveur)

Il faut maintenant copier l'identité publique vers le serveur distant.

**Méthode automatique (Linux / macOS / Windows 10+) :**
Remplacez `user` et `ip-serveur` par vos informations.

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@ip-serveur
```

*(Si la commande `ssh-copy-id` n'est pas disponible, copiez le contenu du fichier `.pub` et collez-le manuellement dans le fichier `~/.ssh/authorized_keys` du serveur).*

---

## 3. Configuration

Une fois la clé en place, nous devons configurer le démon SSH (`sshd`) pour interdire les anciennes méthodes de connexion.

### Étape 3.1 : Modification du fichier de configuration

Connectez-vous au serveur et éditez le fichier de configuration SSH :

```bash
sudo nano /etc/ssh/sshd_config
```

### Étape 3.2 : Paramètres de sécurisation (Hardening)

Recherchez et modifiez les directives suivantes. Si elles n'existent pas, ajoutez-les à la fin du fichier :

```ssh
# 1. Désactiver l'authentification par mot de passe (CRITIQUE)
PasswordAuthentication no

# 2. Désactiver le Challenge-Response
ChallengeResponseAuthentication no

# 3. Interdire la connexion directe en root (Bonne pratique)
PermitRootLogin no

# 4. S'assurer que l'authentification par clé publique est active
PubkeyAuthentication yes

# 5. Désactiver l'authentification PAM (Optionnel mais recommandé si pas utilisé)
UsePAM no
```

Sauvegardez le fichier (`Ctrl + O`, `Entrée`) et quittez (`Ctrl + X`).

---

## 4. Vérification

C'est l'étape la plus critique pour éviter de se bloquer dehors.

> ⚠️ **ATTENTION : Ne fermez surtout pas votre terminal actuel !** Gardez votre session active en cas d'erreur.

### Étape 4.1 : Redémarrage du service

Appliquez la configuration :

```bash
sudo systemctl restart ssh
```

### Étape 4.2 : Test de connexion

Ouvrez un **nouveau** terminal sur votre poste client et tentez une connexion :

```bash
ssh user@ip-serveur
```

* **Succès** : Si le serveur vous connecte directement (ou demande uniquement la passphrase de votre clé) sans demander le mot de passe utilisateur du serveur.
* **Échec** : Si la connexion est refusée ("Permission denied"), reprenez le terminal resté ouvert pour corriger le fichier `/etc/ssh/sshd_config` (vérifiez souvent `PubkeyAuthentication yes`).

---
*Guide réalisé par Paulo Rosa.*
