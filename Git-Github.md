# 🛠️ Git : Commandes Utiles & Gestion Quotidienne

Une fois les bases acquises, un Administrateur Système ou un Développeur doit savoir naviguer dans l'historique, créer des branches pour ne pas casser la production, et revenir en arrière en cas d'erreur. Ce guide recense les commandes indispensables pour survivre au quotidien.

---

## 1. Surveiller et Comprendre l'État

Avant de toucher à quoi que ce soit, il faut savoir où on est.

### Étape 1.1 : L'historique compact
La commande `git log` par défaut est trop verbeuse. Utilisez cette version pour avoir une vue d'ensemble propre (une ligne par commit) :

```bash
git log --oneline --graph --decorate --all
```
* **Résultat** : Vous verrez un arbre ASCII coloré montrant les branches et les fusions.

### Étape 1.2 : Voir les différences
Pour voir exactement ce que vous avez modifié avant de faire un `add` :

```bash
git diff
```
* **Résultat** : Affiche les lignes supprimées en rouge (-) et les ajouts en vert (+).

---

## 2. Travailler avec des Branches (Sécurité)

Ne travaillez jamais directement sur `main` (ou `master`). Créez une branche pour chaque nouvelle fonctionnalité.

### Étape 2.1 : Créer et changer de branche
La commande moderne pour créer une branche et basculer dessus immédiatement :

```bash
# Créer une branche nommée "feature-login" et basculer dessus
git checkout -b feature-login
```

### Étape 2.2 : Revenir sur la branche principale
Une fois le travail fini, revenez à la base :

```bash
git checkout main
```

### Étape 2.3 : Supprimer une branche
Une fois fusionnée, la branche ne sert plus à rien :

```bash
git branch -d feature-login
```

---

## 3. Mettre à Jour et Fusionner

Le travail d'équipe implique de récupérer le code des autres et de l'intégrer au vôtre.

### Étape 3.1 : Récupérer les dernières modifs (Pull)
Avant de commencer à travailler, mettez toujours votre local à jour par rapport au serveur (GitHub) :

```bash
git pull origin main
```

### Étape 3.2 : Fusionner une branche (Merge)
Vous êtes sur `main` et vous voulez récupérer le travail de `feature-login` :

```bash
git merge feature-login
```
*Note : S'il y a des conflits (deux personnes ont modifié la même ligne), Git vous demandera de les résoudre manuellement.*

---

## 4. Le "Panic Button" (Annuler et Réparer)

Tout le monde fait des erreurs. Voici comment les réparer.

### Étape 4.1 : Annuler des modifications non validées
Vous avez modifié un fichier et tout cassé, mais vous n'avez pas encore fait `git add`. Pour revenir à la version précédente :

```bash
# Remet le fichier fichier.txt comme il était au dernier commit
git restore fichier.txt
```

### Étape 4.2 : Modifier le dernier commit (Oups !)
Vous avez fait un commit mais vous avez oublié un fichier ou fait une faute de frappe dans le message ?

```bash
# Ajoutez le fichier oublié
git add fichier_oublie.txt

# Modifiez le commit précédent sans en créer un nouveau
git commit --amend -m "Nouveau message corrigé"
```

### Étape 4.3 : Mettre de côté temporairement (Stash)
Vous êtes en plein travail, c'est le bazar, mais vous devez changer de branche en urgence pour fixer un bug. Ne committez pas du code cassé !

```bash
# 1. Mettre le travail de côté (dans un presse-papier magique)
git stash

# 2. Faites vos autres tâches...

# 3. Récupérer votre travail quand vous revenez
git stash pop
```

---

## 5. Ignorer des Fichiers (.gitignore)

C'est une erreur classique : envoyer des fichiers de configuration avec des mots de passe ou des fichiers temporaires (`.log`, `.tmp`, `node_modules`) sur GitHub.

### Étape 5.1 : Créer la règle
Créez un fichier nommé `.gitignore` à la racine du projet :

```bash
nano .gitignore
```

### Étape 5.2 : Exemple de contenu
Ajoutez-y les fichiers ou dossiers à bannir :

```text
# Ignorer tous les fichiers logs
*.log

# Ignorer le dossier de dépendances
node_modules/

# Ignorer les fichiers systèmes Mac/Windows
.DS_Store
Thumbs.db

# Ignorer les clés privées (CRITIQUE !)
*.pem
*.key
.env
```

---
*Guide réalisé par Paulo Rosa.*
