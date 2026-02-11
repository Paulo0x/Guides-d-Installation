# 📂 Transfert de Fichiers : SCP, SFTP & Rsync

Un administrateur système n'utilise jamais de clé USB sur un serveur. Il utilise le réseau. Pour transférer des fichiers (code, logs, scripts), on oublie le vieux FTP non sécurisé et on utilise le protocole SSH, qui est chiffré.

Ce guide couvre la ligne de commande (**SCP**), l'interface graphique (**FileZilla**) et la synchronisation intelligente (**Rsync**).

---

## 1. La commande SCP (Secure Copy)

SCP est l'outil standard. Il s'utilise depuis votre **ordinateur personnel** (le client), et non depuis le serveur.

### 1.1 Envoyer un fichier (Upload)
**Syntaxe :** `scp fichier_source user@serveur:destination`

```bash
# Envoyer index.html dans le dossier personnel de l'utilisateur 'paulo'
scp index.html paulo@192.168.1.50:/home/paulo/

# Si vous utilisez un port SSH différent (ex: 2222)
scp -P 2222 index.html paulo@192.168.1.50:/home/paulo/
```

### 1.2 Récupérer un fichier (Download)
**Syntaxe :** `scp user@serveur:source destination`

```bash
# Récupérer un log du serveur vers le dossier actuel (.) de mon PC
scp paulo@192.168.1.50:/var/log/nginx/error.log .
```

### 1.3 Transférer un dossier complet
Il faut ajouter l'option `-r` (récursif).

```bash
# Envoyer tout le dossier 'mon-site'
scp -r mon-site/ paulo@192.168.1.50:/home/paulo/
```

---

## 2. Le client graphique (FileZilla / WinSCP)

Pour ceux qui préfèrent la souris, FileZilla est parfait, mais il faut bien le configurer.

### 2.1 Configuration

1.  Ouvrez **FileZilla**.
2.  Allez dans **Fichier > Gestionnaire de Sites**.
3.  Cliquez sur **Nouveau Site** et configurez ainsi :
    * **Protocole** : SFTP - SSH File Transfer Protocol (⚠️ Ne choisissez pas FTP !).
    * **Hôte** : IP de votre serveur (ex: `192.168.1.50`).
    * **Type d'authentification** : Fichier de clé (si clé SSH) ou Normale (si mot de passe).
    * **Utilisateur** : Votre user (ex: `paulo`).
4.  Cliquez sur **Connexion**.

---

## 3. Le Piège des Permissions (Erreur "Permission Denied")

C'est l'erreur n°1 des débutants.

**Le Problème :**
Vous essayez d'envoyer un fichier directement dans `/var/www/html` ou `/etc/nginx` via SCP ou FileZilla. Le transfert échoue (rouge).

**La Cause :**
Ces dossiers appartiennent à `root`. Votre utilisateur `paulo` n'a pas le droit d'écrire dedans directement depuis l'extérieur.

**La Solution TSSR :**
La méthode propre se fait en deux temps :
1.  **Transfert :** Envoyez le fichier dans votre dossier personnel (`/home/paulo/`), là où vous avez tous les droits.
2.  **Placement :** Connectez-vous en SSH et déplacez le fichier avec `sudo`.

```bash
# Étape 1 (Sur votre PC) : Envoi vers le home
scp mon-site.zip paulo@192.168.1.50:/home/paulo/

# Étape 2 (Sur le Serveur) : Déplacement avec droits root
ssh paulo@192.168.1.50
sudo mv /home/paulo/mon-site.zip /var/www/html/
```

---

## 4. Rsync : La copie intelligente

SCP est "bête" : il copie tout, même si le fichier existe déjà.
**Rsync** est intelligent : il compare et n'envoie que les différences. C'est l'outil roi pour les sauvegardes.

**Syntaxe :** `rsync -options source destination`

* `-a` : Archive (conserve les permissions, les dates, les propriétaires).
* `-v` : Verbeux (affiche ce qu'il fait).
* `-z` : Compresse les données pendant le transfert (plus rapide).

```bash
# Synchroniser mon dossier local 'projets' vers le serveur
# Si je relance la commande, seuls les fichiers modifiés seront envoyés.
rsync -avz projets/ paulo@192.168.1.50:/home/paulo/backup_projets/
```

---
*Guide réalisé par Paulo Rosa.*
