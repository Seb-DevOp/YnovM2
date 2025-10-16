# YnovM2

🧱 Projet Conteneurisation / Orchestration
🎯 Objectif

Utiliser Google Cloud Platform (GCP) dans une approche orientée certification Cloud Engineer, tout en construisant une infrastructure Cloud moderne, automatisée et sécurisée.

⚙️ Stack Technique

Cloud Provider : GCP

Orchestration : Kubernetes (GKE)

Infrastructure-as-Code : Terraform

CI/CD : GitHub Actions

Monitoring : Zabbix + Cloud Logging & Monitoring

🧩 Résumé du Projet — Stack GCP Complète

Ce projet vise à créer une infrastructure complète sur GCP, entièrement automatisée via Terraform, pour déployer une stack applicative de type production.

Elle comprend :

Un cluster GKE (Google Kubernetes Engine) pour exécuter les applications conteneurisées.

Un système de supervision Zabbix pour le monitoring.

Un pipeline CI/CD (GitHub Actions ou Cloud Build) pour automatiser les déploiements.

Une application Node.js basique comme démonstration.

La stack s’appuie sur :

Cloud SQL pour la base de données,

Cloud Storage et Artifact Registry pour le stockage et la gestion des images,

Cloud Logging & Monitoring pour l’observabilité,

IAM, Service Accounts et Secret Manager pour la sécurité,

Un VPC isolé avec Load Balancer HTTPS pour l’accès public.

👉 Le projet met en œuvre les piliers du Cloud Engineer GCP : Compute, Networking, IAM, Monitoring et Infrastructure-as-Code.

🪜 Étapes du Projet (de A à Z)
0️⃣ Pré-requis & Cadrage

Compte GCP avec facturation activée

Nom de domaine (facultatif, ex. app.example.com)

Repo GitHub privé avec Actions activé

Outils : Terraform ≥ 1.6, gcloud, kubectl, helm

Décisions Initiales

Environnements : dev / stg / prod

Backend Terraform : Terraform Cloud ou GCS versionné

Région : europe-west1

Type GKE : Autopilot (recommandé pour la certif)

1️⃣ Bootstrap de la Fondation GCP (Terraform)
🎯 Objectif

Créer les briques partagées et sécuriser la base.

🔧 Modules & Ressources

project : création / liaison billing

iam : rôles pour CI/CD, GKE, Cloud SQL

gcs : bucket tf-state (versioning + KMS)

VPC privé + sous-réseaux + Cloud NAT

Artifact Registry (europe-docker.pkg.dev/<project>/apps)

Cloud Storage (ex: app-assets-<env>)

Secret Manager : secrets placeholders

Livrable : infra/bootstrap
Validation : terraform plan/apply propre + VPC visible

2️⃣ Cluster Kubernetes (GKE) & Sécurité d’accès
🎯 Objectif

Cluster prêt pour la production, sécurisé et monitoré.

⚙️ Configurations Clés

GKE Autopilot (VPC privé, Shielded Nodes, Workload Identity)

Add-ons : Cloud Logging/Monitoring, HPA, PDB

Ingress : HTTP(S) Load Balancer (ManagedCertificate)

Livrable : infra/gke
Validation : kubectl get nodes OK, Workload Identity fonctionnelle

3️⃣ Base de Données Managée (Cloud SQL)
🎯 Objectif

Fournir une DB sécurisée en réseau privé.

Cloud SQL (PostgreSQL/MySQL)

Connexion via Cloud SQL Auth Proxy

Secrets dans Secret Manager

Livrable : infra/cloudsql
Validation : Pod smoke-test → SELECT 1 OK

4️⃣ Observabilité & Supervision (Zabbix + Cloud Ops)
🎯 Objectif

Monitoring complet multi-niveaux.

Cloud Monitoring : dashboards + alertes basiques

Zabbix via Helm sur GKE

UI exposée via Ingress interne (auth restreinte)

Livrable : platform/zabbix/values-<env>.yaml
Validation : alertes Slack/mails fonctionnelles

5️⃣ Pipeline CI/CD (GitHub Actions ↔ GCP)
🎯 Objectif

Automatiser build/test/deploy sans clés JSON.

Auth GitHub → GCP via OIDC

Jobs CI : lint, test, build, push image

Jobs CD : déploiement progressif (dev → stg → prod)

