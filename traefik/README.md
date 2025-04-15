---

# Traefik Reverse Proxy Setup with Docker & Cloudflare DNS

This repository contains a ready-to-use setup of **Traefik** as a modern reverse proxy with automatic HTTPS via Let's Encrypt (DNS Challenge using Cloudflare).

---

## 🧰 Prerequisites

- Docker & Docker Compose installed
- Domain name (e.g., `local.domain.com`)
- [Cloudflare](https://cloudflare.com) account managing your domain DNS
- Cloudflare **API Token** with permission to edit DNS records

---

## 🚀 Setup Instructions

### 1. Create a Docker Network

Traefik and all routed containers must share the same network:

```bash
docker network create proxy
```

---

### 2. Generate Basic Auth (optional but recommended)

To secure the Traefik dashboard, generate a bcrypt password:

1. Visit: [https://www.hostingcanada.org/htpasswd-generator](https://www.hostingcanada.org/htpasswd-generator)
2. Enter your desired **username** and **password**
3. Copy the result (e.g., `admin:$2y$...`) and replace the existing value in `docker-compose.yml` if needed

**Default credentials:**

```
Username: admin
Password: admin
```

Already included in:
```yaml
traefik.http.middlewares.myauth.basicauth.users=admin:$2y$10$y.jfFP2fPRSSnGX2zLibg.xAB6rOG7PjHR.3ltdm0uHi.HWKVBEJ6
```

---

### 3. Clone the Repository

```bash
git clone https://github.com/blackxxxstar/docker.git
cd docker/traefik
```

---

### 4. Configure Environment Variables

Copy and edit the environment file:

```bash
cp .env.example .env
```

In `.env`, set:

```env
CF_DNS_API_TOKEN=your_cloudflare_api_token
```

---

### 5. Start Traefik

Pull and start the services:

```bash
docker compose pull
docker compose up -d
```

---

## 🌐 Accessing the Traefik Dashboard

Once your domain is pointed to the server:

- Go to: `https://traefik.local.domain.com`
- Login using `admin` / `admin` (or your custom credentials)

The dashboard is secured with HTTP Basic Auth.

---

## 🔒 HTTPS via Let's Encrypt (Cloudflare DNS)

This setup uses **DNS-01 challenge** to generate and renew SSL certificates automatically via Cloudflare.

Ensure:

- Your domain's DNS is managed by Cloudflare
- Your API token in `.env` has permission to manage DNS
- The domain and subdomains used are configured properly in:
  ```yaml
  - --entrypoints.websecure.http.tls.domains[0].main=local.domain.com
  - --entrypoints.websecure.http.tls.domains[0].sans=*.local.domain.com
  ```

Certificates will be stored in `./certs/acme.json`.

---

## 🧩 Traefik Labels for Your Services

To expose other services via Traefik, add these labels to your container:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myapp.rule=Host(`app.local.domain.com`)"
  - "traefik.http.routers.myapp.entrypoints=websecure"
  - "traefik.http.routers.myapp.tls.certresolver=myresolver"
  - "traefik.docker.network=proxy"
```

And connect the container to the `proxy` network:

```yaml
networks:
  - proxy
```

---

## 📁 File Structure

```
traefik/
├── docker-compose.yml
├── .env.example
├── certs/
│   └── acme.json  # Let's Encrypt certificate store
```

Make sure `acme.json` exists and has correct permissions:

```bash
touch certs/acme.json
chmod 600 certs/acme.json
```

---

## 📜 License

This setup is provided under the MIT License.

---
