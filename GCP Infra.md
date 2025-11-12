# Google Cloud Platform (GCP) Infrastructure Documentation

## 1. Introduction

This document provides a brief overview and setup guide for using **Google Cloud Platform (GCP)** in our development and deployment workflow.  
It covers essential services we use: **Artifact Registry**, **Virtual Machines (VMs)**, and **Identity and Access Management (IAM)**.  
These tools form the foundation for hosting, securing, and managing our backend services and containerized applications.

---

## 2. Prerequisites

You will need:
- A GCP account with **Billing Enabled**
- Access to the project managed by the organization
- Installed **gcloud CLI**
- Basic knowledge of Docker (for Artifact Registry usage)

Install gcloud CLI if not already available:
```bash
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
gcloud init
```

---

## 3. Project and Authentication Setup

Authenticate with your GCP account:
```bash
gcloud auth login
```

Set your active project:
```bash
gcloud config set project <PROJECT_ID>
```

Verify your configuration:
```bash
gcloud config list
```

---

## 4. Artifact Registry Setup

Artifact Registry is used for **storing Docker images** and other artifacts securely within GCP.  
We use it for maintaining builds of backend containers (e.g., Directus, API servers).

### Create an Artifact Registry Repository

```bash
gcloud artifacts repositories create my-repo   --repository-format=docker   --location=asia-south1   --description="Docker repository for backend services"
```

### Authenticate Docker with GCP

```bash
gcloud auth configure-docker asia-south1-docker.pkg.dev
```

### Build and Push Docker Image

```bash
docker build -t asia-south1-docker.pkg.dev/<PROJECT_ID>/my-repo/my-app:latest .
docker push asia-south1-docker.pkg.dev/<PROJECT_ID>/my-repo/my-app:latest
```

### Access Pattern
- **Developers**: Can pull images for local testing.
- **Production**: VM pulls the latest Docker image via startup scripts or deployment pipelines.

---

## 5. Virtual Machine (VM) Setup

We use Compute Engine VMs for hosting our backend services (e.g., Directus, Node.js APIs).

### Create a VM Instance

```bash
gcloud compute instances create my-vm   --zone=asia-south1-c   --machine-type=e2-medium   --image-family=debian-12   --image-project=debian-cloud   --boot-disk-size=30GB   --tags=http-server,https-server
```

### SSH into VM

```bash
gcloud compute ssh my-vm --zone=asia-south1-c
```

### Install and Run Docker on VM

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

### Pull Image from Artifact Registry

```bash
docker pull asia-south1-docker.pkg.dev/<PROJECT_ID>/my-repo/my-app:latest
docker run -d -p 8055:8055 my-app:latest
```

This will launch your containerized app (like Directus or Node backend) on the VM.

---

## 6. Basic IAM (Identity and Access Management) Setup

IAM controls who can access what within the project.

### Add a User to the Project

```bash
gcloud projects add-iam-policy-binding <PROJECT_ID>   --member="user:email@example.com"   --role="roles/viewer"
```

### Common Roles Used

| Role | Purpose |
|------|----------|
| `roles/viewer` | Read-only access |
| `roles/editor` | Modify resources except IAM policies |
| `roles/owner` | Full control of project |
| `roles/artifactregistry.admin` | Manage repositories and artifacts |
| `roles/compute.admin` | Manage Compute Engine VMs |
| `roles/secretmanager.admin` | Manage environment variables and secrets |

### Best Practices
- Grant **least privilege** required.
- Use **service accounts** for automated deployments.
- Rotate access tokens regularly.

---

## 7. Managing Secrets and Environment Variables

Use **Secret Manager** to securely store credentials and `.env` values.

### Create a Secret

```bash
echo -n "MY_SECRET_VALUE" | gcloud secrets create my-secret --data-file=-
```

### Access a Secret

```bash
gcloud secrets versions access latest --secret="my-secret"
```

Integrate secrets into VM or container startup scripts using environment variables.

---

## 8. Monitoring and Logging

We use **Cloud Logging** and **Cloud Monitoring** to track performance and application errors.

### View Logs

```bash
gcloud logging read "resource.type=gce_instance"
```

### Enable Monitoring Dashboard
Visit [https://console.cloud.google.com/monitoring](https://console.cloud.google.com/monitoring)  
Add custom metrics to track uptime and container health.

---

## 9. Conclusion and Next Steps

You now have a working foundation for managing infrastructure on GCP:

✅ Store and deploy Docker images with **Artifact Registry**  
✅ Host containerized apps on **Compute Engine VMs**  
✅ Secure access using **IAM and Secret Manager**  
✅ Monitor system health through **Cloud Logging**
