# Guide Complet d'Installation : Helm, Prometheus et Grafana sur Kubernetes (Windows)
> Guide pour débutants de A à Z

## Introduction

Ce guide vous accompagnera pas à pas dans l'installation et la configuration d'une stack de monitoring complète pour Kubernetes. Nous allons installer :
- Helm : le gestionnaire de paquets pour Kubernetes
- Prometheus : l'outil de collecte de métriques
- Grafana : l'outil de visualisation des données

### À quoi servent ces outils ?

- **Helm** : C'est comme un "App Store" pour Kubernetes. Il simplifie l'installation d'applications complexes.
- **Prometheus** : C'est votre "collecteur de données". Il surveille votre cluster et enregistre des informations comme :
  - L'utilisation CPU et mémoire
  - L'état de santé des applications
  - Les performances du réseau
- **Grafana** : C'est votre "tableau de bord". Il affiche les données de Prometheus sous forme de graphiques faciles à comprendre.

## Prérequis

### 1. Configuration matérielle recommandée
- CPU : 2 cœurs minimum
- RAM : 8 GB minimum
- Espace disque : 20 GB minimum

### 2. Logiciels nécessaires

Installez ces outils dans l'ordre :

1. **Docker Desktop**
   - Téléchargez depuis : https://www.docker.com/products/docker-desktop
   - Installez en suivant les instructions par défaut
   - Activez Kubernetes dans les paramètres de Docker Desktop

2. **kubectl**
   - C'est l'outil de ligne de commande pour Kubernetes
   - Installation via PowerShell (administrateur) :
```powershell
# Télécharger kubectl
curl.exe -LO "https://dl.k8s.io/release/v1.28.0/bin/windows/amd64/kubectl.exe"

# Créer le dossier pour kubectl (si n'existe pas)
New-Item -Path "$env:USERPROFILE\.kube" -Type Directory -Force

# Déplacer kubectl.exe vers un dossier dans le PATH
Move-Item .\kubectl.exe -Destination "C:\Windows\System32\kubectl.exe" -Force

# Vérifier l'installation
kubectl version --client
```

3. **Chocolatey** (gestionnaire de paquets Windows)
   - Ouvrez PowerShell en administrateur
   - Exécutez :
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### 3. Vérification de l'environnement

Vérifiez que tout est bien installé :
```powershell
# Vérifier Docker
docker --version

# Vérifier Kubernetes
kubectl version

# Vérifier Chocolatey
choco --version
```

## Installation étape par étape

### Étape 1 : Installation de Helm

1. Ouvrez PowerShell en administrateur

2. Installez Helm :
```powershell
choco install kubernetes-helm
```

3. Vérifiez l'installation :
```powershell
helm version
```

4. Initialisez Helm :
```powershell
# Ajoutez les dépôts officiels
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

### Étape 2 : Installation de Prometheus

1. Ajoutez le dépôt Prometheus :
```powershell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

2. Créez un espace dédié (namespace) :
```powershell
kubectl create namespace monitoring
```

3. Installez Prometheus :
```powershell
helm install prometheus prometheus-community/prometheus --namespace monitoring
```

4. Vérifiez l'installation :
```powershell
# Vérifiez que tous les pods sont "Running"
kubectl get pods -n monitoring
```

### Étape 3 : Installation de Grafana

1. Ajoutez le dépôt Grafana :
```powershell
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

2. Créez un fichier de configuration `grafana-values.yaml` :
```yaml
adminPassword: admin123  # Changez ce mot de passe !
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.monitoring.svc.cluster.local
      access: proxy
      isDefault: true
persistence:
  enabled: true
  size: 5Gi
