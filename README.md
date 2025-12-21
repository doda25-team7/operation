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

For development purposes a Docker Compose file describing the container setup have been included. To use Docker Compose, a modern version of Docker should to be installed. With Docker installed the containers can be started up using:
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

If the containers are healthy, http://localhost:8080 should show "Hello World!" and the current version of [lib-version](https://github.com/doda25-team7/lib-version). The SMS-checker frontend should be accessible on [http://localhost:8080/sms](http://localhost:8080/sms).

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

First the Kubernetes cluser should be deployed. 
More information on deploying the Kubernetes cluser can be found in infrastructure/README.md.

TODO: continue this readme section.