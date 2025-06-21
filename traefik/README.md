# 🚀 Traefik Reverse Proxy with Cloudflare DNS & Google Public CA (EAB)
This repository contains the configuration to deploy **Traefik** as a reverse proxy using:
- **Cloudflare DNS Challenge** for automatic TLS certificate issuance.
- **Google Public CA** via ACME protocol with **External Account Binding (EAB)**.
- Secure dashboard access with HTTP Basic Authentication.
---

## 📁 Directory Structure
```bash
/opt/traefik
│
├── certs/                  # ACME Certificates storage
├── .env                    # Environment variables
└── docker-compose.yml      # Traefik Docker Compose configuration
```
---

## 🛠️ Setup Steps
### 1. Create Required Folders
```bash
sudo mkdir -p /opt/traefik/certs
