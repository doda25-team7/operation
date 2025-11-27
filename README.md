# DODA 2026 Team 7 – Operation

This repository contains the configuration required to run and operate all services developed by **DODA 2025/2026 Team 7**.

1. [Project Repositories](#project-repositories)  
2. [Project Structure](#project-structure)  
3. [Getting Started](#getting-started)  
4. [Environment Configuration](#environment-configuration)  
5. [Cleanup](#cleanup)  
6. [Comments for A1](#comments-for-a1)

---

## Project Repositories

| Repository                                                                 | Description                                                                 |
| :------------------------------------------------------------------------- | :-------------------------------------------------------------------------- |
| **[model-service]** (https://github.com/doda25-team7/model-service)         | Serves predictions from the trained SMS classification model via REST API.  |
| **[app-service]** (https://github.com/doda25-team7/app-service)             | Main backend service (Java), communicates with `model-service`.             |
| **[app-frontend]** (https://github.com/doda25-team7/app-frontend)           | Frontend application that sends requests to `app-service`.                  |
| **[lib-version]** (https://github.com/doda25-team7/lib-version)             | Minimal versioning library used by `app-service`.                           |
| **[operation]** (https://github.com/doda25-team7/operation)                 | This repository. Orchestrates all services via Docker Compose.              |

---

## Project Structure

- **`docker-compose.yml`**  
  Defines and starts the required containers using images from GitHub Container Registry (GHCR).  
  Uses environment variables (stored in `.env`) to configure image tags and ports.

- **`.env`**  
  Provides configurable variables for image names, tags, and ports.  
  The compose file automatically reads this file.

- **`README.md`**  
  Decription of the structure and opeartion of the project.

---

## Getting Started

Follow the steps below to start the entire system.

---

### 1. Prerequisites

You must have:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed  
- A **GitHub Personal Access Token** (PAT) with at least  
  - `read:packages`  
  - `repo`  

Before running Compose, authenticate to GHCR:

```bash
echo YOUR_GITHUB_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```
### 2. Start the Containers

In you terminal, navigate to the **`operation`** repository.
Then start the containers with:

```bash
docker compose up
```

To run containers in the background, add the '-d' flag

```bash
docker compose up -d
```

### 3. Access service

If the command compiles correctly, go to http://localhost:8080 to see a page with "Hello World!" and the current version of our library. Got to http://localhost:8080/sms to use the application.

## Cleanup

To stop and remove both containers and images, run:

```bash
docker compose down --rmi all
```

---

To stop and remove the containers while leaving the images intact, run:

```bash
docker compose down 
```

---

## Comments for A1

We were missing features F7 and F9

