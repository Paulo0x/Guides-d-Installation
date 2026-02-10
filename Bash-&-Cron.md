# 📜 Bash Scripting : Automatisation & Tâches Cron

L'automatisation est le cœur du métier d'Administrateur Système. Ce guide vous apprend à créer un script Shell (Bash) pour automatiser la maintenance de votre serveur (mise à jour, nettoyage) et à planifier son exécution automatique toutes les nuits grâce au démon "Cron".

---

## 1. Prérequis

* **Système** : Un serveur Linux (Debian/Ubuntu).
* **Compétence** : Savoir utiliser un éditeur de texte (nano/vim).
* **Privilèges** : Accès `root` ou `sudo` (car nous allons toucher aux mises à jour système).

---

## 2. Création du Script

Nous allons créer un script qui met à jour la liste des paquets, installe les mises à jour de sécurité, nettoie les fichiers inutiles et enregistre tout cela dans un fichier journal (log).

### Étape 2.1 : Création du fichier

Créez un dossier pour vos scripts personnels (bonne pratique) :

```bash
mkdir -p /usr/local/scripts
nano /usr/local/scripts/auto_update.sh
```

### Étape 2.2 : Écriture du code

Copiez ce contenu dans le fichier.
*La première ligne `#!/bin/bash` (le Shebang) est obligatoire pour indiquer au système comment lire le fichier.*

```bash
#!/bin/bash

# Définition de la date pour les logs
DATE=$(date "+%Y-%m-%d %H:%M:%S")
LOG_FILE="/var/log/auto_update.log"

echo "--- DÉBUT MAINTENANCE : $DATE ---" >> $LOG_FILE

# 1. Mise à jour de la liste des paquets
echo "[INFO] Mise à jour des dépôts..." >> $LOG_FILE
apt-get update -q >> $LOG_FILE 2>&1

# 2. Installation des mises à jour (sans confirmation -y)
echo "[INFO] Installation des mises à jour..." >> $LOG_FILE
apt-get upgrade -y -q >> $LOG_FILE 2>&1

# 3. Nettoyage des paquets orphelins (fichiers inutiles)
echo "[INFO] Nettoyage du système..." >> $LOG_FILE
apt-get autoremove -y -q >> $LOG_FILE 2>&1
apt-get autoclean -q >> $LOG_FILE 2>&1

echo "--- FIN MAINTENANCE : $(date "+%H:%M:%S") ---" >> $LOG_FILE
echo "" >> $LOG_FILE
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

### Étape 2.3 : Permissions d'exécution

Par défaut, un fichier texte ne peut pas être exécuté. Il faut lui donner le droit "x" (eXecutable).

```bash
sudo chmod +x /usr/local/scripts/auto_update.sh
```

### Étape 2.4 : Test manuel

Avant d'automatiser, vérifiez que le script fonctionne en le lançant manuellement :

```bash
sudo /usr/local/scripts/auto_update.sh
```
*Cela peut prendre une minute. Si le curseur revient sans erreur, c'est bon signe.*

---

## 3. Planification (Cron)

Le démon "Cron" est le réveil-matin du serveur. Il exécute des commandes à des heures précises.

### Étape 3.1 : Édition de la table Cron

Nous utilisons la table de l'utilisateur root, car les commandes `apt` nécessitent des droits élevés.

```bash
sudo crontab -e
```
*(Si on vous demande de choisir un éditeur, tapez 1 pour Nano).*

### Étape 3.2 : Ajout de la tâche

Allez tout en bas du fichier et ajoutez cette ligne :

```cron
# M h  dom mon dow   command
30 03 * * * /usr/local/scripts/auto_update.sh
```

**Explication de la syntaxe :**
* `30` : À la 30ème minute.
* `03` : De la 3ème heure (3h du matin).
* `*` : Tous les jours du mois.
* `*` : Tous les mois.
* `*` : Tous les jours de la semaine.
* La commande à exécuter.

*(Sauvegardez et quittez. Le système vous dira "crontab: installing new crontab").*

---

## 4. Vérification

Comment savoir si votre script a bien tourné pendant la nuit ?

### Étape 4.1 : Vérification des Logs (Le lendemain)

Attendez le lendemain matin (ou changez l'heure du cron pour tester dans 5 minutes). Ensuite, lisez le fichier journal que votre script a créé :

```bash
cat /var/log/auto_update.log
```

* **Succès** : Vous devriez voir quelque chose comme :
> *--- DÉBUT MAINTENANCE : 2024-XX-XX 03:30:01 ---*
> *[INFO] Mise à jour des dépôts...*
> *...*
> *--- FIN MAINTENANCE : 03:31:45 ---*

Cela prouve que votre serveur est désormais autonome pour sa maintenance de base.

---
*Guide réalisé par Paulo Rosa.*