```

3. Installez Grafana :
```powershell
helm install grafana grafana/grafana --namespace monitoring -f grafana-values.yaml
```

## Accès aux interfaces

### Accès à Prometheus

1. Ouvrez un terminal PowerShell
2. Exécutez :
```powershell
kubectl port-forward -n monitoring svc/prometheus-server 9090:80
```
3. Ouvrez votre navigateur et accédez à : http://localhost:9090

### Accès à Grafana

1. Dans un nouveau terminal PowerShell :
```powershell
kubectl port-forward -n monitoring svc/grafana 3000:80
```
2. Ouvrez votre navigateur et accédez à : http://localhost:3000
3. Connectez-vous avec :
   - Utilisateur : admin
   - Mot de passe : admin123 (ou celui que vous avez configuré)

## Configuration de Grafana

### 1. Vérification de la source de données

1. Dans Grafana, allez dans :
   - Configuration (⚙️) > Data Sources
   - Vérifiez que Prometheus est listé
   - Cliquez sur "Test" pour vérifier la connexion

### 2. Import des tableaux de bord

1. Cliquez sur "+" > Import
2. Importez ces dashboards recommandés pour débuter :
   - ID 315 : Vue d'ensemble du cluster
   - ID 7249 : Métriques détaillées
   - ID 8588 : Métriques des applications

## Utilisation basique

### Prometheus
- Utilisez l'onglet "Graph" pour explorer les métriques
- Exemples de métriques utiles :
  - `container_memory_usage_bytes` : utilisation mémoire
  - `container_cpu_usage_seconds_total` : utilisation CPU
  - `kube_pod_status_phase` : état des pods

### Grafana
- Explorez les dashboards importés
- Utilisez la fonction de recherche pour trouver des métriques
- Ajustez les périodes de temps en haut à droite

## Dépannage

### Problèmes courants et solutions

1. **Les pods ne démarrent pas**
```powershell
# Vérifiez l'état détaillé
kubectl describe pod <nom-du-pod> -n monitoring
# Vérifiez les logs
kubectl logs <nom-du-pod> -n monitoring
```

2. **Grafana ne se connecte pas à Prometheus**
- Vérifiez l'URL dans la configuration de la source de données
- Assurez-vous que Prometheus est en cours d'exécution

3. **Erreur "port already in use"**
- Fermez les terminaux précédents
- Vérifiez qu'aucune autre application n'utilise ces ports

## Maintenance

### Mises à jour

1. Prometheus :
```powershell
helm upgrade prometheus prometheus-community/prometheus --namespace monitoring
```

2. Grafana :
```powershell
helm upgrade grafana grafana/grafana --namespace monitoring -f grafana-values.yaml
```

### Sauvegardes

1. Exportez régulièrement :
   - Vos dashboards personnalisés
   - Vos configurations d'alertes
   - Le fichier grafana-values.yaml

## Ressources additionnelles

- Documentation officielle Helm : https://helm.sh/docs/
- Documentation Prometheus : https://prometheus.io/docs/
- Documentation Grafana : https://grafana.com/docs/
- Communauté Kubernetes : https://kubernetes.io/fr/docs/

## Support

En cas de problème :
1. Consultez les logs des applications
2. Vérifiez la documentation officielle
3. Visitez les forums de la communauté
4. Stack Overflow avec les tags : kubernetes, helm, prometheus, grafana

# Concepts Avancés

## 1. Configuration Avancée de Prometheus

### 1.1 Personnalisation des règles d'alerte
Créez un fichier `prometheus-rules.yaml` :
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: custom-alerts
  namespace: monitoring
spec:
  groups:
  - name: node
    rules:
    - alert: HighCPUUsage
      expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        description: "CPU utilisation supérieure à 80% depuis 5 minutes"
```

### 1.2 Configuration des cibles de scraping personnalisées
```yaml
extraScrapeConfigs:
  - job_name: 'custom-endpoints'
    static_configs:
      - targets: ['custom-service:8080']
    metrics_path: '/metrics'
    scheme: 'http'
```

## 2. Configuration Avancée de Grafana

### 2.1 Authentification LDAP
Ajoutez dans `grafana-values.yaml` :
```yaml
ldap:
  enabled: true
  config: |
    [[servers]]
    host = "ldap.example.com"
    port = 389
    use_ssl = false
    bind_dn = "cn=admin,dc=example,dc=com"
    bind_password = "admin_password"
    search_filter = "(sAMAccountName=%s)"
    search_base_dns = ["dc=example,dc=com"]
```

### 2.2 Configuration des notifications
```yaml
notifiers:
  - name: 'team-slack'
    type: 'slack'
    uid: 'team-slack'
    is_default: true
    settings:
      url: 'https://hooks.slack.com/services/your-webhook-url'
```

## 3. Haute Disponibilité

### 3.1 Configuration HA de Prometheus
```yaml
prometheus:
  replicaCount: 2
  persistentVolume:
    enabled: true
    size: 50Gi
  retention: 15d
  statefulSet:
    enabled: true
```

### 3.2 Configuration HA de Grafana
```yaml
replicas: 2
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - grafana
        topologyKey: kubernetes.io/hostname
```

