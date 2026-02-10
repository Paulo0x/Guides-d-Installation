# ⚙️ Systemd : Création de Services & Daemons

Systemd est le système d'initialisation standard de Linux (PID 1). Il est responsable du démarrage de tous les processus au boot. Savoir créer ses propres fichiers "Unit" (.service) est la compétence qui distingue un bricoleur (qui utilise `nohup` ou `screen`) d'un professionnel qui déploie des services robustes et résilients.

---

## 1. Prérequis

* **Système** : Une machine Linux (Debian/Ubuntu/CentOS).
* **Compétence** : Savoir éditer un fichier texte en root.
* **Privilèges** : Accès `root` ou `sudo`.

---

## 2. Préparation du Programme

Pour l'exemple, nous n'allons pas installer un logiciel complexe, mais créer un petit script qui simule un service (il écrit l'heure dans un fichier toutes les 10 secondes).

### Étape 2.1 : Création du script

Créez le fichier exécutable dans un dossier approprié :

```bash
sudo nano /usr/local/bin/mon-service-test.sh
```

Copiez ce contenu :

```bash
#!/bin/bash
while true
do
  echo "Le service est en vie : $(date)" >> /tmp/mon-service.log
  sleep 10
done
```

### Étape 2.2 : Permissions

Rendez le script exécutable (obligatoire pour que Systemd puisse le lancer) :

```bash
sudo chmod +x /usr/local/bin/mon-service-test.sh
```

---

## 3. Configuration (Création de l'Unité)

C'est ici que nous transformons ce simple script en un véritable démon système.

### Étape 3.1 : Le fichier .service

Les services administrateurs se placent dans `/etc/systemd/system/`.

```bash
sudo nano /etc/systemd/system/mon-super-service.service
```

### Étape 3.2 : Définition du Service

Copiez ce contenu. Observez bien la directive `Restart=always` : c'est elle qui assure la résilience.

```ini
[Unit]
Description=Mon Super Service de Test (Portfolio)
After=network.target

[Service]
# Le type 'simple' est le plus courant pour un script qui tourne en boucle
Type=simple

# L'utilisateur qui lance le service (sécurité : évitez root si possible)
User=root

# La commande exacte à lancer
ExecStart=/usr/local/bin/mon-service-test.sh

# Politique de redémarrage : si le script plante, Systemd le relance
Restart=always
# Attendre 3 secondes avant de relancer
RestartSec=3

[Install]
# Indique que ce service doit se lancer quand le système est multi-utilisateurs (boot normal)
WantedBy=multi-user.target
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

---

## 4. Activation et Démarrage

Systemd doit recharger sa configuration pour voir le nouveau fichier.

### Étape 4.1 : Prise en compte

```bash
# Recharger la liste des services pour inclure le nouveau fichier
sudo systemctl daemon-reload
```

### Étape 4.2 : Démarrage et Activation au boot

```bash
# Démarrer le service maintenant
sudo systemctl start mon-super-service.service

# L'activer pour qu'il se lance automatiquement au prochain redémarrage
sudo systemctl enable mon-super-service.service
```
* **Succès** : La commande `enable` doit créer un lien symbolique ("Created symlink...").

---

## 5. Vérification & Debugging

Comment savoir si votre service tourne ou s'il a planté ?

### Étape 5.1 : Statut du service

```bash
sudo systemctl status mon-super-service.service
```
* **Succès** : Vous devez voir un point vert `● active (running)`.

### Étape 5.2 : Simulation de panne (Crash Test)

Tuons le processus manuellement pour voir si Systemd fait son travail de "chien de garde".

1.  Trouvez le PID (numéro de processus) dans la commande status précédente (Main PID).
2.  Tuez-le violemment :
    ```bash
    sudo kill -9 NUMERO_DU_PID
    ```
3.  Attendez 5 secondes et refaites un `status`.
4.  **Magie** : Le service est de nouveau `active (running)`, mais avec un nouveau PID. Systemd l'a ressuscité automatiquement.

### Étape 5.3 : Consulter les logs (Journalctl)

Systemd capture automatiquement tout ce que votre script affiche (stdout/stderr). Plus besoin de gérer des fichiers logs compliqués.

```bash
# Voir les logs du service en temps réel
sudo journalctl -u mon-super-service.service -f
```

---
*Guide réalisé par Paulo Rosa.*
