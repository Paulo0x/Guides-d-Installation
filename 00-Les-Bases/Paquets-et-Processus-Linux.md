# 📦 Paquets & Processus : Installation et Surveillance

Un serveur Linux est modulaire : on installe uniquement ce dont on a besoin. En tant que TSSR, vous devez maîtriser `apt` (pour Debian/Ubuntu) pour gérer les logiciels, et savoir utiliser `top` ou `htop` pour surveiller la charge système.

---

## 1. Gestion des Logiciels (APT)

Sous Linux, on ne télécharge pas des `.exe` sur Internet. On utilise des dépôts sécurisés.

### 1.1 Mettre à jour le système

C'est la première chose à faire sur un nouveau serveur.

```bash
# 1. Rafraîchir la liste des logiciels disponibles (Télécharge le catalogue)
sudo apt update

# 2. Installer les mises à jour (Applique les changements)
sudo apt upgrade -y
```

### 1.2 Installer et Supprimer

```bash
# Installer un paquet (ex: htop, git, curl)
sudo apt install htop git curl -y

# Supprimer un paquet (Garde les fichiers de config)
sudo apt remove nano

# Tout supprimer (Logiciel + Configuration) -> Le "Nettoyage complet"
sudo apt purge apache2

# Nettoyer les dépendances orphelines (Paquets installés qui ne servent plus à rien)
sudo apt autoremove -y
```

### 1.3 Rechercher un paquet

Vous ne connaissez pas le nom exact ?

```bash
apt search python3
```

---

## 2. Surveillance Système (Monitoring)

Votre serveur est lent ? C'est ici qu'on regarde pourquoi.

### 2.1 La commande "top" (L'ancêtre)

Installé partout par défaut.

```bash
top
```
* **Load Average** (en haut à droite) : La charge du processeur sur 1min, 5min, 15min. Si ce chiffre dépasse le nombre de cœurs de votre CPU, le serveur sature.
* **%CPU / %MEM** : Qui consomme le plus ?
* *Appuyez sur `q` pour quitter.*

### 2.2 La commande "htop" (Le moderne)

Plus visuel, avec des barres de couleur et la souris active. (À installer : `sudo apt install htop`).

```bash
htop
```
* Utilisez `F6` pour trier (par RAM, par CPU...).
* Utilisez `F9` pour tuer un processus planté.

---

## 3. Gestion des Processus (PS & Kill)

Un programme a planté et refuse de se fermer ? On va le forcer.

### 3.1 Lister les processus

```bash
# Voir tous les processus (u = user, x = background, a = all)
ps aux | grep nginx
```
*La colonne importante est le **PID** (Process ID), le numéro unique du processus (ex: 1234).*

### 3.2 Tuer un processus (Kill)

On utilise le PID trouvé juste avant.

```bash
# Demander poliment de fermer (SIGTERM - Signal 15)
sudo kill 1234

# Forcer la fermeture immédiate (SIGKILL - Signal 9) -> "Le tir à vue"
# À utiliser si la méthode douce ne marche pas
sudo kill -9 1234
```

---

## 4. Tâches Planifiées (Cron)

Un TSSR automatise tout. Cron est le réveil-matin du serveur.

### 4.1 Éditer les tâches

```bash
crontab -e
```

### 4.2 La syntaxe magique

Chaque ligne correspond à une tâche.
Structure : `Minute  Heure  JourMois  Mois  JourSemaine  Commande`

**Exemples concrets :**

```bash
# Lancer un backup tous les jours à 03h30 du matin
30 03 * * * /home/paulo/scripts/backup.sh

# Redémarrer le service web tous les lundis à 6h00
00 06 * * 1 systemctl restart nginx

# Écrire "Coucou" dans un fichier toutes les 5 minutes
*/5 * * * * echo "Coucou" >> /tmp/test.txt
```

*(Sauvegardez avec `Ctrl+O` pour activer la tâche).*

---
*Guide réalisé par Paulo Rosa.*
