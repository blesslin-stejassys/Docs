# 🏗️ Infrastructure Documentation

This repository contains the complete **infrastructure documentation** for our application ecosystem — covering backend services, monitoring tools, cloud deployment setup, and integrations.  
It is designed to help developers, DevOps engineers, and contributors quickly understand the architecture, tools, and access patterns used across our stack.

---

## 📘 Overview

The infrastructure stack includes multiple key services working together to support our production and development workflows:

| Component | Purpose |
|------------|----------|
| **Sentry** | Real-time error tracking and performance monitoring for web and mobile apps |
| **Supabase** | PostgreSQL-based backend as a service with authentication and APIs |
| **Directus** | Headless CMS and data management layer integrated with Supabase |
| **GCP** | Hosting, artifact management, and IAM for secure and scalable deployment |
| **Next.js** | Frontend framework integrated with Supabase and Directus for seamless UI experience |

Each module is documented in a separate `.md` file for clarity and modularity.

---

## 🗂️ Repository Structure

infra-docs/
├── README.md # Repository overview (this file)
├── sentry.md # Sentry setup, integrations, and monitoring workflow
├── supabase.md # Supabase project creation, database setup, and Next.js integration
├── directus.md # Directus setup, Docker config, and CMS connection with Next.js
├── gcp.md # GCP infrastructure setup for VM, Artifact Registry, IAM, and logging

yaml
Copy code

---

## ⚙️ Setup Summary

| Tool | Key Responsibilities |
|------|------------------------|
| **Sentry** | Error monitoring, release tracking, and alerts |
| **Supabase** | Database, authentication, and API backend |
| **Directus** | Data visualization, content management, and API bridge |
| **GCP** | Hosting backend (VMs), container management, and access control |
| **Next.js** | Frontend interface with data fetched from Supabase/Directus |

---

## 🧭 How to Use This Repository

1. **Start with `gcp.md`** – to understand cloud deployment and IAM setup.  
2. **Move to `supabase.md`** – for backend setup and connecting with your app.  
3. **Refer to `directus.md`** – for CMS setup and data management integration.  
4. **Follow `sentry.md`** – to implement monitoring and alerting in all environments.  
5. Keep `README.md` as your high-level guide for onboarding new team members.

---

## 🔐 Access and Permissions

- Access to GCP, Supabase, and Sentry credentials is managed by **Varun**.
- All environment variables (`.env`, `.env.local`) are stored securely using **GCP Secret Manager**.
- Docker images and deployments are version-controlled through **GCP Artifact Registry**.

---


## 🚀 Deployment Flow (Simplified)

```mermaid
graph TD;
  A[Developer] -->|Push Code| B[GitHub Repository];
  B --> C[Artifact Registry - Docker Build];
  C --> D[VM Deployment on GCP];
  D --> E[Next.js Frontend on Vercel];
  E --> F[Supabase + Directus + Sentry Integration];
👨‍💻 Maintainers
Name	Role	Responsibilities
Varun	Infrastructure Lead	Cloud setup, IAM, and deployments
Blesslin	Developer	Integration, documentation, and automation setup
📄 License
This repository is for internal documentation purposes only.
Do not distribute or publish without authorization.
✅ Next Steps
Add CI/CD pipeline documentation (GitHub Actions or Cloud Build)

Include network and load balancer setup

Document backup and restore strategy for Supabase and Directus
