# ☁️ Proxmox VE 8 : Installation & Configuration Initiale

Proxmox Virtual Environment (PVE) est une plateforme de virtualisation complète (Hyperviseur de Type 1). Ce guide couvre l'installation du système d'exploitation depuis zéro, ainsi que sa configuration post-installation pour un usage en production sans licence entreprise.

---

## 1. Prérequis

Avant de lancer l'installation, assurez-vous de disposer des éléments suivants :

* **Matériel** : Un serveur ou un PC 64-bit avec la virtualisation (VT-x / AMD-v) activée dans le BIOS.
* **Support d'installation** : Une clé USB (8 Go min) contenant l'ISO de Proxmox VE 8 (créée avec Rufus ou BalenaEtcher).
* **Réseau** : Le serveur doit être relié par câble Ethernet (le Wi-Fi est déconseillé pour un hyperviseur).
* **Plan d'adressage** : Vous devez avoir défini une **IP Fixe** pour ce serveur (ex: `192.168.1.10`) et connaître l'adresse de votre passerelle/box (ex: `192.168.1.1`).

---

## 2. Installation

Cette étape décrit le processus d'installation du système d'exploitation sur le disque dur physique du serveur.

### Étape 2.1 : Démarrage
1.  Insérez la clé USB et démarrez le serveur.
2.  Accédez au menu de boot (F11, F12 ou Suppr selon la machine) et choisissez la clé USB (mode UEFI recommandé).
3.  Dans le menu de démarrage Proxmox, sélectionnez **"Install Proxmox VE (Graphical)"**.

### Étape 2.2 : L'Assistant d'Installation
Suivez les écrans de configuration :

1.  **EULA** : Cliquez sur *I agree*.
2.  **Target Harddisk** : Sélectionnez le disque de destination.
    * *Note : Laissez les options par défaut (ext4) sauf besoin spécifique (ZFS/RAID).*
3.  **Country/Time Zone** :
    * Country : `France`
    * Time Zone : `Europe/Paris`
    * Keyboard Layout : `French` (ou selon votre clavier).
4.  **Password & Email** :
    * Définissez le mot de passe `root` (Administrateur système). **Ne l'oubliez pas !**
    * Entrez une adresse email valide (pour recevoir les alertes critiques).
5.  **Network Configuration** (CRITIQUE) :
    * **Hostname** : Donnez un nom complet (ex: `pve.local` ou `srv-proxmox.domaine`).
    * **IP Address** : Saisissez l'IP fixe choisie (ex: `192.168.1.10`).
    * **Gateway** : L'adresse de votre box/routeur (ex: `192.168.1.1`).
    * **DNS Server** : `1.1.1.1` ou l'IP de votre box.
6.  **Summary** : Vérifiez les informations et cliquez sur **Install**.

Une fois l'installation terminée, retirez la clé USB et laissez le serveur redémarrer.

---

## 3. Configuration

Par défaut, Proxmox est configuré pour les clients payants (Entreprise). Nous devons modifier cela pour permettre les mises à jour et nettoyer l'interface.

Toutes ces commandes se font via la **Console** (soit en branchant un écran/clavier sur le serveur, soit via l'interface web > Node > Shell).

### Étape 3.1 : Configuration des Dépôts (Repositories)
Nous allons désactiver le dépôt payant et activer le dépôt communautaire gratuit.

Copiez et exécutez ces commandes :

```bash
# 1. Désactiver le dépôt Entreprise (qui cause des erreurs 401)
sed -i "s/^deb/#deb/g" /etc/apt/sources.list.d/pve-enterprise.list

# 2. Ajouter le dépôt "No-Subscription" (Gratuit et Stable)
echo "deb [http://download.proxmox.com/debian/pve](http://download.proxmox.com/debian/pve) bookworm pve-no-subscription" >> /etc/apt/sources.list

# 3. Corriger le dépôt Ceph (Stockage) pour éviter les erreurs futures
echo "deb [http://download.proxmox.com/debian/ceph-quincy](http://download.proxmox.com/debian/ceph-quincy) bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list
```

### Étape 3.2 : Mise à jour du Système
Maintenant que les sources sont bonnes, mettez à jour le cœur du système :

```bash
apt update && apt dist-upgrade -y
```

### Étape 3.3 : Suppression de la bannière "No Subscription"
Pour supprimer le message d'avertissement qui apparaît à chaque connexion :

```bash
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid subscription'\),)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

*Redémarrez le serveur une dernière fois pour appliquer le nouveau noyau Linux :* `reboot`

---

## 4. Vérification

La dernière étape consiste à valider que l'hyperviseur est opérationnel et prêt à héberger des machines virtuelles.

### Étape 4.1 : Accès à l'interface Web
Depuis un autre ordinateur du réseau, ouvrez un navigateur et accédez à :
`https://IP-DE-VOTRE-SERVEUR:8006`

* Acceptez l'avertissement de sécurité SSL (C'est normal, le certificat est auto-signé).
* Connectez-vous avec l'utilisateur `root` et votre mot de passe.
* **Résultat attendu** : Vous accédez au tableau de bord (Datacenter) et voyez les graphiques de performance (CPU/RAM).

### Étape 4.2 : Test de téléchargement (ISO)
Vérifions que le serveur accède bien à Internet pour télécharger des images disques.

1.  Dans le menu de gauche, déroulez votre nœud et cliquez sur **local (pve)**.
2.  Allez dans la section **ISO Images**.
3.  Cliquez sur le bouton **Download from URL**.
4.  Entrez l'URL d'une image test (ex: Debian Netinst) :
    `https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.5.0-amd64-netinst.iso`
5.  Cliquez sur **Query URL** (le nom du fichier doit apparaître) puis sur **Download**.

**Succès** : Si la barre de progression se termine par "TASK OK", votre serveur Proxmox est parfaitement installé, à jour et connecté.

---
*Guide réalisé par Paulo Rosa.*
