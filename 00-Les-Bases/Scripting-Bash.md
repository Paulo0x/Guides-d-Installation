# 📜 Scripting Bash : L'Art de l'Automatisation

Un bon administrateur système est un administrateur paresseux : il écrit des scripts pour faire le travail à sa place. Le **Bash** est le langage de programmation par défaut de Linux. Ce guide vous apprend à créer vos premiers scripts pour automatiser les sauvegardes et la maintenance.

---

## 1. Les Règles d'Or

Pour qu'un fichier texte soit considéré comme un script, il lui faut deux choses :
1.  **Le Shebang** : La première ligne doit être `#!/bin/bash` (dit à Linux quel interpréteur utiliser).
2.  **L'Exécution** : Le fichier doit avoir la permission `x` (`chmod +x script.sh`).

---

## 2. Votre Premier Script (Hello World)

Créons un script simple qui vous dit bonjour et vous donne l'heure.

```bash
# 1. Créer le fichier
nano bonjour.sh
```

**Contenu du fichier :**
```bash
#!/bin/bash

# Ceci est un commentaire (ignoré par le script)
echo "👋 Bonjour $USER !"
echo "Nous sommes le $(date)"
echo "Tu es connecté sur la machine : $(hostname)"
```

**Exécution :**
```bash
# Rendre exécutable
chmod +x bonjour.sh

# Lancer (le ./ est obligatoire pour dire "dossier actuel")
./bonjour.sh
```

---

## 3. Les Variables (La Mémoire)

Les variables permettent de stocker des informations pour les réutiliser.
* **Définition** : `MA_VAR="valeur"` (Pas d'espaces autour du = !).
* **Utilisation** : `$MA_VAR` (Avec un dollar).

```bash
#!/bin/bash

SOURCE="/var/www/html"
DESTINATION="/home/paulo/backups"
DATE_JOUR=$(date +%Y-%m-%d)
NOM_ARCHIVE="site-web-$DATE_JOUR.tar.gz"

echo "Je vais sauvegarder $SOURCE vers $DESTINATION/$NOM_ARCHIVE"
```

---

## 4. Les Conditions (Si... Alors...)

Un script doit pouvoir prendre des décisions.

```bash
#!/bin/bash

# Vérifier si l'utilisateur est root (UID 0)
if [ "$EUID" -ne 0 ]
then
  echo "🚫 Erreur : Ce script doit être lancé avec sudo !"
  exit 1
fi

echo "✅ Vous êtes root, on continue..."

# Vérifier si un dossier existe (-d)
if [ -d "/var/www/html" ]
then
  echo "Le dossier web existe."
else
  echo "Le dossier web est introuvable !"
fi
```

---

## 5. TP Concret : Script de Sauvegarde Automatique

Voici un script professionnel que vous pouvez utiliser réellement. Il sauvegarde votre site web, le compresse, et supprime les sauvegardes vieilles de plus de 7 jours.

**Fichier :** `backup-site.sh`

```bash
#!/bin/bash

# --- CONFIGURATION ---
SOURCE="/var/www/html"
DEST="/home/paulo/backups"
DATE=$(date +%Y-%m-%d_%Hh%M)
LOGFILE="/var/log/mon-backup.log"

# --- DEBUT DU SCRIPT ---
echo "--- Début du backup : $(date) ---" >> $LOGFILE

# 1. Créer le dossier de destination s'il n'existe pas
mkdir -p $DEST

# 2. Créer l'archive (.tar.gz)
# c = create, z = gzip, f = file
echo "Compression en cours..."
tar -czf $DEST/backup-site-$DATE.tar.gz $SOURCE 2>> $LOGFILE

# Vérifier si la commande tar a réussi ($? contient le code de retour, 0 = succès)
if [ $? -eq 0 ]; then
    echo "✅ Sauvegarde réussie : $DEST/backup-site-$DATE.tar.gz"
    echo "Succès : $DEST/backup-site-$DATE.tar.gz" >> $LOGFILE
else
    echo "❌ Échec de la sauvegarde !"
    echo "Échec de la sauvegarde !" >> $LOGFILE
    exit 1
fi

# 3. Nettoyage (Rotation)
# Supprimer les fichiers dans $DEST qui ont plus de (+7) jours
echo "Nettoyage des vieilles archives..."
find $DEST -type f -name "*.tar.gz" -mtime +7 -delete

echo "--- Fin du backup ---" >> $LOGFILE
echo "Terminé."
```

---

## 6. Automatiser avec Cron

Votre script est prêt. Maintenant, faites en sorte qu'il se lance tous les jours à 2h00 du matin sans vous.

```bash
# Ouvrir l'éditeur Cron
crontab -e
```

**Ajouter la ligne :**
```bash
# m h  dom mon dow   command
00 02 * * * /home/paulo/scripts/backup-site.sh
```

---
*Guide réalisé par Paulo Rosa.*
