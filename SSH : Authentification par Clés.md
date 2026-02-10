# 🔑 SSH : Authentification par Clés & Sécurisation

L'authentification par mot de passe est vulnérable aux attaques par force brute et au vol d'identifiants. La norme professionnelle consiste à utiliser des paires de clés cryptographiques (Publique/Privée). Ce guide explique comment générer ces clés et configurer le serveur pour refuser toute connexion par mot de passe classique.

---

## 1. Prérequis

* **Serveur** : Une machine Linux (Debian/Ubuntu) accessible en SSH.
* **Client** : Votre poste de travail (Windows avec PowerShell, ou Linux/Mac).
* **Accès** : Avoir les droits `root` ou `sudo` sur le serveur.

---

## 2. Génération des Clés (Côté Client)

Cette étape se fait sur **votre ordinateur personnel** (pas sur le serveur). Nous allons utiliser l'algorithme *Ed25519*, plus rapide et sécurisé que le vieux RSA.

### Étape 2.1 : Création de la paire
Ouvrez votre terminal (PowerShell ou Bash) et tapez :

```bash
ssh-keygen -t ed25519 -C "admin@portfolio"
```

1.  Appuyez sur **Entrée** pour valider l'emplacement par défaut.
2.  **Passphrase** : Il est fortement recommandé de mettre une phrase de passe pour chiffrer la clé privée (c'est une sécurité supplémentaire en cas de vol de votre PC).

### Étape 2.2 : Transfert de la clé publique
Il faut maintenant envoyer votre "cadenas" (clé publique) sur le serveur.

**Option A : Depuis Linux / Mac / WSL**
Remplacez `user` et `IP_SERVEUR` par vos infos :

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@IP_SERVEUR
```

**Option B : Depuis Windows (PowerShell)**
Si vous n'avez pas `ssh-copy-id`, utilisez cette commande :

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh user@IP_SERVEUR "cat >> .ssh/authorized_keys"
```

---

## 3. Configuration du Serveur

Maintenant que la clé est en place, nous allons dire au serveur SSH de **refuser** les mots de passe.

### Étape 3.1 : Test de connexion (Critique)
**Avant de modifier quoi que ce soit**, essayez de vous connecter :

```bash
ssh user@IP_SERVEUR
```
*Si le serveur vous connecte sans demander le mot de passe de l'utilisateur (mais peut-être celui de la clé), c'est gagné. Passez à la suite.*

### Étape 3.2 : Désactivation des mots de passe
Connecté sur le serveur, éditez la configuration du démon SSH :

```bash
sudo nano /etc/ssh/sshd_config
```

Trouvez et modifiez (ou ajoutez) les lignes suivantes pour qu'elles correspondent exactement à ceci :

```ini
# Désactive l'authentification par mot de passe
PasswordAuthentication no

# Désactive le PAM (souvent nécessaire pour bloquer totalement le password)
ChallengeResponseAuthentication no
UsePAM no

# Interdit le login root (bonne pratique, utilisez sudo avec votre user)
PermitRootLogin no

# Indique où chercher les clés (par défaut)
PubkeyAuthentication yes
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

### Étape 3.3 : Validation et Redémarrage
Vérifiez qu'il n'y a pas d'erreur de syntaxe dans le fichier :

```bash
sudo sshd -t
```
*(Si rien ne s'affiche, c'est bon).*

Redémarrez le service SSH :

```bash
sudo systemctl restart ssh
```

---

## 4. Vérification

Il est temps de vérifier que la porte est bien fermée aux intrus.

### Étape 4.1 : Test de la clé
Depuis votre poste, connectez-vous normalement :
`ssh user@IP_SERVEUR` -> **Succès** (Vous entrez).

### Étape 4.2 : Test du refus de mot de passe
Pour être sûr, forcez SSH à ignorer votre clé et à tenter le mot de passe (ce qui doit échouer) :

```bash
ssh -o PubkeyAuthentication=no user@IP_SERVEUR
```

* **Résultat attendu** :
    > *Permission denied (publickey).*
    ou
    > *user@ip: Permission denied (publickey).*

Si vous voyez ce message, votre serveur est désormais hermétique aux attaques par dictionnaire et brute-force classiques sur les mots de passe.

---
*Guide réalisé par Paulo Rosa.*
