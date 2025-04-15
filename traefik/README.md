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

> The network must be named `proxy`.

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

- Visit: `https://traefik.local.domain.com`
- Login using `admin` / `admin` (or your custom credentials)

Dashboard is secured with HTTP Basic Auth.

---

## 🔒 HTTPS via Let's Encrypt (Cloudflare DNS)

This setup uses **DNS-01 challenge** to generate and renew SSL certificates automatically via Cloudflare.

Certificates are stored in `./certs/acme.json`.

Make sure the file exists and has correct permissions:

```bash
touch certs/acme.json
chmod 600 certs/acme.json
```

---

### 🔐 Cloudflare API Tokens

To allow Traefik to manage DNS records for certificate generation, create an API token with:

- **Zone / Zone / Read**
- **Zone / DNS / Edit**

Scope the access to **all zones** you'll be using.

Use the token in `.env`:

```env
CF_DNS_API_TOKEN=your_cloudflare_api_token
```

> This token is passed to [Lego](https://go-acme.github.io/lego/) — the ACME client used by Traefik.

More info: [Cloudflare API Token Docs](https://developers.cloudflare.com/api/tokens/create/)

---

## 🧩 Traefik Labels for Your Services

To expose a container through Traefik, example labels:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myapp.rule=Host(`app.local.domain.com`)"
  - "traefik.http.routers.myapp.entrypoints=websecure"
  - "traefik.http.routers.myapp.tls.certresolver=myresolver"
  - "traefik.docker.network=proxy"
```

And connect to the `proxy` network:

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

---

## 📜 License

This setup is provided under the MIT License.

---
