---

# 🌐 Traefik Reverse Proxy with Docker & Cloudflare DNS

This repository provides a ready-to-use setup for **Traefik** as a reverse proxy, featuring:

- HTTPS with Let's Encrypt via **Cloudflare DNS Challenge**
- Basic Auth-protected **Traefik dashboard**
- Docker Compose-based deployment
- Dynamic routing using container labels

---

## 🧰 Prerequisites

- Docker & Docker Compose installed
- A domain (e.g., `exampledomain.com`)
- Cloudflare account managing your domain DNS
- Cloudflare API Token with required permissions

---

## 🚀 Setup Instructions

### 1. Create Docker Network

Traefik and routed containers must share the same Docker network:

```bash
docker network create proxy
```

---

### 2. Generate Basic Auth

To secure the Traefik dashboard with basic auth:

1. Go to [https://www.hostingcanada.org/htpasswd-generator](https://www.hostingcanada.org/htpasswd-generator)
2. Enter your username (`admin`) and password (`admin`) or your own
3. Copy the output (e.g., `admin:$2y$...`) and use it in your `docker-compose.yml`

**Default included:**

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

### 4. Configure Environment

Copy the example `.env` and set your Cloudflare token:

```bash
cp .env.example .env
```

Edit `.env`:

```env
CF_DNS_API_TOKEN=your_cloudflare_api_token
```

---

### 5. Start Traefik

```bash
docker compose pull
docker compose up -d
```

---

## 🔒 Cloudflare API Token

To allow Traefik to generate SSL certificates using Cloudflare DNS, you must create a token with the following permissions:

- **Zone / Zone / Read**
- **Zone / DNS / Edit**

Scope it to **all zones** you'll be using, and set it as `CF_DNS_API_TOKEN` in your `.env` file.

> Traefik uses [Lego](https://go-acme.github.io/lego/) under the hood to handle certificate requests.

More info: [Cloudflare Token Guide](https://developers.cloudflare.com/api/tokens/create/)

---

## 🛠️ Traefik Configuration Summary

Traefik will be available at:

```https
https://traefik.local.domain.com
```

Protected with:

- **Username**: `admin`
- **Password**: `admin`

All routed containers must connect to the `proxy` network and use appropriate labels.

---

## 🧩 Traefik Labels for Your Services

Here's how to expose a container through Traefik:

```yaml
services:
  myapp:
    image: myapp-image
    container_name: myapp
    ports:
      - "8080:8080"
    networks:
      - proxy
    labels:
      - traefik.enable=true
      - traefik.docker.network=proxy
      - traefik.http.services.myapp.loadbalancer.server.port=8080
      - traefik.http.routers.myapp.rule=Host(`app.exampledomain.com`)
      - traefik.http.routers.myapp.entrypoints=websecure
      - traefik.http.routers.myapp.tls.certresolver=myresolver

networks:
  proxy:
    external: true
```

### Explanation:

| Label | Purpose |
|-------|---------|
| `traefik.enable=true` | Enables routing for this container |
| `traefik.docker.network=proxy` | Specifies the shared Docker network |
| `traefik.http.services.myapp.loadbalancer.server.port=8080` | App’s internal port |
| `traefik.http.routers.myapp.rule=Host(...)` | Domain name used to access the app |
| `traefik.http.routers.myapp.entrypoints=websecure` | Enables HTTPS |
| `traefik.http.routers.myapp.tls.certresolver=myresolver` | Uses Let's Encrypt resolver |

---

## 📁 File Structure

```
docker/
└── traefik/
    ├── docker-compose.yml
    ├── .env.example
    └── certs/
        └── acme.json  # Let's Encrypt certificate store
```

Make sure `acme.json` exists:

```bash
touch certs/acme.json
chmod 600 certs/acme.json
```

---

## 📜 License

MIT License

---
