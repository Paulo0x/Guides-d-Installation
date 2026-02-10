# 💾 LVM : Gestion Flexible du Stockage

LVM (Logical Volume Manager) est une couche d'abstraction entre vos disques physiques et le système de fichiers. Contrairement au partitionnement classique (figé), LVM permet de redimensionner, étendre ou fusionner des volumes "à chaud", sans redémarrer le serveur. C'est le standard industriel pour la gestion du stockage sous Linux.

---

## 1. Prérequis

* **Système** : Une machine Linux (Debian/Ubuntu).
* **Matériel** : Pour ce guide, il faut ajouter un **second disque dur virtuel** (ex: 10 Go) à votre machine virtuelle.
* **Privilèges** : Accès `root` ou `sudo` (car nous touchons aux disques).

---

## 2. Installation

L'outil est souvent pré-installé, mais assurons-nous d'avoir les commandes nécessaires.

### Étape 2.1 : Installation du paquet

```bash
sudo apt update
sudo apt install lvm2 -y
```

### Étape 2.2 : Identification du nouveau disque

Vérifiez que votre second disque est bien détecté (généralement `/dev/sdb`) :

```bash
lsblk
```
*Recherchez un disque de la taille que vous avez ajoutée (ex: 10G) qui n'a pas encore de partitions.*

---

## 3. Configuration

La logique LVM se fait en 3 couches : **PV** (Disque Physique) > **VG** (Groupe de Volumes) > **LV** (Volume Logique).

### Étape 3.1 : Création du Volume Physique (PV)

Nous déclarons le disque `/dev/sdb` comme étant géré par LVM :

```bash
sudo pvcreate /dev/sdb
# Doit retourner : Physical volume "/dev/sdb" successfully created.
```

### Étape 3.2 : Création du Groupe de Volumes (VG)

Nous créons un "sac" de stockage nommé `vg_data` utilisant ce disque :

```bash
sudo vgcreate vg_data /dev/sdb
# Doit retourner : Volume group "vg_data" successfully created.
```

### Étape 3.3 : Création du Volume Logique (LV)

Nous allons découper une "part" de 5 Go dans ce sac pour nos données :

```bash
# -L : Taille (5G)
# -n : Nom du volume (lv_backups)
sudo lvcreate -L 5G -n lv_backups vg_data
```

### Étape 3.4 : Formatage et Montage

Le volume est créé, il faut maintenant le formater (créer le système de fichiers) et le monter.

```bash
# Formatage en ext4
sudo mkfs.ext4 /dev/vg_data/lv_backups

# Création du point de montage
sudo mkdir -p /mnt/backups

# Montage
sudo mount /dev/vg_data/lv_backups /mnt/backups

# Vérification
df -h | grep backups
```
* **Succès** : Vous devez voir une partition de 5 Go montée sur `/mnt/backups`.

---

## 4. Vérification (Le "Magic Trick")

C'est ici que LVM brille : nous allons agrandir cette partition alors qu'elle est utilisée.

### Étape 4.1 : Simulation de saturation

Imaginez que vous n'avez plus de place. Vérifiez la taille actuelle :

```bash
sudo lvs
# Doit afficher 5.00g
```

### Étape 4.2 : Extension "À chaud"

Nous allons ajouter 2 Go supplémentaires (pris sur l'espace restant du groupe `vg_data`).

```bash
# 1. Agrandir le conteneur LVM (+2Go)
sudo lvextend -L +2G /dev/vg_data/lv_backups

# 2. Agrandir le système de fichiers (pour qu'il voit la nouvelle place)
sudo resize2fs /dev/vg_data/lv_backups
```

### Étape 4.3 : Résultat final

Vérifiez à nouveau la taille disponible :

```bash
df -h /mnt/backups
```
* **Succès** : Le disque fait maintenant **7 Go** (5 + 2) sans avoir démonté le disque ni redémarré la machine. C'est la puissance de LVM.

---
*Guide réalisé par Paulo Rosa.*
