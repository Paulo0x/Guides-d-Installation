# 📈 Zabbix 7.0 LTS : Installation du Serveur de Supervision

Zabbix est une solution de monitoring de classe entreprise capable de surveiller des milliers de serveurs, machines virtuelles et équipements réseaux. Ce guide couvre l'installation complète d'un serveur Zabbix 7.0 (LTS) sur Debian 12 avec une base de données MariaDB.

---

## 1. Prérequis

L'installation nécessite un environnement Linux propre :

* **Serveur** : Une machine virtuelle ou physique sous **Debian 12 (Bookworm)**.
* **Ressources** : Minimum 2 vCPU et 4 Go de RAM (recommandé pour la production).
* **Privilèges** : Accès `root` ou `sudo`.
* **Base de Données** : Nous installerons MariaDB durant ce guide.

---

## 2. Installation

Nous allons installer le dépôt officiel Zabbix, le moteur de base de données et les composants serveur.

### Étape 2.1 : Installation du dépôt Zabbix
Les paquets Debian par défaut sont obsolètes. Il faut ajouter le dépôt officiel Zabbix 7.0 LTS.

```bash
# 1. Télécharger le paquet de configuration du dépôt
wget [https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_7.0-2+debian12_all.deb](https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_7.0-2+debian12_all.deb)

# 2. Installer le dépôt
dpkg -i zabbix-release_7.0-2+debian12_all.deb

# 3. Mettre à jour la liste des paquets
apt update
```

### Étape 2.2 : Installation des composants
Nous installons le serveur Zabbix, l'interface web (Frontend), l'agent local et le serveur de base de données MariaDB.

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent mariadb-server -y
```

---

## 3. Configuration

C'est l'étape critique. Nous devons préparer la base de données et connecter Zabbix à celle-ci.

### Étape 3.1 : Création de la Base de Données
Lancez les commandes SQL suivantes pour créer l'utilisateur et la base dédiée.
*(Remplacez 'votre_password_robuste' par un mot de passe fort)*.

```bash
mariadb -u root
```

Une fois dans l'invite SQL (`MariaDB [(none)]>`), tapez ceci ligne par ligne :

```sql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'votre_password_robuste';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit;
```

### Étape 3.2 : Importation du Schéma Initial
Nous devons importer la structure des tables Zabbix dans la base de données vide.
*(Le système vous demandera le mot de passe défini à l'étape précédente)*.

```bash
zcat /usr/share/doc/zabbix-sql-scripts/mysql/server.sql.gz | mariadb -u zabbix -p zabbix
```

*Note : Une fois l'import terminé, désactivez l'option log_bin (sécurité) :*

```bash
mariadb -u root -e "set global log_bin_trust_function_creators = 0;"
```

### Étape 3.3 : Configuration du Serveur Zabbix
Il faut indiquer au serveur Zabbix le mot de passe pour se connecter à la base.

1.  Éditez le fichier de configuration :
    ```bash
    nano /etc/zabbix/zabbix_server.conf
    ```
2.  Cherchez la ligne `DBPassword=` (utilisez `Ctrl+W` pour chercher).
3.  Décommentez-la (enlevez le `#`) et ajoutez votre mot de passe :
    ```ini
    DBPassword=votre_password_robuste
    ```
4.  Sauvegardez (`Ctrl+O`) et quittez (`Ctrl+X`).

### Étape 3.4 : Démarrage des Services
Activez les services pour qu'ils démarrent automatiquement avec le serveur.

```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
```

---

## 4. Vérification

L'installation en ligne de commande est terminée. Passons à l'interface web pour finaliser.

### Étape 4.1 : Accès à l'interface Web
Ouvrez votre navigateur et accédez à :
`http://IP_DE_VOTRE_SERVEUR/zabbix`

Vous devriez voir l'écran de bienvenue "Welcome to Zabbix 7.0".

### Étape 4.2 : Assistant de configuration
1.  **Check of pre-requisites** : Tout doit être en vert (OK).
2.  **Configure DB Connection** : Entrez le mot de passe de la base de données (celui défini à l'étape 3.1).
3.  **Settings** : Donnez un nom à votre serveur (ex: "Zabbix Master").
4.  Terminez l'assistant.

### Étape 4.3 : Connexion
Connectez-vous avec les identifiants par défaut (Attention à la majuscule !) :

* **Utilisateur** : `Admin`
* **Mot de passe** : `zabbix`

**Succès :** Si vous arrivez sur le tableau de bord (Dashboard) "Global View", votre serveur de supervision est opérationnel.

---
*Guide réalisé par Paulo Rosa.*
