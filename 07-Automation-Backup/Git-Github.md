# 🐙 Git & GitHub : Maîtrise Totale du Workflow

Git est l'outil standard de gestion de versions. Ce guide couvre le cycle complet : de l'installation à la publication d'un projet existant sur votre propre compte (migration), en passant par les commandes indispensables pour gérer les branches et réparer les erreurs au quotidien.

---

## 1. Prérequis

* **Système** : Linux (Debian/Ubuntu), Windows (Git Bash) ou macOS.
* **Compte** : Un compte GitHub actif.
* **Authentification** : Une clé SSH configurée ou un Token d'accès personnel (PAT).

---

## 2. Installation

Commençons par installer l'outil et définir votre identité numérique pour signer les commits.

### Étape 2.1 : Installation des paquets

Sur une distribution Debian/Ubuntu :

```bash
sudo apt update
sudo apt install git -y
```

### Étape 2.2 : Configuration de l'identité

Git doit savoir qui vous êtes. Ces informations apparaîtront dans l'historique des modifications.

```bash
# Définir votre nom et email
git config --global user.name "Votre Pseudo GitHub"
git config --global user.email "votre_email@exemple.com"

# Définir 'main' comme branche par défaut (standard moderne)
git config --global init.defaultBranch main
```

---

## 3. Scénario : Cloner, Migrer et Publier

C'est le cas d'usage fréquent : récupérer un projet ailleurs, casser le lien avec l'auteur original, et le publier sur **votre** GitHub.

### Étape 3.1 : Récupération (Clone)

Téléchargez le projet source sur votre machine :

```bash
git clone [https://github.com/auteur-original/projet-source.git](https://github.com/auteur-original/projet-source.git)
cd projet-source
```

### Étape 3.2 : Changement d'Origine (Remote)

Nous devons rediriger le projet vers un dépôt vide que vous avez créé sur votre GitHub.

```bash
# 1. Supprimer le lien avec l'auteur original
git remote remove origin

# 2. Ajouter le lien vers VOTRE nouveau dépôt vide
# (Remplacez l'URL par la vôtre)
git remote add origin [https://github.com/VOTRE_PSEUDO/mon-nouveau-projet.git](https://github.com/VOTRE_PSEUDO/mon-nouveau-projet.git)

# 3. Vérifier que l'URL est bien la vôtre
git remote -v
```

### Étape 3.3 : Envoi initial (Push)

Envoyez tout le code sur votre compte :

```bash
# Lier la branche locale 'main' à votre version distante
git push -u origin main
```

---

## 4. Commandes Quotidiennes (Configuration Avancée)

Une fois le projet en place, voici les commandes pour travailler proprement sans casser la production.

### Étape 4.1 : Travailler avec des Branches

Ne codez jamais directement sur `main`. Isolez chaque fonctionnalité.

```bash
# Créer une branche "login" et basculer dessus
git checkout -b feature-login

# ... (Vous faites vos modifications ici) ...

# Revenir sur la branche principale
git checkout main

# Fusionner le travail de "login" dans "main"
git merge feature-login
```

### Étape 4.2 : Le "Panic Button" (Réparations)

Tout le monde fait des erreurs. Voici comment les corriger.

```bash
# Mettre le travail de côté temporairement (Stash) pour changer de branche en urgence
git stash
# Récupérer le travail plus tard
git stash pop

# Annuler les modifications d'un fichier (avant le 'add')
git restore mon_fichier.txt

# Modifier le dernier commit (oubli de fichier ou faute de frappe)
git add fichier_oublie.txt
git commit --amend -m "Message corrigé"
```

### Étape 4.3 : Ignorer des fichiers (.gitignore)

Créez un fichier `.gitignore` à la racine pour éviter d'envoyer des fichiers sensibles ou inutiles.

```text
# Exemple de contenu .gitignore
node_modules/
*.log
.env
.DS_Store
```

---

## 5. Vérification

Assurons-nous que l'historique est propre et que le projet est bien synchronisé.

### Étape 5.1 : Voir l'historique graphique

Utilisez cette commande pour visualiser l'arbre des commits et les branches :

```bash
git log --oneline --graph --decorate --all
```
* **Résultat** : Un arbre ASCII montrant l'évolution du projet.

### Étape 5.2 : Statut du dépôt

```bash
git status
```
* **Succès** : Le message doit indiquer `On branch main` et `Your branch is up to date with 'origin/main'` (si vous avez tout poussé).

---
*Guide réalisé par Paulo Rosa.*
