# 🛡️ CrowdSec : Sécurité Collaborative (IPS)

CrowdSec est un système de prévention d'intrusion (IPS) moderne et collaboratif. Contrairement à Fail2Ban qui agit isolément, CrowdSec analyse les comportements, détecte les attaques (brute-force, scan de ports, web attacks) et partage les IP malveillantes avec la communauté mondiale. Si une IP est signalée comme dangereuse par le réseau, votre serveur la bloque instantanément.

---

## 1. Prérequis

* **Système** : Une machine Debian 11/12 ou Ubuntu 22.04.
* **Ports** : Accès sortant vers Internet (pour récupérer les listes noires).
* **Conflits** : Il est possible de faire cohabiter CrowdSec et Fail2Ban, mais pour ce guide, nous assumons que CrowdSec est le seul maître à bord.

---

## 2. Installation de l'Agent

L'agent est la partie "Cerveau" : il lit les logs et détecte les attaques.

### Étape 2.1 : Ajout du dépôt officiel

```bash
# Installation des dépendances
sudo apt update && sudo apt install -y curl gnupg

# Ajout du dépôt CrowdSec (Script officiel)
curl -s [https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh](https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh) | sudo bash
```

### Étape 2.2 : Installation du paquet

```bash
sudo apt install crowdsec -y
```
*Durant l'installation, CrowdSec détectera automatiquement vos services (SSH, Nginx, Apache...) et configurera les scénarios adaptés.*

---

## 3. Installation du Bouncer (Pare-feu)

L'agent détecte, mais ne bloque pas. Pour bloquer, il faut un "Bouncer". Nous installerons le bouncer "iptables" qui coupe la connexion au niveau du pare-feu Linux.

### Étape 3.1 : Installation

```bash
sudo apt install crowdsec-firewall-bouncer-iptables -y
```

### Étape 3.2 : Vérification du fonctionnement

Une fois installé, vérifions que le bouncer discute bien avec l'agent :

```bash
sudo cscli bouncers list
```
* **Succès** : Vous devez voir une ligne avec `crowdsec-firewall-bouncer` et le statut `✔️` (Valid).

---

## 4. Utilisation & Commandes

CrowdSec se gère via la ligne de commande `cscli`.

### Étape 4.1 : Tester la détection (Simulation)

Ne nous bannissons pas nous-mêmes ! Utilisons un outil pour simuler une attaque sans danger (ou essayez de vous connecter en SSH avec un mauvais mot de passe depuis une autre connexion, ex: 4G).

Pour voir ce qui se passe en temps réel :
```bash
sudo cscli metrics
```

Pour voir les décisions de bannissement actives :
```bash
sudo cscli decisions list
```

### Étape 4.2 : Bannir une IP manuellement

Si vous voulez forcer le blocage d'une IP (ex: 1.2.3.4) :

```bash
sudo cscli decisions add --ip 1.2.3.4 --duration 4h --reason "Test manuel"
```

Vérifiez qu'elle est bien bloquée :
```bash
sudo cscli decisions list
```

Pour la débannir :
```bash
sudo cscli decisions delete --ip 1.2.3.4
```

---

## 5. Connexion à la Console (Optionnel mais Recommandé)

CrowdSec propose une interface web gratuite (Console) pour visualiser vos alertes.

1.  Créez un compte sur [app.crowdsec.net](https://app.crowdsec.net).
2.  Sur votre serveur, lancez la commande d'enrôlement :
    ```bash
    sudo cscli console enroll VOTRE_CLE_D_ENROLEMENT
    ```
3.  **Succès** : Votre serveur apparaît désormais sur le tableau de bord web, et vous bénéficiez des "Community Blocklists" (les IP les plus dangereuses du moment sont poussées vers votre serveur).

---

## 6. Maintenance

Mettre à jour les scénarios de détection et la base de données d'IP :

```bash
sudo cscli hub update
sudo cscli hub upgrade
```

---
*Guide réalisé par Paulo Rosa.*
