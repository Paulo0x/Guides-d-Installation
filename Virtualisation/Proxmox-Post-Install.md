# ☁️ Proxmox VE 8 : Installation & Configuration Initiale

Proxmox Virtual Environment (PVE) est une plateforme de virtualisation complète. Ce guide couvre l'installation du système d'exploitation (Hyperviseur) depuis zéro, jusqu'à sa configuration pour un usage en production sans licence payante.

---

## 1. Prérequis

Avant de commencer, assurez-vous d'avoir :

* **Matériel** : Un serveur ou PC dédié (64-bit, support VT-x/AMD-v activé dans le BIOS).
* **Support d'installation** : Une clé USB (8 Go min) contenant l'ISO de Proxmox VE 8 (créée avec Rufus ou BalenaEtcher).
* **Réseau** : Une connexion Internet par câble Ethernet (le Wi-Fi est fortement déconseillé pour un serveur).
* **Infos Réseau** : Avoir choisi une IP fixe pour le serveur (ex: `192.168.1.10`) et connaître sa passerelle (Gateway).

---

## 2. Installation

Cette étape décrit le processus d'installation du système d'exploitation sur le disque dur.

### Étape 2.1 : Démarrage
1.  Insérez la clé USB et démarrez le serveur.
2.  Accédez au menu de boot (F11, F12 ou Suppr selon la machine) et choisissez la clé USB (UEFI de préférence).
3.  Dans le menu Proxmox, sélectionnez **"Install Proxmox VE (Graphical)"**.

### Étape 2.2 : L'Assistant d'Installation (Pas à pas)
Suivez les écrans de l'installateur :

1.  **EULA** : Cliquez sur *I agree*.
2.  **Target Harddisk** : Sélectionnez le disque de destination.
    * *Conseil : Laissez les options par défaut (ext4) sauf si vous savez configurer du ZFS/RAID.*
3.  **Country/Time Zone** :
    * Country : `France`
    * Time Zone : `Europe/Paris`
    * Keyboard Layout : `French` (ou votre clavier).
4.  **Password & Email** :
    * Définissez le mot de passe `root` (Admin). **Ne l'oubliez pas !**
    * Entrez une adresse email valide (pour les alertes système).
5.  **Network Configuration** (CRITIQUE) :
    * **Hostname** : Donnez un nom complet (ex: `pve1.mon-domaine.local`).
    * **IP Address** : Mettez l'IP fixe choisie (ex: `192.168.1.10`).
    * **Gateway** : L'adresse de votre box/routeur (ex: `192.168.1.1`).
    * **DNS Server** : `1.1.1.1` ou `8.8.8.8` (ou votre box).
6.  **Summary** : Vérifiez tout et cliquez sur **Install**.

Une fois l'installation finie, retirez la clé USB et le serveur redémarrera.

---

## 3. Configuration

Maintenant que le système est installé, nous devons le configurer pour qu'il fonctionne sans licence entreprise. Connectez-vous en SSH ou utilisez la "Shell" via l'interface web (`https://IP-DE-VOTRE-SERVEUR:8006`).

### Étape 3.1 : Configuration des Dépôts (Repositories)
Par défaut, les mises à jour sont bloquées sans licence. On bascule sur les dépôts communautaires.

Exécutez ces commandes une par une :

# 1. Désactiver le dépôt Entreprise (qui cause des erreurs)
sed -i "s/^deb/#deb/g" /etc/apt/sources.list.d/pve-enterprise.list

# 2. Ajouter le dépôt "No-Subscription" (Gratuit)
echo "deb [http://download.proxmox.com/debian/pve](http://download.proxmox.com/debian/pve) bookworm pve-no-subscription" >> /etc/apt/sources.list

# 3. Corriger le dépôt Ceph (Stockage distribué)
echo "deb [http://download.proxmox.com/debian/ceph-quincy](http://download.proxmox.com/debian/ceph-quincy) bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list

Étape 3.2 : Mise à jour du Système
Mettez à jour le cœur du système avec les nouveaux dépôts :

apt update && apt dist-upgrade -y

Étape 3.3 : Suppression du message "No Subscription"
Pour ne plus avoir la fenêtre d'avertissement à chaque connexion :

sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid subscription'\),)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service

Redémarrez ensuite le serveur pour valider le nouveau noyau Linux : reboot

4. Vérification
Assurons-nous que votre hyperviseur est prêt pour la production.

Étape 4.1 : Accès à l'interface
Depuis votre PC, ouvrez un navigateur et allez sur : https://IP-DE-VOTRE-SERVEUR:8006

Acceptez l'avertissement de sécurité SSL.

Connectez-vous avec l'utilisateur root et votre mot de passe.

Succès : Si vous arrivez sur le tableau de bord avec les graphiques de RAM/CPU.

Étape 4.2 : Test fonctionnel (Télécharger une ISO)
Vérifions que le réseau et le stockage fonctionnent en téléchargeant une image d'installation.

Dans le menu de gauche, cliquez sur local (pve) > ISO Images.

Cliquez sur Download from URL.

Entrez l'URL d'une petite distribution (ex: Alpine Linux) : https://dl-cdn.alpinelinux.org/alpine/v3.19/releases/x86_64/alpine-virt-3.19.1-x86_64.iso

Cliquez sur Query URL puis Download.

Succès : Si la barre de progression va jusqu'à "TASK OK", votre serveur est opérationnel.

Guide réalisé par Paulo Rosa.
