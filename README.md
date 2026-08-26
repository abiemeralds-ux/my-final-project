# 🚀 DevOps Final Project – Jenkins CI/CD with Docker on AWS EC2

## 📌 Project Overview

This project demonstrates an end-to-end DevOps deployment pipeline using:

- GitHub – Source Code Management
- Jenkins – CI/CD Automation
- AWS EC2 – Application Deployment Server
- Amazon Linux 2023 – EC2 Operating System
- Docker – Application Containerization
- Docker Compose – Multi-container Application Deployment
- Docker Buildx – Docker image building
- SSH – Secure communication between Jenkins and EC2

The pipeline automatically retrieves the application source code from GitHub, transfers it to an AWS EC2 instance, builds the Docker images, and deploys the applications using Docker Compose.

---

# 🏗️ Architecture

```text
                         ┌─────────────────────┐
                         │       GitHub        │
                         │                     │
                         │ my-final-project    │
                         └──────────┬──────────┘
                                    │
                                    │ Git Checkout
                                    ▼
                         ┌─────────────────────┐
                         │       Jenkins      │
                         │                     │
                         │ CI/CD Pipeline      │
                         └──────────┬──────────┘
                                    │
                                    │ SSH / SCP
                                    ▼
                    ┌──────────────────────────────┐
                    │          AWS EC2             │
                    │                              │
                    │     Amazon Linux 2023        │
                    │                              │
                    │       Docker Engine          │
                    │             │                │
                    │       Docker Compose         │
                    │             │                │
                    │       ┌─────┴─────┐          │
                    │       ▼           ▼          │
                    │   Portfolio    Java App      │
                    │   Container    Container     │
                    └──────────────────────────────┘
