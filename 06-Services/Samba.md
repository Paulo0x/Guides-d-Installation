# 📂 Samba : Serveur de Fichiers & Partage Windows

Samba est l'implémentation libre du protocole SMB/CIFS. Il permet à un serveur Linux (Debian/Ubuntu) de s'intégrer parfaitement dans un réseau Windows pour partager des fichiers et des imprimantes, tout en gérant finement les droits d'accès utilisateurs.

---

## 1. Prérequis

* **Système** : Un serveur Linux (Debian 12 ou Ubuntu 22.04).
* **Client** : Un PC sous Windows (10 ou 11) connecté au même réseau.
* **Réseau** : Les ports `139` et `445` (TCP) doivent être ouverts.

---

## 2. Installation

Samba est disponible dans les dépôts standards. Nous installons également le client pour tester localement.

### Étape 2.1 : Installation des paquets

Mettez à jour et installez le service :

```bash
sudo apt update
sudo apt install samba samba-common-bin -y
```

### Étape 2.2 : Vérification du service

Assurez-vous que le démon SMB tourne correctement :

```bash
sudo systemctl status smbd
```
*Le statut doit être "Active (running)".*

---

## 3. Configuration

Nous allons créer un dossier partagé sécurisé, accessible uniquement avec un mot de passe.

### Étape 3.1 : Création du dossier

Créez le répertoire qui contiendra vos données et attribuez les droits :

```bash
# Création du dossier
sudo mkdir -p /srv/samba/partage

# Attribution des droits (Lecture/Écriture pour le propriétaire)
sudo chown -R $USER:$USER /srv/samba/partage
sudo chmod -R 770 /srv/samba/partage
```

### Étape 3.2 : Configuration du partage

Sauvegardez le fichier de configuration d'origine par sécurité :

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

Éditez le fichier :

```bash
sudo nano /etc/samba/smb.conf
```

Allez tout à la fin du fichier et ajoutez ce bloc :

```ini
[MonPartage]
   comment = Partage de fichiers Portfolio
   path = /srv/samba/partage
   browseable = yes
   read only = no
   create mask = 0770
   directory mask = 0770
   valid users = @sambashare
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

### Étape 3.3 : Gestion des utilisateurs

Samba possède sa propre base de données d'utilisateurs. Il faut lier votre utilisateur Linux à Samba.

1.  **Création du mot de passe Samba** (différent du mot de passe SSH) :
    ```bash
    sudo smbpasswd -a $USER
    ```
2.  **Ajout au groupe autorisé** :
    ```bash
    sudo usermod -aG sambashare $USER
    ```
3.  **Redémarrage du service** :
    ```bash
    sudo systemctl restart smbd
    ```

---

## 4. Vérification

Testons l'accès depuis un poste client Windows.

### Étape 4.1 : Test local (syntaxe)

Vérifiez que votre fichier de configuration ne contient pas d'erreurs :

```bash
testparm
```
*Appuyez sur Entrée. Si vous voyez "Loaded services file OK", c'est bon.*

### Étape 4.2 : Connexion depuis Windows

1.  Sur votre PC Windows, ouvrez l'explorateur de fichiers.
2.  Dans la barre d'adresse en haut, tapez l'adresse IP de votre serveur :
    `\\IP_DE_VOTRE_SERVEUR` (ex: `\\192.168.1.50`)
3.  Appuyez sur **Entrée**.
4.  Une fenêtre d'authentification s'ouvre :
    * **Nom** : Votre utilisateur Linux (ex: `paulo`).
    * **Mot de passe** : Celui défini avec `smbpasswd` à l'étape 3.3.
5.  **Succès** : Vous voyez le dossier "MonPartage". Vous pouvez créer, modifier et supprimer des fichiers à l'intérieur.

---
*Guide réalisé par Paulo Rosa.*
