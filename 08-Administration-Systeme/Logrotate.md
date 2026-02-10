# 📜 Logrotate : Gestion Automatique des Logs

Les journaux système (logs) sont vitaux pour le débogage, mais ils sont aussi dangereux : s'ils ne sont pas gérés, ils grossissent indéfiniment jusqu'à saturer le disque dur (Disaster Recovery). **Logrotate** est l'outil standard sous Linux pour archiver, compresser et supprimer automatiquement les vieux logs.

---

## 1. Prérequis

* **Système** : Une machine Linux (Debian/Ubuntu/CentOS).
* **Service Cible** : Avoir un service qui génère des logs (nous utiliserons l'exemple précédent `mon-super-service`).
* **Privilèges** : Accès `root` ou `sudo`.

---

## 2. Comprendre le Fonctionnement

Logrotate est lancé quotidiennement par une tâche Cron (`/etc/cron.daily/logrotate`).
Sa configuration se divise en deux :
1.  **Global** : `/etc/logrotate.conf` (paramètres par défaut).
2.  **Spécifique** : `/etc/logrotate.d/` (un fichier par service).

---

## 3. Configuration (Création d'une Règle)

Nous allons créer une règle pour gérer le fichier `/tmp/mon-service.log` créé dans le guide Systemd précédent.

### Étape 3.1 : Création du fichier de config

Ne touchez jamais au fichier global. Créez un fichier dédié dans le dossier `.d` :

```bash
sudo nano /etc/logrotate.d/mon-super-service
```

### Étape 3.2 : Définition des règles

Copiez ce contenu. Chaque ligne est importante :

```nginx
/tmp/mon-service.log {
    # Rotation quotidienne
    daily
    
    # Garder 7 fichiers d'archives (donc 1 semaine d'historique)
    rotate 7
    
    # Compresser les archives (.gz) pour gagner de la place
    compress
    
    # Ne pas compresser le log d'hier (utile pour le debugging immédiat)
    delaycompress
    
    # Ignorer si le fichier log est manquant (évite les erreurs)
    missingok
    
    # Ne pas faire de rotation si le fichier est vide
    notifempty
    
    # Créer un nouveau fichier vide avec les bonnes permissions après rotation
    create 640 root root
}
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

---

## 4. Vérification et Test

Il ne faut pas attendre demain pour savoir si ça marche. Nous allons forcer Logrotate à agir maintenant.

### Étape 4.1 : Test à blanc (Dry Run)

Cette commande simule l'action sans rien toucher. Idéal pour vérifier la syntaxe.

```bash
# -d = Debug (Simulation)
sudo logrotate -d /etc/logrotate.d/mon-super-service
```
* **Résultat** : Vous devez voir des lignes comme `rotating pattern: /tmp/mon-service.log` et `planning to rotate log`. S'il y a une erreur, elle s'affichera ici.

### Étape 4.2 : Forcer la rotation (Force)

Appliquons la rotation immédiatement, même si le délai d'un jour n'est pas passé.

```bash
# -f = Force
sudo logrotate -f /etc/logrotate.d/mon-super-service
```

### Étape 4.3 : Vérification des fichiers

Allons voir si l'archivage a eu lieu.

```bash
ls -l /tmp/mon-service*
```

* **Succès** : Vous devriez voir deux fichiers :
    1.  `mon-service.log` (Le nouveau fichier, vide ou presque).
    2.  `mon-service.log.1.gz` (L'archive compressée de l'ancien contenu).

---
*Guide réalisé par Paulo Rosa.*
