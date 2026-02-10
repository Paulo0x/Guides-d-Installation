# 🚀 Ansible : Automatisation & Infrastructure as Code

Ansible est le standard industriel pour la gestion de configuration et le déploiement d'applications. Contrairement aux scripts impératifs, il est **déclaratif** (on décrit l'état final souhaité), **idempotent** (on peut le lancer 100 fois sans casser le système) et **sans agent** (il utilise simplement SSH). Ce guide configure un serveur de contrôle pour piloter et déployer des nœuds distants.

---

## 1. Prérequis

* **Nœud de Contrôle** : Une machine Debian/Ubuntu (votre PC ou VM de gestion).
* **Nœud Cible** : Une ou plusieurs machines à configurer (IP connue).
* **Accès SSH** : Une paire de clés SSH doit être configurée. Le Nœud de Contrôle doit pouvoir se connecter au Nœud Cible sans mot de passe.
* **Privilèges** : L'utilisateur distant doit avoir les droits `sudo` (avec ou sans mot de passe).

---

## 2. Installation

Nous préparons l'environnement de travail sur le Nœud de Contrôle.

### Étape 2.1 : Installation du moteur

Mettez à jour les dépôts et installez Ansible :

```bash
sudo apt update
sudo apt install ansible -y
```

### Étape 2.2 : Structure du Projet

Les bonnes pratiques DevOps imposent une structure propre. Créez un dossier projet :

```bash
mkdir -p ~/ansible-project
cd ~/ansible-project
```

### Étape 2.3 : Configuration Locale (ansible.cfg)

Pour éviter les avertissements SSH et définir les paramètres par défaut, créez un fichier de configuration à la racine du projet :

```bash
nano ansible.cfg
```

Ajoutez ce contenu pour optimiser l'exécution :

```ini
[defaults]
# Emplacement de l'inventaire par défaut
inventory = ./inventory.ini
# Désactive la vérification des clés hôtes (évite le prompt "yes/no")
host_key_checking = False
# Utilisateur par défaut pour la connexion SSH
remote_user = votre_utilisateur_linux
# Désactive la création de fichiers .retry en cas d'erreur
retry_files_enabled = False
# Affiche le temps d'exécution des tâches (optionnel, pour le debug)
callbacks_enabled = profile_tasks
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

---

## 3. Configuration de l'Inventaire & Playbook

### Étape 3.1 : L'Inventaire (inventory.ini)

Ce fichier liste vos machines et les groupe par fonction.

```bash
nano inventory.ini
```

Ajoutez vos serveurs cibles (remplacez les IP) :

```ini
[webservers]
192.168.1.50

[dbservers]
192.168.1.51

[all:vars]
# Variable globale : force l'interpréteur Python 3
ansible_python_interpreter=/usr/bin/python3
```

*(Sauvegardez et quittez)*.

### Étape 3.2 : Test de Connectivité (Ad-Hoc)

Avant de scripter, on vérifie que Ansible "voit" les serveurs via SSH :

```bash
ansible all -m ping
```
* **Succès** : Vous devez obtenir une réponse verte `SUCCESS => {"ping": "pong"}`.

### Étape 3.3 : Le Playbook (site.yml)

Nous allons écrire un scénario complet qui : met à jour le serveur, installe Nginx, le démarre et déploie une page d'accueil personnalisée.

```bash
nano site.yml
```

Copiez ce code YAML (Attention aux indentations !) :

```yaml
---
- name: Déploiement d'un Serveur Web Nginx
  hosts: webservers
  become: yes  # Élévation de privilèges (sudo)

  tasks:
    - name: 1. Mise à jour du cache APT
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: 2. Installation de Nginx et outils
      apt:
        name:
          - nginx
          - curl
          - git
        state: present

    - name: 3. Démarrage et activation du service Nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: 4. Déploiement d'une page d'accueil personnalisée
      copy:
        dest: /var/www/html/index.html
        content: |
          <h1>Serveur Configuré par Ansible</h1>
          <p>Déploiement automatique réussi par Paulo Rosa.</p>
          <p>Dernière mise à jour : {{ ansible_date_time.date }}</p>
        mode: '0644'
      notify: Redémarrer Nginx

  handlers:
    # Ce bloc ne s'exécute que si la tâche "Déploiement..." modifie quelque chose
    - name: Redémarrer Nginx
      service:
        name: nginx
        state: restarted
```

*(Sauvegardez et quittez)*.

---

## 4. Vérification

C'est le moment de lancer l'automatisation.

### Étape 4.1 : Exécution du Playbook

Lancez la commande suivante.
*Note : Si votre utilisateur distant demande un mot de passe pour `sudo`, ajoutez l'option `-K`.*

```bash
ansible-playbook site.yml
```

### Étape 4.2 : Analyse du retour

Observez les couleurs dans le terminal :
* **Vert (ok)** : Rien n'a changé (état déjà conforme).
* **Jaune (changed)** : Ansible a appliqué une modification.
* **Rouge (failed)** : Erreur critique (connexion, droits...).

### Étape 4.3 : Preuve fonctionnelle

Vérifiez que le serveur Web répond bien avec votre page personnalisée :

```bash
curl [http://192.168.1.50](http://192.168.1.50)
```
* **Succès** : Le terminal doit afficher le code HTML contenant `<p>Déploiement automatique réussi par Paulo Rosa.</p>`.

---
*Guide réalisé par Paulo Rosa.*
