# 💾 Gestion du Stockage : Disques, Partitions et Montage

Contrairement à Windows, Linux n'attribue pas de lettre (C:, D:) aux disques. Il les range dans `/dev/` (ex: `/dev/sdb`) et il faut les "monter" dans un dossier pour les utiliser. Ce guide couvre l'ajout d'un disque, son formatage et son montage persistant.

---

## 1. Identifier les Disques

Vous venez de brancher un nouveau disque dur (physique ou virtuel). Il faut le trouver.

### 1.1 Lister les périphériques de stockage

```bash
lsblk
```
* **sda** : C'est généralement votre disque système (celui où Linux est installé).
* **sdb** : C'est souvent le nouveau disque (regardez la taille pour être sûr).
* **sr0** : C'est le lecteur CD/DVD.

### 1.2 Voir les détails techniques

```bash
sudo fdisk -l
```

---

## 2. Partitionner le Disque

Un disque brut ne peut pas être utilisé. Il faut créer une partition (une case de rangement). Nous utiliserons le disque `/dev/sdb` pour l'exemple. **Attention, toutes les données sur ce disque seront effacées.**

```bash
# Lancer l'outil de partitionnement semi-graphique (plus simple)
sudo cfdisk /dev/sdb
```

1.  **Label Type** : Choisissez `gpt` (standard moderne).
2.  **New** : Créez une nouvelle partition (Prenez toute la taille).
3.  **Write** : Écrivez les changements (tapez `yes` pour confirmer).
4.  **Quit** : Quittez.

Vérifiez avec `lsblk` : vous devriez voir une ligne `sdb1` sous `sdb`.

---

## 3. Formater (Créer le Système de Fichiers)

La partition existe (`sdb1`), mais elle est vide. Il faut tracer les lignes pour ranger les fichiers. On utilise le format **ext4** (le standard Linux).

```bash
# mkfs = Make FileSystem
sudo mkfs.ext4 /dev/sdb1
```

---

## 4. Le Montage (Mount)

Pour utiliser ce disque, on doit l'attacher à un dossier vide.

### 4.1 Montage Temporaire

```bash
# 1. Créer le point de montage (le dossier d'accès)
sudo mkdir -p /mnt/data

# 2. Attacher le disque au dossier
sudo mount /dev/sdb1 /mnt/data

# 3. Vérifier
df -h
```
*Vous devriez voir `/dev/sdb1` monté sur `/mnt/data` avec la bonne taille.*

### 4.2 Problème de Permissions

Par défaut, seul `root` peut écrire sur un disque nouvellement monté. Donnons les droits à notre utilisateur (ex: paulo).

```bash
sudo chown -R paulo:paulo /mnt/data
# Maintenant, vous pouvez créer des fichiers dedans.
touch /mnt/data/test.txt
```

---

## 5. Le Montage Persistant (/etc/fstab)

Si vous redémarrez maintenant, le montage disparaît. Pour qu'il revienne automatiquement au démarrage, il faut l'inscrire dans le fichier `/etc/fstab`.

**Règle d'or** : On n'utilise jamais le nom `/dev/sdb1` (car il peut changer si on change le câble de port). On utilise l'**UUID** (l'empreinte digitale unique du disque).

### 5.1 Trouver l'UUID

```bash
sudo blkid
```
*Copiez la suite de caractères ressemblant à `UUID="a1b2c3d4-..."` correspondant à `/dev/sdb1` (sans les guillemets).*

### 5.2 Modifier fstab

```bash
sudo nano /etc/fstab
```

Ajoutez cette ligne à la fin du fichier :

```ini
# <Système de fichiers>  <Point de montage>  <Type>  <Options>      <Dump>  <Pass>
UUID=votre-uuid-ici      /mnt/data           ext4    defaults        0       2
```

### 5.3 Test Vital (Avant de redémarrer !)

Si vous faites une erreur dans fstab, le serveur ne redémarrera pas. Testez toujours votre configuration :

```bash
# 1. Démonter pour tester
sudo umount /mnt/data

# 2. Tout remonter selon le fichier fstab
sudo mount -a

# 3. Vérifier que c'est revenu
df -h
```
*Si `mount -a` ne renvoie aucune erreur et que le disque est là, c'est gagné. Vous pouvez redémarrer en toute sécurité.*

---
*Guide réalisé par Paulo Rosa.*
