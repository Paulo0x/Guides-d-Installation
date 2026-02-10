# 💾 Restic : Sauvegardes Chiffrées & Dédupliquées

Restic est une solution de sauvegarde moderne, rapide et sécurisée. Contrairement aux outils classiques, il chiffre les données par défaut, gère la déduplication (économie d'espace disque) et permet d'envoyer les sauvegardes vers le Cloud (S3, Backblaze) ou un serveur SFTP aussi simplement que sur un disque local.

---

## 1. Prérequis

* **Système** : Une machine Linux (Serveur à sauvegarder).
* **Stockage** : Un second disque, une clé USB montée, ou un serveur distant (pour ce guide, nous simulerons un dépôt local).
* **Privilèges** : Accès `root` ou `sudo` pour lire tous les fichiers.

---

## 2. Installation

Restic est disponible dans les dépôts officiels de Debian/Ubuntu, mais il est recommandé de récupérer la dernière version binaire pour avoir les dernières fonctionnalités.

### Étape 2.1 : Installation via le gestionnaire de paquets
Pour faire simple et robuste :

```bash
sudo apt update
sudo apt install restic -y
```

### Étape 2.2 : Vérification
Assurez-vous que l'outil est prêt :

```bash
restic version
```

---

## 3. Initialisation du Dépôt (Repository)

Avant de sauvegarder, il faut créer le "coffre-fort" (repository) où seront stockées les données.

### Étape 3.1 : Création du dossier de stockage
Nous allons simuler un disque externe dans `/srv/backup` :

```bash
sudo mkdir -p /srv/backup
sudo chown $USER:$USER /srv/backup
```

### Étape 3.2 : Initialisation chiffrée
Cette commande crée la structure de la base de données.
**Attention :** Vous DEVEZ retenir le mot de passe, sinon vos données sont perdues à jamais.

```bash
restic init --repo /srv/backup
```
*Entrez un mot de passe solide quand il vous le demande.*

---

## 4. Utilisation (Backup & Restore)

### Étape 4.1 : Créer une sauvegarde (Snapshot)
Sauvegardons le dossier `/etc` (configurations système) pour l'exemple.
*L'argument `-r` spécifie le chemin du dépôt.*

```bash
sudo restic -r /srv/backup backup /etc
```
*Tapez le mot de passe du dépôt.*

**Notez la rapidité :** La première fois, Restic copie tout. Relancez la commande une seconde fois : elle sera instantanée car Restic voit que rien n'a changé (Déduplication).

### Étape 4.2 : Lister les sauvegardes
Pour voir l'historique de vos "Snapshots" :

```bash
sudo restic -r /srv/backup snapshots
```
*Vous verrez une liste avec des ID (ex: `a1b2c3d4`), la date et le chemin sauvegardé.*

### Étape 4.3 : Restauration de données
Imaginons que vous ayez effacé un fichier critique. Restaurons la dernière version de `/etc` dans un dossier temporaire pour récupérer le fichier.

```bash
# Création du dossier de restauration
mkdir /tmp/restore-test

# Restauration du dernier snapshot ('latest')
sudo restic -r /srv/backup restore latest --target /tmp/restore-test
```

---

## 5. Vérification

Prouvons que la sauvegarde est intègre et utilisable.

### Étape 5.1 : Contrôle des fichiers restaurés
Vérifiez que vos fichiers sont bien revenus dans le dossier temporaire :

```bash
ls -l /tmp/restore-test/etc/hostname
```
* **Succès** : Si le fichier s'affiche, votre système de sauvegarde est opérationnel.*

### Étape 5.2 : Vérification de l'intégrité du dépôt
Restic possède une fonction puissante pour vérifier qu'aucun fichier de sauvegarde n'est corrompu (bit rot) :

```bash
sudo restic -r /srv/backup check
```
* **Succès** : Le message final doit être `no errors were found`.

---
*Guide réalisé par Paulo Rosa.*
