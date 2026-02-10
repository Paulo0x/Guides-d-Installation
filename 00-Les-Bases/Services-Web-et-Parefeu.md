# 🧱 Services Web & Pare-feu : Nginx et UFW

Un serveur ne sert à rien s'il ne communique pas, mais il est vulnérable s'il communique trop. Ce guide vous apprend à déployer le serveur web le plus rapide du monde (**Nginx**) et à fermer toutes les portes sauf celles nécessaires avec **UFW** (Uncomplicated Firewall).

---

## 1. Le Pare-feu (UFW)

Avant d'installer quoi que ce soit, on verrouille. Par défaut, UFW bloque tout ce qui entre et autorise tout ce qui sort.

### 1.1 Installation et Règles de Base

**ATTENTION :** La première règle OBLIGATOIRE est d'autoriser SSH, sinon vous serez coupé de votre serveur à jamais lors de l'activation.

```bash
# 1. Installer UFW (souvent déjà là)
sudo apt install ufw -y

# 2. Règle VITALE : Laisser passer SSH (Port 22)
sudo ufw allow ssh
# Ou si vous avez changé le port (ex: 2222) : sudo ufw allow 2222/tcp

# 3. Laisser passer le Web (Ports 80 et 443)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 4. Activer le pare-feu (Confirmez avec 'y')
sudo ufw enable
```

### 1.2 Vérification

```bash
sudo ufw status verbose
```
*Vous devez voir "Status: active" et la liste des ports autorisés (Allow).*

---

## 2. Le Serveur Web (Nginx)

Nginx (prononcé "Engine-X") est un serveur web performant, souvent utilisé comme Reverse Proxy.

### 2.1 Installation

```bash
# 1. Installer le paquet
sudo apt update
sudo apt install nginx -y

# 2. Vérifier qu'il tourne
systemctl status nginx
```

### 2.2 Test immédiat

Ouvrez votre navigateur sur votre PC et tapez l'IP du serveur : `http://192.168.1.50` (ou votre IP).
* **Succès** : Vous devez voir la page "Welcome to nginx!".

---

## 3. Héberger votre propre site

La page par défaut est ennuyeuse. Remplaçons-la.

### 3.1 Localiser les fichiers

Sur Debian/Ubuntu, le site par défaut est dans `/var/www/html`.

```bash
cd /var/www/html
ls
# Vous verrez index.nginx-debian.html
```

### 3.2 Modifier la page d'accueil

Nous allons écraser le fichier par défaut avec notre propre HTML.

```bash
# Devenir propriétaire du dossier pour pouvoir écrire dedans (remplacez 'paulo' par votre user)
sudo chown -R paulo:paulo /var/www/html/

# Créer un nouvel index.html
nano /var/www/html/index.html
```

Collez ce code HTML simple :

```html
<!DOCTYPE html>
<html>
<head>
    <title>Mon Premier Serveur TSSR</title>
</head>
<body>
    <h1>Serveur Opérationnel !</h1>
    <p>Ce site est hébergé sur Nginx et protégé par UFW.</p>
    <p>Installé par Paulo.</p>
</body>
</html>
```

*(Sauvegardez avec `Ctrl+O`, quittez avec `Ctrl+X`)*.

Rafraîchissez votre navigateur. Votre site est en ligne !

---

## 4. Analyse des Logs (Dépannage)

Un client vous appelle : "Le site ne marche pas !". Le réflexe TSSR, c'est de lire les logs.

### 4.1 Logs d'accès (Qui vient ?)

```bash
# Voir les dernières connexions en temps réel
# Copiez l'IP qui s'affiche, c'est votre PC !
sudo tail -f /var/log/nginx/access.log
```
*(Faites `Ctrl+C` pour arrêter le défilement).*

### 4.2 Logs d'erreur (Pourquoi ça plante ?)

Si Nginx refuse de démarrer ou si une page affiche "500 Error", c'est ici :

```bash
sudo cat /var/log/nginx/error.log
```

---

## 5. Commandes de Service (Rappel)

Comme vu dans le module précédent, Nginx est un service géré par Systemd.

```bash
# Redémarrer (Stop + Start)
sudo systemctl restart nginx

# Recharger la configuration (Sans couper les connexions actives - Plus pro)
sudo systemctl reload nginx

# Arrêter le service
sudo systemctl stop nginx
```

---
*Guide réalisé par Paulo Rosa.*
