# 🌐 Réseau & SSH : Configuration Essentielle

Un serveur ne reste jamais en DHCP (adresse IP changeante). Un administrateur doit savoir fixer une IP statique et sécuriser l'accès distant (SSH). Ce guide couvre la configuration réseau sous Debian/Ubuntu et le durcissement du service SSH.

---

## 1. Nom de la Machine (Hostname)

Il est crucial d'identifier proprement ses serveurs.

```bash
# Voir le nom actuel
hostnamectl

# Changer le nom (ex: srv-web-01)
sudo hostnamectl set-hostname srv-web-01

# Vérifier le fichier hosts (pour que le serveur se reconnaisse lui-même)
sudo nano /etc/hosts
# Ajoutez la ligne : 127.0.1.1  srv-web-01
```

---

## 2. Configuration IP Statique (Méthode Debian)

Sur les serveurs, on modifie souvent `/etc/network/interfaces`.

### 2.1 Identifier son interface

```bash
ip a
```
*Notez le nom de votre carte réseau (ex: `ens18`, `eth0`).*

### 2.2 Modifier la configuration

```bash
sudo nano /etc/network/interfaces
```

Remplacez `iface ens18 inet dhcp` par ceci (adaptez les IP à votre réseau !) :

```bash
# L'interface de bouclage (toujours là)
auto lo
iface lo inet loopback

# Votre carte réseau en IP Fixe
auto ens18
iface ens18 inet static
    address 192.168.1.50/24    # L'IP que vous voulez
    gateway 192.168.1.1        # La box Internet / Routeur
    dns-nameservers 1.1.1.1    # Serveur DNS (Cloudflare ou Google 8.8.8.8)
```

### 2.3 Appliquer (Redémarrage recommandé pour les débutants)

```bash
sudo systemctl restart networking
# Ou si ça ne suffit pas :
sudo reboot
```

---

## 3. Sécurisation SSH (Indispensable)

SSH est la porte d'entrée principale des pirates. Il faut la blinder.

### 3.1 Création des clés (Sur votre PC, pas le serveur !)

On ne se connecte pas avec un mot de passe, mais avec une clé cryptographique. Sur votre ordinateur (Windows PowerShell ou Linux) :

```bash
# Générer la paire de clés (Faites Entrée partout)
ssh-keygen -t ed25519 -C "mon-pc-perso"

# Envoyer la clé publique sur le serveur
# (Remplacez paulo et l'IP par les vôtres)
ssh-copy-id paulo@192.168.1.50
```

*Testez la connexion : `ssh paulo@192.168.1.50`. Si on ne vous demande pas de mot de passe, c'est gagné !*

### 3.2 Durcissement du Serveur (sshd_config)

Maintenant que la clé fonctionne, on interdit les mots de passe et le root.

```bash
sudo nano /etc/ssh/sshd_config
```

Modifiez ou ajoutez ces lignes :

```ini
# 1. Interdire la connexion directe en root (Oblige à passer par un user normal + sudo)
PermitRootLogin no

# 2. Interdire l'authentification par mot de passe (Clé SSH obligatoire)
PasswordAuthentication no

# 3. (Optionnel) Changer le port pour éviter les scripts automatiques
# Port 2222
```

### 3.3 Appliquer la sécurité

**Attention :** Ne fermez pas votre terminal actuel avant d'avoir testé la connexion dans un autre terminal ! Si vous vous trompez, vous êtes enfermé dehors.

```bash
sudo systemctl restart ssh
```

---

## 4. Analyse des Ports (Netstat/SS)

Un bon TSSR sait toujours quels services écoutent sur sa machine.

```bash
# Voir les ports TCP (t) en écoute (l) avec les processus (p) et numéros (n)
sudo ss -lntp
```

* `:22` (ou votre nouveau port) : C'est SSH.
* `:80` / `:443` : Serveur Web.
* Si vous voyez un port inconnu, enquêtez !

---
*Guide réalisé par Paulo Rosa.*