## 4. Métriques Personnalisées

### 4.1 Création d'un ServiceMonitor
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: custom-app-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: custom-app
  endpoints:
  - port: metrics
```

### 4.2 Configuration des Recording Rules
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: recording-rules
spec:
  groups:
  - name: custom.rules
    rules:
    - record: job:http_requests_total:rate5m
      expr: rate(http_requests_total[5m])
```

## 5. Sécurité Avancée

### 5.1 Configuration TLS
```yaml
tls:
  enabled: true
  cert:
    secretName: prometheus-tls
  ca:
    configMap: prometheus-ca
```

### 5.2 RBAC personnalisé
```yaml
rbac:
  create: true
  pspEnabled: true
  namespaced: true
  extraRoles:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "list", "watch"]
```

## 6. Optimisation des Performances

### 6.1 Tuning de Prometheus
```yaml
prometheus:
  resources:
    requests:
      cpu: 2
      memory: 4Gi
    limits:
      cpu: 4
      memory: 8Gi
  storage:
    tsdb:
      retention.time: 15d
      retention.size: 50GB
```

### 6.2 Configuration des scrape intervals
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  scrape_timeout: 10s
```

## 7. Intégration avec d'autres outils

### 7.1 Intégration avec Alertmanager
```yaml
alertmanager:
  config:
    global:
      resolve_timeout: 5m
    route:
      group_by: ['alertname', 'cluster', 'service']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'slack'
    receivers:
    - name: 'slack'
      slack_configs:
      - channel: '#alerts'
        api_url: 'https://hooks.slack.com/services/YOUR-WEBHOOK-URL'
```

### 7.2 Intégration avec Loki
```yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Loki
      type: loki
      url: http://loki-gateway.monitoring.svc.cluster.local
```

## 8. Bonnes Pratiques

### 8.1 Gestion des ressources
- Utilisez des limites de ressources appropriées
- Configurez le stockage persistant
- Mettez en place des politiques de rétention

### 8.2 Monitoring du monitoring
- Surveillez les performances de Prometheus
- Configurez des alertes pour le système de monitoring
- Effectuez des sauvegardes régulières

### 8.3 Documentation
- Documentez toutes les configurations personnalisées
- Maintenez un registre des dashboards
- Documentez les procédures de récupération

## 9. Troubleshooting Avancé

### 9.1 Commandes de diagnostic
```powershell
# Vérifier les métriques de Prometheus
kubectl exec -it -n monitoring prometheus-server-0 -- promtool query instant query="up"

# Analyser les performances
kubectl top pod -n monitoring

# Vérifier les logs détaillés
kubectl logs -f -n monitoring deployment/grafana -c grafana
```

### 9.2 Résolution des problèmes courants
- Problèmes de mémoire
- Latence élevée
- Perte de données
- Problèmes de connectivité

# Cas d'Utilisation Avancés et Meilleures Pratiques en Production

## 10. Architecture Multi-Cluster

### 10.1 Configuration du Thanos
```yaml
thanos:
  enabled: true
  objectStorageConfig:
    name: thanos-storage
    key: object-store.yaml
  queries:
    enabled: true
  stores:
    enabled: true
  compactor:
    enabled: true
    retentionResolutionRaw: 30d
    retentionResolution5m: 60d
    retentionResolution1h: 90d
```

### 10.2 Configuration du stockage distant
```yaml
objstore:
  type: S3
  config:
    bucket: "monitoring-metrics"
    endpoint: "s3.amazonaws.com"
    region: "eu-west-1"
    access_key: "${AWS_ACCESS_KEY}"
    secret_key: "${AWS_SECRET_KEY}"
```

## 11. Automatisation et CI/CD

### 11.1 Pipeline GitLab CI pour Helm
```yaml
deploy_monitoring:
  stage: deploy
  script:
    - helm upgrade --install prometheus prometheus-community/prometheus
      --namespace monitoring
      --values prometheus-values.yaml
      --atomic
      --timeout 10m
    - helm upgrade --install grafana grafana/grafana
      --namespace monitoring
      --values grafana-values.yaml
      --atomic
      --timeout 5m
  environment:
    name: production
