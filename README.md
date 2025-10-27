# Blue-Green Deployment – Stage 2 DevOps Task 

This repository contains the **Part A implementation** of the Stage 2 DevOps challenge.  
The goal of this task is to set up a **blue-green deployment** architecture using Docker and Nginx, allowing seamless switching between two running application versions (`blue` and `green`).

---

## 🧩 Project Overview

**Blue-green deployment** is a release strategy that reduces downtime and risk by running two identical production environments—**Blue** and **Green**.  
At any time, only one environment is live and serving traffic, while the other can be updated, tested, and prepared for the next release.

When the new version (e.g., Green) is ready and tested, the Nginx reverse proxy switches traffic from Blue → Green instantly.

---

## 🗂️ Project Structure

blue-green-deployment/
│
├── Dockerfile.nginx # Custom Dockerfile for building the Nginx proxy
├── docker-compose.yml # Defines services: Nginx, Blue app, Green app
├── nginx.conf.template # Nginx reverse proxy template
├── entrypoint.sh # Entry script for Nginx to load environment variables
├── .env.example # Sample environment file
├── README.md # Project documentation


---

## ⚙️ Prerequisites

Before running this setup, make sure you have:

- [Docker](https://docs.docker.com/get-docker/) installed  
- [Docker Compose](https://docs.docker.com/compose/) installed  
- Basic terminal knowledge

---

## 🧾 Environment Configuration

Below is a full example of how to configure and use the setup from start to finish:

```bash
# 1️⃣ Edit your .env file with your preferred active pool and image versions:

# App images
BLUE_IMAGE=yimikaade/wonderful:devops-stage-two
GREEN_IMAGE=yimikaade/wonderful:devops-stage-two

# Active pool
ACTIVE_POOL=blue

# Release IDs
RELEASE_ID_BLUE=v1.0.0
RELEASE_ID_GREEN=v1.0.1

# Ports
NGINX_PORT=8080
BLUE_PORT=8081
GREEN_PORT=8082


# 🚀 How to Run the Setup
# Step 1: Build and Start the Containers
docker-compose up --build -d
# This command:
# - Builds the Nginx proxy using the provided Dockerfile.nginx
# - Launches both Blue and Green containers
# - Exposes:
#     Blue app → localhost:8081
#     Green app → localhost:8082
#     Nginx proxy → localhost:8080


# Step 2: Check Active Pool
docker exec -it nginx_proxy printenv ACTIVE_POOL
# Expected output:
# blue


# Step 3: Access the Application
# Visit:
# http://localhost:8080/version
# You should see a JSON response from the active environment (Blue or Green).
# Example:
# {"status":"OK","message":"Application version in header"}


# Step 4: Switch Environments
# To simulate a new release, change the active pool in your .env:
# ACTIVE_POOL=green

# Then redeploy:
docker-compose up --build -d
# The proxy will now route traffic to the Green container.


# 🧠 How It Works
# - Both environments (Blue and Green) run simultaneously.
# - Nginx acts as a reverse proxy, forwarding requests to the active pool.
# - The active environment is dynamically substituted in the Nginx config using envsubst during container startup.

# Simplified flow:
# User Request → Nginx Proxy → Active App (Blue or Green)


# 🧰 Key Files Explained
# nginx.conf.template
#   Contains the Nginx upstream and proxy configuration.
#   envsubst replaces variables (e.g., $ACTIVE_POOL) before startup.

# entrypoint.sh
#   Runs environment substitution and starts Nginx:
#   envsubst '${PORT}' < /etc/nginx/nginx.conf.template > /etc/nginx/nginx.conf
#   nginx -g 'daemon off;'

# docker-compose.yml
#   Defines three services:
#   - nginx (reverse proxy)
#   - app_blue
#   - app_green
#   All connected via a shared app_network.


# 🧩 Troubleshooting
# Port already in use
# Stop any previous containers before restarting:
docker ps
docker stop <container_id>

# Blank response on /version
# Ensure your backend listens on port 3000 internally and the proxy is configured correctly.

 🧾 License
This project is for educational purposes under the Backend.im Stage 2 DevOps Challenge.
