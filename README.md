# SMS Checker - DODA 2026 Team 7 – Operation
This repository contains the orchestration code of a dummy SMS spam classifier. More information about the SMS checker can be found on [proksch/sms-checker](https://github.com/proksch/sms-checker).

The repository, organization and the accompanying releases are used to learn about DevOps practices by student group [doda25-team7](https://github.com/doda25-team7). The work was done in the context of the course [DevOps for Distribued Apps (CS4295)](https://studyguide.tudelft.nl/courses/study-guide/educations/14776) at the TU Delft. The groups organization page links to the associated repositories.


## Project Repositories


The following repositories are used in this project:
| Repository                                                                 | Description                                                                 |
| :------------------------------------------------------------------------- | :-------------------------------------------------------------------------- |
| **[model-service]** (https://github.com/doda25-team7/model-service)         | Backend service for SMS classification predictions and all other ML code. |
| **[app-service]** (https://github.com/doda25-team7/app-service)             | Frontend service communicating with `model-service`, includes `lib-version`.             |
| **[lib-version]** (https://github.com/doda25-team7/lib-version)             | Minimal dummy versioned library used by `app-service`.                           |
| **[operation]** (https://github.com/doda25-team7/operation)                 | This repository, contains all orchestration code.             |

---

## Project Structure

- **`docker-compose.yml`**  
  Defines and starts the required containers using images from GitHub Container Registry (ghcr.io).  
  Uses environment variables (stored in `.env`) to configure image tags and ports.

- **`.env`**  
  Provides configurable variables for image names, tags, and ports.  
  The compose file automatically reads this file.

- **`infrastructure`**
  Contains the orchestration code to setup a local kubernetes cluster using Vagrant and Ansible. More information, and a setup guide can be found in the infrastructure/README.md

- **`SMS-checker`**
  Defines the Helm Chart to deploy SMS-checker to a Kubernetes cluster.

- **`.ACTIVITY.MD`**
  Contains a course-related activity record to show that the coursework was divided appropriately, and weekly contributions where made.

---

## Getting Started using Docker Compose

For development purposes a Docker Compose file describing the container setup have been included. To use Docker Compose, a modern version of Docker should to be installed. Configuration of the Docker deployment is done in the .env file.

With Docker installed the containers can be started up using:
```bash
docker compose up -d
```
The status of the containers can be checked using:
```bash
docker compose ps
```
And the logs checked using the following commands:
```bash
docker compose logs app-service --follow
docker compose logs model-service --follow
```

### Accessing the services

If the Docker containers are created with the default `.env` configuration file, `app-service` should be accessable on: [localhost:8080](http://localhost:8080). Visiting the root of this page should show "Hello World!" and the current version of [lib-version](https://github.com/doda25-team7/lib-version). The SMS-checker frontend should be accessible on [http://localhost:8080/sms](http://localhost:8080/sms).

### Cleanup

To stop and remove the containers while leaving the images intact, run:
```bash
docker compose down 
```
To stop and remove the containers, and clear the images from the local cache, run:
```bash
docker compose down --rmi all
```
## Getting Stated using Kubernetes

To get started deploying SMS-checker to Kubernetes, a Kubernetes cluster first has to be deployed. The code that was used to create the Kubernetes cluster that SMS-checker uses, accompnied by a README.md can be find in the [infrastructure](infrastructure) folder in the root of this repository. SMS-checker has been verified to work on this Kubernetes cluster and [minikube](https://github.com/kubernetes/minikube), but it should also work on any other deployment.

To get started using the infrastructure configuraiton provided run the following:
```bash
cd infrastructure
vagrant up --provision
ansible-playbook --inventory .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory ./ansible/finalization.yaml 
```
More information on deploying and debugging the Kubernetes cluser can be found in infrastructure/README.md.

## Deploying with Helm

To deploy SMS-checker into a Kubernetes cluster, there is a Helm chart [SMS-checker](SMS-checker), in the root of the repository. To deploy the helm chart, using the infrastructure as described above, run the following code from the infrastructure folder:

```bash
vagrant ssh ctrl
cd /operation/SMS-checker
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm dependency update
cd /operation
helm upgrade --install sms-checker ./SMS-checker --namespace sms-checker --create-namespace
```

Because the routing of the HTTP packets is based on the host header the following should be added into the ```/etc/hosts``` (Linux/Mac) or ```%WINDIR%\system32\drivers\etc\hosts``` (Windows):
```
192.168.56.91  doda.local
192.168.56.90  grafana.local
192.168.56.90  prometheus.local
192.168.56.90  dashboard.local
```

**How to edit the hosts file:**

**Mac/Linux:**
```bash
sudo nano /etc/hosts
# Add the lines above, save with Ctrl+O, exit with Ctrl+X
```

**Windows (run as Administrator):**
```powershell
notepad C:\Windows\System32\drivers\etc\hosts
# Add the lines above and save
```

This deployment acts as a complete stack, including **Prometheus** for metrics collection and **Grafana** for visualization, accessible on ```prometheus.local``` and ```grafana.local``` accordingly. The Grafana instance has the default username of ```admin``` and password of ```prom-operator```. ```doda.local``` provides access to the main application with Istio service mesh features.

### Monitoring (Prometheus & Grafana)

- **Prometheus** scrapes application metrics exposed at `/actuator/prometheus`.
   1. **Notifications**: Sends a notification using a Discord webhook when a metric has reached a threshold.
- **Grafana** is pre-configured with two dashboards:
   1. **SMS App Metrics**: Shows request rates, prediction ratios, and latencies.
   2. **SMS System Metrics**: Shows system resource usage.

### Enabling Discord notifications

To enable alerting in Discord, add a Discord webhook URL as described on [the "Intro to Webhooks" guide created by Discord](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) as a K8S secret. To create a K8S Secret to hold this URL, run this, replacing `YOUR_DISCORD_WEBHOOK_URL` with your Discord Webhook URL:
```bash
kubectl create secret generic alertmanager-discord-webhook \
  --from-literal=webhook-url='YOUR_DISCORD_WEBHOOK_URL' \
  --namespace sms-checker
```

### To access Grafana:
1. Open [http://grafana.local](http://grafana.local).
2. Default credentials (from `kube-prometheus-stack`): `admin` / `prom-operator` (or check secret `sms-stack-grafana`).
3. Dashboards are automatically loaded under `Dashboards` -> `Browse`.

### Manual Dashboard Import
Sometimes the dashboards are not imported automatically, if you need to import dashboards manually run the following:
1. JSON files are located in `operation/SMS-checker/dashboards/`.
2. In Grafana UI, go to **Dashboards** -> **New** -> **Import**.

---

## Quick Setup & Test Guide

### 1. Start Infrastructure
```bash
cd infrastructure
vagrant up
vagrant provision ctrl --provision-with finalization
```

### 2. Deploy SMS-Checker
```bash
vagrant ssh ctrl
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm dependency update
cd /operation
helm upgrade --install sms-checker ./SMS-checker --namespace sms-checker --create-namespace
kubectl get pods  # Wait for all pods to show 2/2 Running
```

### 3. Test Canary Routing (90/10 Split)
```bash
# Run 20 requests, count stable vs canary
for i in {1..20}; do 
  curl -sI -H "Host: doda.local" http://192.168.56.91/ | grep x-version
done | sort | uniq -c
# Expected: ~18 stable, ~2 canary
```

### 4. Test Header-Based Routing
```bash
# Force canary
curl -sI -H "Host: doda.local" -H "x-version: canary" http://192.168.56.91/ | grep x-version
# Expected: x-version: canary

# Force stable
curl -sI -H "Host: doda.local" -H "x-version: stable" http://192.168.56.91/ | grep x-version
# Expected: x-version: stable
```

### 5. Test Shadow Mirroring
```bash
# Terminal 1: Watch shadow logs
kubectl logs -f deployment/sms-checker-model-service-shadow -c istio-proxy

# Terminal 2: Send request (shadow receives mirrored copy)
curl -X POST http://doda.local/sms/ -H "Content-Type: application/json" -d '{"sms": "Hello this is a test message", "guess":"spam"}'
```

### 6. Access Monitoring
- **Grafana**: http://grafana.local (admin / prom-operator)
- **App**: http://doda.local

### 7. Cleanup
```bash
helm uninstall sms-checker -n default
exit
cd infrastructure && vagrant destroy -f
```

---

## Team

DODA Team 7

Maciej Bober, Tess Hobbes, Francisco Duque de Morais Amaro, Tobias Peeters, Renyi Yang

