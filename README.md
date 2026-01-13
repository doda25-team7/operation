# SMS Checker - DODA 2026 Team 7 – Operation
This repository contains the orchestration code of a dummy SMS spam classifier. More information about the SMS checker can be found on [proksch/sms-checker](https://github.com/proksch/sms-checker). 

The repository, organization and the accompanying releases are used to learn about DevOps practices by student group [doda25-team7](https://github.com/doda25-team7). The work was done in the context of the course [DevOps for Distribued Apps (CS4295)](https://studyguide.tudelft.nl/courses/study-guide/educations/14776) at the TU Delft. The groups organization page links to the associated repositories. 

This repository contains the configuration required to run and operate all services developed,

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
  Contains a course-related activity record to show that the coursework was divided appropriately. 

---

## Getting Started using Docker Compose 

For development purposes a Docker Compose file describing the container setup have been included. To use Docker Compose, a modern version of Docker should to be installed. To configure the Docker deployment, change the config in the .env file. 

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

To get started deploying SMS-checker to Kubernetes, a Kubernetes cluster first has to be deployed. The code that was used to create the Kubernetes cluster that SMS-checker uses, accompnied by a README.md can be find in the [infrastructure](infrastructure) folder in the root of this repository. SMS-checker has been verified to work on this Kubernetes cluster, but it should also work on any other deployment.

To get started using the infrastructure configuraiton provided run the following: 
```bash
cd infrastructure
vagrant up
ansible-playbook --inventory .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory ./ansible/finalization.yaml 
```
More information on deploying the Kubernetes cluser can be found in infrastructure/README.md.

## Deploying with Helm

To deploy SMS-checker into a Kubernetes cluster, there is a Helm chart [SMS-checker](SMS-checker), in the root of the repository. To deploy the helm chart, using the infrastructure as described above, run the following code from the infrastructure folder:
```bash
vagrant ssh ctrl
cd /operation
helm upgrade --install SMS-checker SMS-checker --namespace prod-SMS-checker --create-namespace
```

The deployment acts as a complete stack, including **Prometheus** for metrics collection and **Grafana** for visualization.

### Monitoring (Prometheus & Grafana)

- **Prometheus** scrapes application metrics exposed at `/actuator/prometheus`.
    1. **Notifications**: Sends a notification using a Discord webhook when a metric has reached a threshold. 
- **Grafana** is pre-configured with two dashboards:
    1. **SMS App Metrics**: Shows request rates, prediction ratios, and latencies.
    2. **SMS System Metrics**: Shows system resource usage.

To enable prometheus based monitoring:
```bash
vagrant ssh ctrl
helm repo add prom-repo https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade myprom prom-repo/kube-prometheus-stack --namespace monitoring --create-namespace
```

### To access Prometheus

To access the Prometheus UI run the following:
```bash
kubectl port-forward -n monitoring svc/myprom-kube-prometheus-sta-prometheus 9090:9090

curl http://localhost:8080/metrics
```

### Enabling Discord notifications

To enable alerting in Discord, first enable to the alertmanager:
```bash
kubectl label namespace sms alertmanager-config=enabled --overwrite
helm upgrade myprom prom-repo/kube-prometheus-stack \
  --namespace monitoring \
  --set alertmanager.alertmanagerSpec.alertmanagerConfigNamespaceSelector.matchLabels.alertmanager-config=enabled
```
Then create a webhook URL as described on [the "Intro to Webhooks" guide created by Discord](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks). To create a Kubernetes Secret to hold this URL, run this, replacing `YOUR_DISCORD_WEBHOOK_URL` with your Discord Webhook URL:
```bash
kubectl create secret generic alertmanager-discord-webhook \
  --from-literal=webhook-url='YOUR_DISCORD_WEBHOOK_URL' \
  --namespace prod-SMS-checker
```

### To access Grafana:
1. Forward the port:
   ```bash
   kubectl port-forward svc/sms-stack-grafana 8080:80
   ```
2. Open [http://localhost:8080](http://localhost:8080).
3. Default credentials (from `kube-prometheus-stack`): `admin` / `prom-operator` (or check secret `sms-stack-grafana`).
4. Dashboards are automatically loaded under `Dashboards` -> `Browse`.

### Manual Dashboard Import
Sometimes the dashboards are not imported automatically, if you need to import dashboards manually run the following:
1. JSON files are located in `operation/SMS-checker/dashboards/`.
2. In Grafana UI, go to **Dashboards** -> **New** -> **Import**.
3. Upload `app-metrics.json` or `system-metrics.json`.