Livrable : .github/workflows/ci.yml et cd.yml
Validation : PR = CI, Merge = déploiement dev

# Exemple d’extrait CI
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.GCP_WIP }}
          service_account: ci-cd@${{ secrets.GCP_PROJECT }}.iam.gserviceaccount.com
      - run: |
          docker build -t europe-docker.pkg.dev/${{ secrets.GCP_PROJECT }}/apps/web:${{ github.sha }} .
          gcloud auth configure-docker europe-docker.pkg.dev -q
          docker push europe-docker.pkg.dev/${{ secrets.GCP_PROJECT }}/apps/web:${{ github.sha }}

6️⃣ Application JS Basique
🎯 Objectif

Valider la chaîne de déploiement.

Node.js (Express) avec /healthz, /version, /dbcheck

K8s : Deployment, Service, Ingress, HPA, PDB, NetworkPolicy

Livrable : apps/web/ + k8s/overlays/{dev,stg,prod}
Validation : curl https://app.dev.example.com/healthz → 200

7️⃣ Réseau & Exposition Publique
🎯 Objectif

Accès externe sécurisé.

Ingress GKE + ManagedCertificate

Cloud DNS pour le domaine

Cloud Armor (optionnel) : OWASP ruleset

Validation : HTTPS actif, redirection OK

8️⃣ Sécurité, IAM & Secrets
🎯 Objectif

Principe du moindre privilège.

SA ci-cd@ et workload-app@

Workload Identity : ksa:web-runner → gsa:workload-app

Secrets dans Secret Manager via CSI

Bonnes pratiques :

Pas de clés externes

Rotation périodique des secrets

Labels et budgets configurés

9️⃣ Coûts & Étiquetage

Budgets GCP + alertes 50/80/100%

Labels : env, app, owner, cost-center

Optimisations : Autopilot, classes Storage régionales

🔟 Tests, SLOs & Runbooks

SLO : Dispo 99.5%, p95 latence, <1% erreurs 5xx

Tests : Liveness/Readiness + e2e (k6 ou curl loop)

Runbooks : rollback, crashloop, DB indispo

1️⃣1️⃣ Documentation & Livrables Finaux

README principal (diagrammes, flux CI/CD, URLs)

Handbook certif : mapping des objectifs GCP (Compute, Networking, IAM, Monitoring, IaC)

🗂️ Arborescence du Repo
.
├── infra/
│   ├── bootstrap/
│   ├── gke/
│   ├── cloudsql/
│   ├── dns-lb/
│   └── modules/
├── platform/
│   └── zabbix/
├── apps/
│   └── web/
│       ├── src/
│       └── Dockerfile
├── k8s/
│   ├── base/
│   └── overlays/
│       ├── dev/
│       ├── stg/
│       └── prod/
└── .github/workflows/
    ├── ci.yml
    └── cd.yml

✅ Critères d’Acceptation

 Terraform apply sans erreur (bootstrap, GKE, Cloud SQL)

 CI : build & push image OK

 CD : déploiement dev → stg → prod

 App : endpoints /healthz & /dbcheck OK

 Monitoring : alertes et dashboards opérationnels

 Sécurité : Workload Identity + secrets protégés

 Coûts : budget & alertes actives

 Documentation complète & à jour

💻 Commandes Utiles
# Auth locale
gcloud auth login && gcloud config set project <PROJECT_ID>

# Backend Terraform
gsutil mb -l europe-west1 gs://<PROJECT_ID>-tf-state
gsutil versioning set on gs://<PROJECT_ID>-tf-state

# Connexion GKE
gcloud container clusters get-credentials <CLUSTER_NAME> --region europe-west1 --project <PROJECT_ID>

# Déploiement applicatif
kubectl apply -k k8s/overlays/dev
kubectl rollout status deploy/web -n dev

# Helm Zabbix
helm repo add zabbix https://zabbix.github.io/zabbix-helm
helm install zabbix zabbix/zabbix -n observability -f platform/zabbix/values-dev.yaml

🎓 Mapping Certification GCP Cloud Engineer
Domaine	Élément du Projet
Compute	GKE + autoscaling + Ingress
Networking	VPC privé, NAT, HTTPS LB, DNS
IAM	Service Accounts + Workload Identity
Monitoring	Cloud Ops + Zabbix + alerting
IaC	Terraform + GitHub Actions CI/CD
