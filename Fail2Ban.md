# 🛡️ Fail2Ban : Protection Anti-Brute-Force

Fail2Ban est un framework de prévention d'intrusion qui protège les serveurs Linux contre les attaques par force brute. Il surveille les fichiers journaux (logs) en temps réel et bannit temporairement ou définitivement les adresses IP présentant un comportement malveillant (trop d'échecs de connexion).

---

## 1. Prérequis

* **Système** : Un serveur Linux (Debian, Ubuntu, CentOS...) opérationnel.
* **Service Cible** : Un service à protéger (généralement le service SSH `sshd`).
* **Privilèges** : Accès `root` ou `sudo`.

---

## 2. Installation

Fail2Ban est disponible dans les dépôts officiels de la plupart des distributions.

### Étape 2.1 : Installation du paquet

Mettez à jour votre liste de paquets et installez l'outil :

```bash
sudo apt update
sudo apt install fail2ban -y
```

### Étape 2.2 : Démarrage du service

Assurez-vous que le service est actif et se lance au démarrage :

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

---

## 3. Configuration

La règle d'or de Fail2Ban est de **ne jamais modifier** le fichier `/etc/fail2ban/jail.conf` (car il est écrasé lors des mises à jour). Nous devons créer un fichier "local".

### Étape 3.1 : Création de la configuration locale

Copiez le fichier de configuration par défaut pour créer votre version personnalisée :

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

### Étape 3.2 : Paramétrage de la prison (Jail)

Éditez le fichier local :

```bash
sudo nano /etc/fail2ban/jail.local
```

Recherchez la section `[DEFAULT]` ou `[sshd]` et ajustez les valeurs suivantes pour durcir la sécurité :

```ini
[sshd]
enabled = true
port    = ssh
filter  = sshd
logpath = /var/log/auth.log
backend = systemd

# Nombre d'essais autorisés avant bannissement
maxretry = 3

# Temps pendant lequel l'IP doit commettre les échecs (ex: 3 essais en 10 min)
findtime = 10m

# Durée du bannissement (ici 1 heure)
bantime = 1h

# (Optionnel) Pour bannir définitivement les récidivistes
# bantime.increment = true
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

### Étape 3.3 : Application des changements

Redémarrez le service pour prendre en compte la nouvelle configuration :

```bash
sudo systemctl restart fail2ban
```

---

## 4. Vérification

Il est crucial de vérifier que la prison est active et surveille bien le service.

### Étape 4.1 : Statut du client

Utilisez la commande client pour interroger le serveur Fail2Ban :

```bash
sudo fail2ban-client status sshd
```

* **Succès** : La commande doit retourner le statut "Currently failed" (nombre d'échecs actuels) et "Total banned" (nombre d'IP bannies).

### Étape 4.2 : Test (Simulation)

Pour tester sans vous bannir vous-même :
1.  Ouvrez une **autre** connexion SSH depuis une autre IP (ex: via la 4G de votre téléphone).
2.  Entrez un mauvais mot de passe volontairement 3 fois.
3.  La connexion devrait être coupée ("Connection refused" ou "Timeout").
4.  Sur votre serveur (via votre connexion principale), vérifiez le ban :
    ```bash
    sudo fail2ban-client status sshd
    ```
    *Vous devriez voir 1 IP dans la liste des bannis.*

### Étape 4.3 : Débannir une IP (En cas d'erreur)

Si vous avez besoin de débannir une adresse IP manuellement :

```bash
sudo fail2ban-client set sshd unbanip IP_A_DEBANNIR
```

---
*Guide réalisé par Paulo Rosa.*
