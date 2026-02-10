# 🛡️ pfSense : Installation & Configuration Initiale

pfSense est une solution de pare-feu et de routage open-source basée sur FreeBSD. Ce guide couvre l'installation de la version Community Edition (CE) et la configuration fondamentale pour sécuriser un réseau local (LAN) avec accès Internet.

---

## 1. Prérequis

Avant de commencer, l'environnement doit respecter ces contraintes strictes :

* **Matériel** : Un PC ou serveur avec **2 cartes réseaux physiques** minimum (ex: `em0` pour le WAN, `em1` pour le LAN).
* **Support** : Une clé USB avec l'image ISO de *pfSense CE 2.7+* (gravée via Rufus).
* **Câblage** :
    * **Port WAN** : Relié à votre Box Internet (ou modem).
    * **Port LAN** : Relié à votre Switch ou directement à votre PC de configuration.
* **Architecture** : Processeur 64-bits (AMD64).

---

## 2. Installation

Cette étape installe le système FreeBSD optimisé sur le disque dur.

### Étape 2.1 : Démarrage et Options
1.  Bootez sur la clé USB.
2.  Acceptez le copyright (Appuyez sur **Accept**).
3.  Choisissez **Install pfSense** (Sélectionnez *Install* et validez).
4.  **Keymap** : Choisissez votre clavier (ex: `French ISO`) ou gardez par défaut si vous êtes à l'aise en QWERTY.
5.  **Partitionnement** : Choisissez **Auto (ZFS)** (C'est le système de fichiers le plus robuste moderne).
6.  Validez les options ZFS par défaut (**Install** > **Stripe** si un seul disque).
7.  Confirmez l'effacement du disque.

Une fois l'installation terminée, choisissez **Reboot** et **retirez la clé USB**.

---

## 3. Configuration

La configuration de pfSense se fait en deux temps : d'abord l'assignation des interfaces en ligne de commande (Console), puis le paramétrage fin via l'interface Web.

### Étape 3.1 : Assignation des Interfaces (Console)
Au redémarrage, vous arrivez sur un menu textuel (numéroté de 0 à 16).

1.  Le système va demander : `Should VLANs be set up now?` -> Répondez **n** (Non).
2.  `Enter the WAN interface name` : Tapez le nom de votre carte connectée à Internet (ex: `em0` ou `vtnet0`).
3.  `Enter the LAN interface name` : Tapez le nom de votre carte réseau local (ex: `em1` ou `vtnet1`).
4.  Confirmez avec **y** (Yes).

### Étape 3.2 : Définir l'adresse IP du LAN
Par défaut, pfSense prend `192.168.1.1`. Si votre Box Internet utilise déjà cette IP, il y aura un conflit ! Changeons-la.

1.  Dans le menu, tapez **2** (Set interface(s) IP address).
2.  Choisissez l'interface LAN (tapez **2**).
3.  **IP Address** : Saisissez une IP unique, ex: `192.168.10.1`.
4.  **Subnet Mask** : Tapez `24` (pour 255.255.255.0).
5.  **Gateway** : Appuyez sur *Entrée* (Laissez vide pour le LAN).
6.  **DHCP Server** : Répondez **y** pour activer le DHCP.
7.  **Range** : Définissez la plage (ex: `192.168.10.100` à `192.168.10.200`).

### Étape 3.3 : L'Assistant Web (Wizard)
Connectez votre PC sur le port LAN du pfSense.
1.  Ouvrez votre navigateur et allez sur : `https://192.168.10.1` (ou l'IP définie).
2.  Identifiants par défaut :
    * User : `admin`
    * Password : `pfsense`
3.  L'assistant de configuration (Wizard) démarre :
    * **Hostname** : Donnez un nom (ex: `firewall`).
    * **DNS Servers** : Mettez `1.1.1.1` et `8.8.8.8`.
    * **Timezone** : `Europe/Paris`.
    * **WAN Configuration** : Laissez en **DHCP** (si derrière une box) ou **PPPoE** (si fibre directe). **Décochez** "Block private networks" si votre WAN est derrière une Box (RFC1918).
    * **Set Admin WebGUI Password** : Changez impérativement le mot de passe `pfsense`.
4.  Cliquez sur **Reload**.

---

## 4. Vérification

Assurons-nous que le pare-feu filtre et route correctement le trafic.

### Étape 4.1 : Vérification du Tableau de Bord (Dashboard)
Sur la page d'accueil de l'interface web :
* Vérifiez le widget **Interfaces** :
    * **WAN** : Doit avoir une IP verte (ex: `192.168.1.x` ou IP Publique).
    * **LAN** : Doit afficher `192.168.10.1`.

### Étape 4.2 : Test de Connectivité Client
Depuis votre PC connecté au LAN :
1.  Ouvrez un terminal (CMD ou PowerShell).
2.  Vérifiez votre IP : `ipconfig` (Vous devez avoir une IP en 192.168.10.x).
3.  Testez le DNS et la sortie Internet :
    ```bash
    ping google.com
    ```
    * *Succès : Si le ping répond, pfSense route bien les paquets vers Internet.*

### Étape 4.3 : Test de Filtrage (Optionnel)
Allez dans **Firewall** > **Rules** > **LAN**.
* Par défaut, une règle "Default Allow LAN to any rule" laisse tout passer.
* Le fait de naviguer sur Internet confirme que cette règle fonctionne.

---
*Guide réalisé par Paulo Rosa.*
