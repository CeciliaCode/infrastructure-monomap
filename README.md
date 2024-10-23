# MonoMap Infrastructure Deployment

This repository provides the Terraform configuration and the necessary Docker setup to deploy the MonoMap API on a cloud-based virtual machine. The setup includes an automated SSL certificate via Let's Encrypt, a reverse proxy using Nginx, the MonoMap API, and MongoDB as the database.

![Infrastructure](https://github.com/user-attachments/assets/4867082d-a38d-4f1a-bcb6-18eef948f178)

## Table of Contents

- [Features](#features)
- [Used Technologies](#used-technologies)
- [Infrastructure Overview](#infrastructure-overview)
- [Prerequisites](#prerequisites)
- [Setup and Deployment](#setup-and-deployment)
  - [Step 1: Clone the Repository](#step-1-clone-the-repository)
  - [Step 2: Configure Terraform Variables and GitHub Secrets](#step-2-configure-terraform-variables-and-github-secrets)
  - [Step 3: Deploy Infrastructure](#step-3-deploy-infrastructure)
  - [Step 4: Accessing the VM and Monitoring Logs](#step-4-accessing-the-vm-and-monitoring-logs)
  - [Step 5: Destroy Infrastructure](#step-5-destroy-infrastructure)
- [Docker Compose Services](#docker-compose-services)
- [CI/CD Pipeline](#ci-cd-pipeline)
- [Terraform State Management](#terraform-state-management)
  - [Step 1: Creating the Azure Storage Account for Terraform State](#step-1-creating-the-azure-storage-account-for-terraform-state)
  - [Step 2: Configuring the Terraform Backend](#step-2-configuring-the-terraform-backend)
- [Using NO-IP as a DNS Provider](#using-no-ip-as-a-dns-provider)
- [API Endpoints](#api-endpoints)
- [Appendix](#appendix)
  - [Generating SSL Certificates with Let's Encrypt](#generating-ssl-certificates-with-lets-encrypt)

---

## Features

- **Cloud-Based Deployment**: Provision a cloud-based virtual machine with Docker pre-installed.
- **Automated Docker Setup**: Run multiple containers including a reverse proxy, SSL certificate management, MonoMap API, and MongoDB.
- **SSL Certification**: Automatically secure the API using Let's Encrypt for SSL certificates.
- **GitHub Actions Integration**: CI/CD pipeline automates infrastructure deployment and destruction.
- **Remote State Management**: Terraform state is securely stored in an Azure Storage container.

## Used Technologies

- **Terraform**: Infrastructure as Code (IaC) for managing cloud resources.
- **Docker**: Containerization to run the MonoMap API, Nginx, Let's Encrypt, and MongoDB.
- **Nginx**: Reverse proxy for handling traffic and SSL termination.
- **Let's Encrypt**: Automatic SSL certificate generation and renewal.
- **MongoDB**: Database for storing API data.
- **GitHub Actions**: Automate deployments and destructions via CI/CD workflows.
- **Azure Storage**: Remote storage for Terraform state.
- **NO-IP**: Free dynamic DNS provider to map the virtual machine’s IP to a domain.

## Infrastructure Overview

This infrastructure deploys a virtual machine in the cloud with Docker installed. Upon creation, it automatically runs a `docker-compose.yml` file, which contains the following services:

1. **Nginx**: A reverse proxy that routes traffic to the API.
2. **Let's Encrypt**: Automatically manages SSL certificates.
3. **MonoMap API**: The API for managing Monkeypox cases.
4. **MongoDB**: A database for storing case data.

The deployment is automated using GitHub Actions, and the state of Terraform is stored remotely in Azure Storage.

**Here is an explanation of the file structure within this project, detailing the purpose of each folder and file.**

```
.github/
└── workflows/
    ├── deploy-dev.yml
    ├── destroy-dev.yml
env/
└── dev/
    ├── containers/
    │   ├── docker-compose.yml
    ├── scripts/
    │   ├── docker-install.tpl
    ├── main.tf
    ├── providers.tf
    ├── variables.tf
modules/
└── vm/
    ├── scripts/
    │   ├── docker-install.tpl
    ├── main.tf
    ├── outputs.tf
    ├── providers.tf
    ├── variables.tf
LICENSE
README.md
.gitignore
```

#### **`.github/workflows/`**
- **deploy-dev.yml**: Contains the GitHub Actions workflow for deploying the development environment. It automates the process of provisioning infrastructure, configuring the environment, and applying the Terraform scripts.
- **destroy-dev.yml**: This workflow is responsible for tearing down or destroying the development infrastructure when needed. It is triggered manually and helps ensure clean resource management.

#### **`env/dev/`**
- **containers/**: 
  - **docker-compose.yml**: Defines the services that will be run inside Docker containers, including the MonoMap API, MongoDB, Nginx, and Let's Encrypt for SSL certificates. This file is critical for orchestrating these containers.
- **scripts/**:
  - **docker-install.tpl**: This template file includes the script for installing Docker on the virtual machine (VM) during deployment. It ensures that Docker is available and ready for running containers after the VM is provisioned.
- **main.tf**: The main Terraform configuration file for the development environment. It outlines the resources to be provisioned, such as virtual machines, networking, and storage.
- **providers.tf**: Specifies the cloud provider (Azure in this case) and necessary configurations for interacting with the cloud provider's APIs.
- **variables.tf**: Contains variable definitions used in the Terraform files, allowing for reusable and customizable infrastructure configurations.

#### **`modules/vm/`**
- **scripts/**:
  - **docker-install.tpl**: Similar to the `env/dev/scripts/` file, this is another template for the Docker installation script that will be executed on the virtual machine.
- **main.tf**: The Terraform file that defines the virtual machine (VM) itself. This includes the configuration for provisioning the VM, setting up network interfaces, and installing necessary packages (e.g., Docker).
- **outputs.tf**: Specifies the output values from the VM provisioning process, such as IP addresses or connection details, which can be referenced by other Terraform configurations or scripts.
- **providers.tf**: Defines the cloud provider for the VM, typically mirroring the configuration found in the `env/dev` folder.
- **variables.tf**: Contains variables specific to the VM module, making it easier to reuse and customize the VM provisioning across different environments.

#### **`LICENSE`**
- Contains the legal information about the software’s usage rights and permissions.

#### **`README.md`**
- The documentation file explaining the project setup, usage, deployment, and other critical information for developers or users.

#### **`.gitignore`**
- Lists files and directories that should be ignored by Git during version control. This typically includes sensitive files (like secrets) or files generated during the build or deployment process.

---

## Prerequisites

- **Terraform**: Install [Terraform](https://www.terraform.io/downloads.html) to manage infrastructure.
- **Azure Subscription**: Required for provisioning cloud resources.
- **Docker**: Install [Docker](https://www.docker.com/) if developing locally.
- **GitHub Account**: To manage the repository and GitHub Actions.
- **Postman**: For testing API endpoints.
- **NO-IP Account**: To manage DNS mapping between the VM's public IP and a domain name.
  
## Setup and Deployment

### Step 1: Clone the Repository

```bash
git clone https://github.com/CeciliaCode/infrastructure-monomap.git
cd infrastructure-monomap
```

### Step 2: Configure Terraform Variables and GitHub Secrets

The project uses **Terraform variables** and **GitHub Secrets** to securely manage the configuration for deployment. These variables include secrets like MongoDB credentials, email settings, and Azure credentials.

#### Terraform Variables
In the Terraform files (terraform.tfvars), you will find predefined variables that you need to configure. These variables are passed into Terraform either via the command line or through your GitHub Secrets.

#### GitHub Secrets
In GitHub Actions, you'll need to set up secrets for sensitive information such as API keys and passwords. To create GitHub Secrets:

1. Go to your GitHub repository.
2. Navigate to **Settings > Secrets and variables > Actions**.
3. Click **New repository secret** and add the following secrets (make sure to keep the TF_VAR prefix for them to be properly validated by Terraform) with their respective values:

| Secret Name                          | Description                                        |
|---------------------------------------|----------------------------------------------------|
| `ARM_CLIENT_ID`                       | Azure Client ID used for authentication.           |
| `ARM_CLIENT_SECRET`                   | Azure Client Secret for authentication.            |
| `ARM_SUBSCRIPTION_ID`                 | Azure Subscription ID to deploy resources.         |
| `ARM_TENANT_ID`                       | Azure Tenant ID for the subscription.              |
| `SSH_PRIVATE_KEY`                     | Private SSH key to access the virtual machine.     |
| `SSH_PUBLIC_KEY`                      | Public SSH key for VM configuration.               |
| `TF_VAR_ADMIN_USERNAME`               | The username for the VM's admin user.              |
| `TF_VAR_DOMAIN`                       | The domain to be used with NO-IP for API access.   |
| `TF_VAR_ENVIRONMENT`                  | The environment for deployment (e.g., dev, prod).  |
| `TF_VAR_IP_NAME`                      | The name of the IP resource in Azure.              |
| `TF_VAR_LOCATION`                     | The location for Azure resource deployment.        |
| `TF_VAR_MAIL_SECRET_KEY`              | Secret key for the email service (e.g., Gmail).    |
| `TF_VAR_MAIL_SERVICE`                 | The email service provider (e.g., Gmail).          |
| `TF_VAR_MAIL_USER`                    | The email address used for notifications.          |
| `TF_VAR_MAPBOX_ACCESS_TOKEN`          | Access token for Mapbox API (for geolocation).     |
| `TF_VAR_MONGO_DB`                     | The MongoDB database name.                         |
| `TF_VAR_MONGO_INITDB_ROOT_PASSWORD`   | Root password for MongoDB initialization.          |
| `TF_VAR_MONGO_INITDB_ROOT_USERNAME`   | Root username for MongoDB initialization.          |
| `TF_VAR_MONGO_URL`                    | MongoDB connection string for local development.   |
| `TF_VAR_MONGO_URL_DOCKER`             | MongoDB connection string when running in Docker.  |
| `TF_VAR_NIC_NAME`                     | The name of the network interface for the VM.      |
| `TF_VAR_PORT`                         | The port on which the MonoMap API will run.        |
| `TF_VAR_RESOURCE_GROUP`               | The name of the resource group in Azure.           |
| `TF_VAR_SECURITY_GROUP_NAME`          | The name of the security group in Azure.           |
| `TF_VAR_SERVER_NAME`                  | The name of the virtual machine or server.         |
| `TF_VAR_SSH_KEY_PATH`                 | The file path to the SSH key.                      |
| `TF_VAR_SUBNET_NAME`                  | The subnet name for networking.                    |
| `TF_VAR_VNET_NAME`                    | The virtual network name for the infrastructure.   |

Once these secrets are set up, the GitHub Actions pipeline will automatically reference them during the deployment process.

### Step 3: Deploy Infrastructure

To deploy the infrastructure, push changes to the `main` branch or run the GitHub Action manually. The GitHub Action will provision a virtual machine, configure Docker, and run the `docker-compose.yml`.

1. **Push to main**:
    ```bash
    git push origin main
    ```

2. Alternatively, trigger the workflow manually from the GitHub Actions tab.

### Step 4: Accessing the VM and Monitoring Logs

After the VM is deployed and running, you can connect to it via SSH to monitor containers and view logs.

1. **Connect to the VM** via SSH:
    ```bash
    ssh -i path/to/private_key username@your_vm_ip
    ```

2. **Monitor Docker containers using `ctop`**:
    Once inside the VM, you can use `ctop` (an interactive container monitoring tool) to view logs and monitor the status of the running containers.

    Install and use `ctop`:
    ```bash
    sudo apt-get install ctop
    sudo ctop
    ```

    This will give you a real-time view of all the containers running on the VM, including resource usage and logs for each container.

3. **Access Error Logs**: To view logs directly from specific containers, use:
    ```bash
    docker logs <container_name>
    ```

### Step 5: Destroy Infrastructure

To manually destroy the infrastructure, use the GitHub Action `workflow_dispatch`. This will terminate the cloud resources and clean up the deployment.

---

## Docker Compose Services

The `docker-compose.yml` file defines the following services:

- **Nginx**:

 Routes traffic to the MonoMap API and handles SSL termination.
- **Let's Encrypt**: Manages SSL certificates for secure API communication.
- **MonoMap API**: The core API service for managing Monkeypox cases.
- **MongoDB**: Stores data related to the API.

### Docker Compose File

```yaml
version: '3'
services:
  nginx:
    image: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app
      - db
  app:
    image: yourdockerhubusername/monomap:latest
    environment:
      - MONGO_URL=${MONGO_URL_DOCKER}
    ports:
      - "3000:3000"
  db:
    image: mongo
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
    ports:
      - "27017:27017"
```

## CI/CD Pipeline

The project uses GitHub Actions to automate infrastructure deployment and destruction. The pipeline is configured as follows:

- **deploy-dev.yml**: Automatically deploys infrastructure when changes are pushed to the `main` branch.
- **destroy-dev.yml**: Manually triggered to destroy the deployed infrastructure.

### Example Workflow: `deploy-dev.yml`

```yaml
name: Deploy Dev Infrastructure

on:
  push:
    branches:
      - main

jobs:
  terraform-plan-apply:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.9.2

      - name: Terraform Init
        run: terraform -chdir=env/dev init

      - name: Terraform Apply
        run: terraform -chdir=env/dev apply --auto-approve
```

---

## Terraform State Management

To store the Terraform state remotely and securely, you'll need to set up an Azure Storage account and container.

### Step 1: Creating the Azure Storage Account for Terraform State

Follow these steps to create the necessary storage account and container in Azure:

1. **Create a resource group**:
    ```bash
    az group create --name tfstateRG --location eastus2
    ```

2. **Create a storage account**:
    ```bash
    az storage account create --resource-group tfstateRG --name tfstateaccount --sku Standard_LRS --location eastus2 --encryption-services blob
    ```

3. **Retrieve the storage account key**:
    ```bash
    az storage account keys list --resource-group tfstateRG --account-name tfstateaccount --query "[0].value" --output tsv
    ```

4. **Create a storage container**:
    ```bash
    az storage container create --name tfstatecontainer --account-name tfstateaccount --account-key <your-account-key>
    ```

Replace `<your-account-key>` with the value obtained from step 3.

### Step 2: Configuring the Terraform Backend

Once the storage account and container are created, configure the `backend` block in your `providers.tf` file within the path `env>dev` to use the Azure Storage for state management:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name   = "tfstateRG"
    storage_account_name  = "tfstateaccount"
    container_name        = "tfstatecontainer"
    key                   = "terraform.tfstate"
  }
}
```

This configuration ensures that your Terraform state is stored remotely and securely in Azure.

---

## Using NO-IP as a DNS Provider

Since the VM will have a dynamic public IP address, it is recommended to use a DNS service like **NO-IP** to map the VM’s IP to a domain name. This will allow you to access the API easily through a custom domain, even if the IP changes.

### Steps to Set Up NO-IP:

1. Create an account on [NO-IP](https://www.noip.com/).
2. Create a free dynamic DNS hostname in NO-IP.
3. Map the VM's public IP to the created hostname.
4. Update the DNS records whenever the VM’s IP changes.
5. Use the domain in both browser and Postman for API requests.

For example:
- Access the API through your browser:
  ```
  https://yourcustomdomain.com/api/cases
  ```

- Test the API in Postman with your domain:
  ```
  https://yourcustomdomain.com/api/cases
  ```

---

## API Endpoints

### Create a New Case

- **Method**: `POST`
- **URL**: `https://yourcustomdomain.com/api/cases`
- **Body**: (raw JSON)
  
  ```json
  {
    "lat": 19.432608,
    "lng": -99.133209,
    "genre": "Male",
    "age": 25
  }
  ```

### Get All Cases

- **Method**: `GET`
- **URL**: `https://yourcustomdomain.com/api/cases`

### Get Case by ID

- **Method**: `GET`
- **URL**: `https://yourcustomdomain.com/api/cases/:id`
  - Replace `:id` with the actual case ID.

### Update a Case

- **Method**: `PUT`
- **URL**: `https://yourcustomdomain.com/api/cases/:id`
- **Body**: (raw JSON)
  
  ```json
  {
    "lat": 19.432608,
    "lng": -99.133209,
    "genre": "Female",
    "age": 30
  }
  ```

### Delete a Case

- **Method**: `DELETE`
- **URL**: `https://yourcustomdomain.com/api/cases/:id`
  - Replace `:id` with the actual case ID.

### Get Cases from the Last Week

- **Method**: `GET`
- **URL**: `https://yourcustomdomain.com/api/cases/last`

---

## Appendix

### Generating SSL Certificates with Let's Encrypt

Let's Encrypt is used to automatically generate SSL certificates for the MonoMap API. The certificates are renewed automatically and stored securely. The configuration is included in the `docker-compose.yml`.

---
