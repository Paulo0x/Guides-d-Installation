# 📊 Prometheus & Grafana : La Supervision Moderne

Alors que Zabbix est très puissant pour l'infrastructure classique, le duo **Prometheus + Grafana** est devenu le standard pour le monitoring des conteneurs et du Cloud. Ce guide déploie une stack complète via Docker Compose incluant :
1.  **Prometheus** : Le moteur qui stocke les données (Time Series Database).
2.  **Node Exporter** : L'agent qui récupère les métriques du serveur hôte (CPU, RAM, Disque).
3.  **Grafana** : L'interface visuelle pour afficher les graphiques.

---

## 1. Prérequis

* **Système** : Une machine avec Docker et Docker Compose installés (voir guide *02-Conteneurs/Docker-Install.md*).
* **Ports** : Les ports `3000` (Grafana) et `9090` (Prometheus) doivent être libres.

---

## 2. Préparation de l'environnement

Nous allons créer un dossier dédié pour stocker la configuration et les données persistantes.

### Étape 2.1 : Structure des dossiers

```bash
mkdir -p ~/monitoring/prometheus
cd ~/monitoring
```

### Étape 2.2 : Configuration de Prometheus (prometheus.yml)

Prometheus a besoin d'un fichier de configuration pour savoir *qui* surveiller. Créez ce fichier :

```bash
nano prometheus/prometheus.yml
```

Copiez ce contenu YAML :

```yaml
global:
  scrape_interval: 15s # Récupérer les métriques toutes les 15s

scrape_configs:
  # Job 1 : Prometheus se surveille lui-même
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Job 2 : Surveillance du serveur hôte via Node Exporter
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

*(Sauvegardez avec `Ctrl+O`, puis quittez avec `Ctrl+X`)*.

---

## 3. Création de la Stack (Docker Compose)

Nous allons définir nos 3 services dans un seul fichier pour les lancer ensemble.

```bash
nano docker-compose.yml
```

Copiez ce contenu complet (attention à l'alignement) :

```yaml
version: '3.8'

services:
  # 1. Le Cerveau (Base de données temporelle)
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
    networks:
      - monitoring

  # 2. L'Agent (Collecteur de métriques système)
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - 9100:9100
    networks:
      - monitoring

  # 3. L'Interface Visuelle
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - 3000:3000
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin  # Changez ce mot de passe à la première connexion !
    networks:
      - monitoring

volumes:
  prometheus_data:
  grafana_data:

networks:
  monitoring:
    driver: bridge
```

*(Sauvegardez et quittez)*.

---

## 4. Démarrage et Vérification

### Étape 4.1 : Lancement

```bash
docker compose up -d
```
*Docker va télécharger les images et lancer les conteneurs en arrière-plan.*

### Étape 4.2 : Vérification des conteneurs

```bash
docker compose ps
```
* **Succès** : Les 3 conteneurs (`prometheus`, `node-exporter`, `grafana`) doivent être en statut `Up`.

---

## 5. Configuration de Grafana

C'est là que la magie opère.

1.  Ouvrez votre navigateur : `http://IP_DU_SERVEUR:3000`.
2.  Connectez-vous (User: `admin` / Pass: `admin`). Grafana vous demandera de changer le mot de passe.
3.  **Ajouter la source de données** :
    * Allez dans **Connections** (ou Configuration) > **Data Sources**.
    * Cliquez sur **+ Add new data source**.
    * Sélectionnez **Prometheus**.
    * Dans le champ URL, mettez : `http://prometheus:9090` (C'est le nom du conteneur Docker, pas localhost).
    * Cliquez sur **Save & Test** (Doit afficher "Successfully queried the Prometheus API").

4.  **Importer un Dashboard** :
    * Pas besoin de tout créer à la main !
    * Cliquez sur le **+** en haut à droite > **Import dashboard**.
    * Dans le champ "Import via grafana.com", tapez l'ID : `1860` (Node Exporter Full).
    * Cliquez sur **Load**.
    * Sélectionnez votre source Prometheus en bas.
    * Cliquez sur **Import**.

* **Succès** : Vous avez maintenant un tableau de bord complet "Node Exporter Full" avec les graphiques CPU, RAM, Disque et Réseau de votre serveur en temps réel.

---
*Guide réalisé par Paulo Rosa.*
