# 🐧 Linux : Les Bases Indispensables du TSSR

Avant de monter des serveurs complexes, un Technicien Système doit maîtriser son environnement. En entreprise, les serveurs n'ont pas d'interface graphique (GUI). Tout se fait en ligne de commande (CLI). Ce guide couvre la survie, la gestion des utilisateurs et la sécurité des fichiers.

---

## 1. Navigation & Manipulation de Fichiers

Savoir où l'on est et comment bouger est la base absolue.

### 1.1 Les Commandes de Survie

* `pwd` : **P**rint **W**orking **D**irectory (Où suis-je ?).
* `ls -lah` : Lister tout (**a**ll), en format liste (**l**) et taille lisible (**h**uman readable).
* `cd dossier` : Entrer dans un dossier.
* `cd ..` : Remonter d'un cran en arrière.
* `cd ~` : Revenir à son dossier personnel (Home).

### 1.2 Créer et Supprimer

```bash
# Créer une arborescence complète d'un coup (-p)
mkdir -p /tmp/projet/dossier1/sous-dossier

# Créer un fichier vide
touch /tmp/projet/fichier_vide.txt

# Copier un fichier (cp source destination)
cp /tmp/projet/fichier_vide.txt /tmp/projet/fichier_copie.txt

# Déplacer ou Renommer (mv source destination)
mv /tmp/projet/fichier_copie.txt /tmp/projet/fichier_final.txt

# Supprimer un dossier et tout son contenu (DANGER !)
# -r = récursif, -f = force (sans confirmation)
rm -rf /tmp/projet
```

---

## 2. Édition de Fichiers (Nano)

Oubliez la souris. Sur un serveur, on utilise `nano` (ou `vim` pour les experts).

```bash
nano mon_fichier.txt
```

**Raccourcis vitaux pour Nano :**
* `Ctrl + O` : Sauvegarder (Write **O**ut).
* `Entrée` : Confirmer le nom du fichier.
* `Ctrl + X` : Quitter (E**x**it).
* `Ctrl + W` : Chercher un mot (**W**here is).

---

## 3. Gestion des Utilisateurs (Sécurité)

On ne travaille **jamais** en `root` (le super-administrateur) au quotidien. On crée un utilisateur nominatif.

### 3.1 Création d'un utilisateur

```bash
# Créer l'utilisateur "paulo" (crée aussi son dossier /home/paulo)
sudo adduser paulo
```
*Répondez aux questions (mot de passe, nom...).*

### 3.2 Donner les droits d'administration (Sudo)

Pour que "paulo" puisse lancer des commandes `root`, il faut l'ajouter au groupe `sudo`.

```bash
sudo usermod -aG sudo paulo
```
*(**a**ppend **G**roup : ajouter au groupe sans le retirer des autres).*

### 3.3 Se connecter en tant que cet utilisateur

```bash
su - paulo
```

---

## 4. Droits et Permissions (chmod & chown)

C'est souvent là que les débutants bloquent. Chaque fichier appartient à un **User** (Propriétaire) et un **Group**.
Les droits sont : **r** (lecture), **w** (écriture), **x** (exécution).

### 4.1 Changer le propriétaire (chown)

```bash
# Rendre 'paulo' propriétaire du fichier
# Syntaxe : chown user:group fichier
sudo chown paulo:paulo /var/www/html/index.html

# Le faire pour tout un dossier (-R = Récursif)
sudo chown -R paulo:paulo /var/www/html/
```

### 4.2 Changer les permissions (chmod)

On utilise souvent la notation chiffrée :
* **7** = Lire + Écrire + Exécuter (rwx)
* **6** = Lire + Écrire (rw-)
* **5** = Lire + Exécuter (r-x)
* **4** = Lire seulement (r--)
* **0** = Aucun droit (---)

L'ordre est toujours : **Propriétaire / Groupe / Les Autres**.

```bash
# Cas 1 : Fichier privé (Seul moi je lis et écris)
chmod 600 fichier_secret.txt

# Cas 2 : Script exécutable (Tout le monde peut le lancer)
chmod 755 mon_script.sh
# (7 pour moi, 5 pour le groupe, 5 pour les autres)

# Cas 3 : La configuration web standard (Dossiers en 755, Fichiers en 644)
find /var/www -type d -exec chmod 755 {} \;
find /var/www -type f -exec chmod 644 {} \;
```

---

## 5. Diagnostic Réseau de Base

Le serveur ne répond pas ? Voici les premiers réflexes.

### 5.1 Vérifier son IP

```bash
ip a
```
*Cherchez `inet` sur votre interface (souvent `eth0` ou `ens18`).*

### 5.2 Tester la connexion (Ping)

```bash
# Est-ce que je sors sur Internet ?
ping -c 4 8.8.8.8

# Est-ce que le DNS fonctionne ? (Est-ce que je traduis google.com en IP ?)
ping -c 4 google.com
```

### 5.3 Voir les ports ouverts (Listening)

Quels services écoutent sur mon serveur ?

```bash
# s = statistiques, s = sockets, l = listening, n = numérique (pas de noms), t = tcp, p = process
sudo ss -lntp
```
*Si vous voyez `:80`, c'est qu'un serveur Web tourne. Si vous voyez `:22`, c'est SSH.*

---
*Guide réalisé par Paulo Rosa.*