```

### 11.2 ArgoCD Application
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: monitoring-stack
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/your-org/monitoring-config.git'
    path: helm
    targetRevision: HEAD
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## 12. Dashboards Avancés

### 12.1 Dashboard pour Microservices
```json
{
  "annotations": {...},
  "panels": [
    {
      "title": "Service Latency",
      "targets": [
        {
          "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{service=~\"$service\"}[5m])) by (le))",
          "legendFormat": "P95"
        }
      ]
    },
    {
      "title": "Error Rate",
      "targets": [
        {
          "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100",
          "legendFormat": "Error %"
        }
      ]
    }
  ]
}
```

### 12.2 Alertes Intelligentes
```yaml
groups:
- name: ServiceAlerts
  rules:
  - alert: AnomalyDetected
    expr: |
      abs(
        rate(http_requests_total[30m])
        - avg_over_time(rate(http_requests_total[30m])[4h:30m])
      ) > 2 * stddev_over_time(rate(http_requests_total[30m])[4h:30m])
    for: 15m
    labels:
      severity: warning
    annotations:
      description: "Anomalie détectée dans le trafic du service {{ $labels.service }}"
```

## 13. Sécurité Renforcée

### 13.1 Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: monitoring-network-policy
  namespace: monitoring
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: TCP
      port: 9090
```

### 13.2 Pod Security Context
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 65534
  fsGroup: 65534
  seccompProfile:
    type: RuntimeDefault
  capabilities:
    drop: ["ALL"]
```

## 14. Optimisation des Coûts

### 14.1 Configuration du Stockage Adaptatif
```yaml
prometheus:
  storage:
    tsdb:
      outOfOrderTimeWindow: 10m
      maxBytes: "50GB"
    retention:
      size: "40GB"
      time: 15d
  compaction:
    enabled: true
    maxBlockDuration: "2h"
```

### 14.2 Politique de Rétention Dynamique
```yaml
rules:
  - record: storage_retention
    expr: |
      avg_over_time(prometheus_tsdb_storage_blocks_bytes[7d]) > 
      0.8 * prometheus_tsdb_retention_limit_bytes
    labels:
      action: reduce_retention
```

## 15. Observabilité Avancée

### 15.1 Tracing Distribué avec Jaeger
```yaml
tracing:
  jaeger:
    enabled: true
    agent:
      enabled: true
    collector:
      enabled: true
    query:
      enabled: true
    storage:
      type: elasticsearch
```

### 15.2 Métriques Personnalisées Avancées
```yaml
customMetrics:
  - name: business_metrics
    type: counter
    help: "Métriques métier personnalisées"
    labels:
      - name: transaction_type
      - name: status
      - name: customer_segment
```

## 16. Disaster Recovery

### 16.1 Plan de Sauvegarde
```bash
# Script de sauvegarde automatique
#!/bin/bash
BACKUP_DIR="/backups/monitoring"
DATE=$(date +%Y%m%d)

# Sauvegarde des configurations
kubectl get -n monitoring configmap,secret -o yaml > "$BACKUP_DIR/configs_$DATE.yaml"

# Sauvegarde des données Prometheus
kubectl exec -n monitoring prometheus-0 -- tar czf /tmp/prometheus-data.tar.gz /prometheus

# Sauvegarde des dashboards Grafana
curl -X GET "http://grafana:3000/api/dashboards" \
  -H "Authorization: Bearer $GRAFANA_API_KEY" \
  > "$BACKUP_DIR/dashboards_$DATE.json"
```

### 16.2 Plan de Restauration
```yaml
restore:
  enabled: true
  source:
    s3:
      bucket: "monitoring-backups"
      prefix: "prometheus"
  schedule: "0 2 * * *"
  retention:
    keepLast: 7
```

## 17. Documentation et Formation

### 17.1 Wiki Technique
- Architecture détaillée
- Procédures opérationnelles
- Guides de dépannage
- Modèles d'alertes

### 17.2 Formation des Équipes
- Sessions de formation régulières
- Documentation des cas d'utilisation
- Exercices pratiques
- Procédures d'escalade

## 18. Métriques Business

### 18.1 KPIs Métier
```yaml
businessMetrics:
  - name: conversion_rate
    query: |
      sum(rate(sales_completed_total[1h]))
      /
      sum(rate(visitor_sessions_total[1h]))
  - name: revenue_per_user
    query: |
      sum(transaction_amount_total)
      /
      count(distinct user_id)
```

### 18.2 Dashboards Exécutifs
- Vue d'ensemble des KPIs
- Tendances business
- Analyses prédictives
- Rapports automatisés

---
© 2024 Niaina Nomenjanahary / Niainar's Dev