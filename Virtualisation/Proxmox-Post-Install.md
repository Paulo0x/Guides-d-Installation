# ☁️ Proxmox VE 8 : Initialisation & Bonnes Pratiques

Proxmox Virtual Environment (PVE) est une solution complète de gestion de virtualisation serveur open-source. Ce guide détaille les étapes critiques à effectuer juste après l'installation pour transformer une installation brute en un hyperviseur de production stable et prêt à l'emploi.

---

## 1. Prérequis

* **Serveur** : Une machine physique avec Proxmox VE 8.x installé.
* **Accès** : Accès à l'interface web (`https://IP-SERVEUR:8006`) ou accès SSH (`root`).
* **Réseau** : Le serveur doit avoir un accès Internet pour télécharger les mises à jour.

---

## 2. Configuration des Dépôts (Repositories)

Par défaut, Proxmox est configuré pour utiliser les dépôts "Entreprise" (payants). Si vous n'avez pas de licence, les mises à jour échoueront. Nous allons basculer sur les dépôts "No-Subscription" (Communauté).

### Étape 2.1 : Désactivation du dépôt Entreprise
Dans le terminal du serveur (Shell), commentez la ligne du dépôt entreprise :

```bash
sed -i "s/^deb/#deb/g" /etc/apt/sources.list.d/pve-enterprise.list
```

### Étape 2.2 : Ajout du dépôt "No-Subscription"
Ajoutez la source communautaire officielle pour Proxmox 8 (Bookworm) :

```bash
echo "deb [http://download.proxmox.com/debian/pve](http://download.proxmox.com/debian/pve) bookworm pve-no-subscription" >> /etc/apt/sources.list
```

### Étape 2.3 : Correction des dépôts Ceph (Optionnel mais recommandé)
Même si vous n'utilisez pas Ceph immédiatement, il vaut mieux désactiver son dépôt entreprise pour éviter des erreurs lors des updates :

```bash
echo "deb [http://download.proxmox.com/debian/ceph-quincy](http://download.proxmox.com/debian/ceph-quincy) bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list
```

---

## 3. Mise à jour du Système

Une fois les sources corrigées, il est impératif de mettre à jour le cœur du système (Kernel Linux et paquets Proxmox).

Lancez la commande suivante :

```bash
apt update && apt dist-upgrade -y
```

> **Note importante** : Sur Proxmox, utilisez toujours `dist-upgrade` plutôt que `upgrade` simple, car cela permet de gérer intelligemment les changements de dépendances du noyau.

Une fois terminé, redémarrez l'hyperviseur pour charger le nouveau noyau :

```bash
reboot
```

---

## 4. Confort & Interface (Suppression du Nag Screen)

À chaque connexion, Proxmox affiche une fenêtre contextuelle "You do not have a valid subscription". Voici comment la désactiver proprement pour un usage Lab/Home.

Exécutez cette commande unique pour patcher l'interface web :

```bash
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid subscription'\),)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

*L'interface web va redémarrer (cela ne coupe pas les VMs).*

---

## 5. Configuration Réseau (Bridge)

Pour que vos VMs puissent accéder à votre réseau local comme si elles étaient physiquement branchées, nous utilisons un **Linux Bridge** (`vmbr0`).

1.  Connectez-vous à l'interface web.
2.  Allez dans **Datacenter** > **[Votre Node]** > **System** > **Network**.
3.  Vérifiez la présence de `vmbr0`.

**Configuration type recommandée (IP Statique) :**
* **Interface** : `vmbr0`
* **IPv4/CIDR** : `192.168.1.10/24` (Adaptez à votre réseau)
* **Gateway** : `192.168.1.1` (Votre Box/Routeur)
* **Bridge ports** : `enp3s0` (Le nom de votre carte réseau physique)

*Si vous modifiez cette partie, cliquez sur "Apply Configuration" en haut. Attention, une erreur ici coupe votre accès !*

---

## 6. Vérification & Premier déploiement

Avant de créer des machines virtuelles, assurons-nous que le stockage des ISOs est prêt.

### Étape 6.1 : Télécharger une ISO directement
Proxmox permet de télécharger des ISOs directement depuis une URL (plus rapide que d'uploader depuis votre PC).

1.  Allez dans **Datacenter** > **[Votre Node]** > **local (pve)** > **ISO Images**.
2.  Cliquez sur **"Download from URL"**.
3.  Exemple pour Debian 12 (Netinst) :
    * URL : `https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.5.0-amd64-netinst.iso`
    * Cliquez sur **Query URL** puis **Download**.

### Étape 6.2 : Création de VM test
Si vous pouvez effectuer ces actions sans erreur, votre nœud Proxmox est sain et prêt pour la production :
1.  Créer une VM (`Create VM`).
2.  Lui assigner l'ISO téléchargée.
3.  Lui donner accès au réseau via `vmbr0`.
4.  La démarrer et obtenir une IP via DHCP.

---
*Guide d'initialisation réalisé par Paulo Rosa.*
