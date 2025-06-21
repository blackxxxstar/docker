
---

# 🚀 Traefik Reverse Proxy with Cloudflare DNS & Google Public CA (EAB)

This repository contains the configuration to deploy [Traefik](https://traefik.io/) as a reverse proxy using:

* **Cloudflare DNS Challenge** for automatic TLS certificate issuance.
* **Google Public CA** via ACME protocol with **External Account Binding (EAB)**.
* Secure dashboard access with HTTP Basic Authentication.

> **Note:**
> This example is designed for **local or internal environments** to allow SSL on `localhost` by pre-configuring your **local DNS server or `/etc/hosts` file** to resolve `*.local.example.com` to your Traefik host.

---

## 📁 Directory Structure

```
/opt/traefik
│
├── certs/                  # ACME certificates storage
├── .env                    # Environment variables
└── docker-compose.yml      # Traefik Docker Compose configuration
```

---

## 🛠️ Setup Steps

### 1. Create Required Folders

```bash
sudo mkdir -p /opt/traefik/certs
```

### 2. Create Docker Network

```bash
docker network create proxy
```

### 3. Configure Environment Variables

Create a `.env` file:

```dotenv
CF_DNS_API_TOKEN=your_cloudflare_dns_token
ACME_EAB_KID=chureeth7ooj0bip9jeghaivootaib2H
ACME_EAB_HMAC_KEY=caiYaemah3chahvaiQuaof1eeWeeph6me4uod5ahei2kahHoolaijooshaig1tao
AUTH_USERS=admin
AUTH_PASSWORD=$$2y$$10$$BAqQbyOOU2hdhqPuCk57OuEcMTLqK0.kXTt63B7DZGxqBxEydlyt6
```

💡 **Explanation:**

| Variable            | Description                                                              |
| ------------------- | ------------------------------------------------------------------------ |
| `CF_DNS_API_TOKEN`  | Cloudflare API Token with **Zone\:Read** and **DNS\:Edit** permissions.  |
| `ACME_EAB_KID`      | EAB Key ID obtained from Google Public CA.                               |
| `ACME_EAB_HMAC_KEY` | EAB HMAC Key obtained from Google Public CA.                             |
| `AUTH_USERS`        | HTTP Basic Auth username for Traefik Dashboard access. Example: `admin`. |
| `AUTH_PASSWORD`     | **bcrypt hash** for Traefik Dashboard password.                          |

> **Important about `AUTH_PASSWORD`:**
> Since Docker Compose automatically parses `$` for variable substitution, all dollar signs (`$`) in the bcrypt hash **must be escaped by doubling them (`$$`)** inside the `.env` file.
> For example:
>
> Original bcrypt hash:
>
> ```
> $2y$10$BAqQbyOOU2hdhqPuCk57OuEcMTLqK0.kXTt63B7DZGxqBxEydlyt6
> ```
>
> Must be written in `.env` as:
>
> ```
> $$2y$$10$$BAqQbyOOU2hdhqPuCk57OuEcMTLqK0.kXTt63B7DZGxqBxEydlyt6
> ```
>
> Otherwise, Docker Compose will throw a variable substitution error.

---

## 🔑 How to Generate Bcrypt Password for Traefik Dashboard

You can generate a **bcrypt password hash** on Ubuntu 24.04 Server using either **`htpasswd`** (from `apache2-utils`) or Python.

Here's how to generate a bcrypt hash for:

* **Username**: `admin`
* **Password**: `admin`

---

### ✅ Method 1: Using `htpasswd` (bcrypt)

1. Install `apache2-utils` (if not already installed):

   ```bash
   sudo apt update
   sudo apt install apache2-utils
   ```

2. Generate bcrypt hash:

   ```bash
   htpasswd -nbB admin admin
   ```

   Example output:

   ```
   admin:$2y$05$QXeU8rF8GHFd0/ybxGJOf.OZIbfrKj.vz9ykT9S6Xfdbyxy1YjvZa
   ```

---

### ✅ Method 2: Using Python

```bash
python3 -c 'import bcrypt; print(bcrypt.hashpw(b"admin", bcrypt.gensalt()).decode())'
```

Example output:

```
$2b$12$5q8f1k3YOz0G8W4p8JDIf.fzYejIJoXRe1TuAsG4XbHGFbgO1kaEi
```

Use only the hash part (without `admin:`) and remember to escape `$` with `$$` in `.env`.

---

## 🐳 Docker Compose Configuration

Create `docker-compose.yml` file:

```yaml
name: traefik
services:
  traefik:
    container_name: traefik
    image: traefik
    command:
      - --api.dashboard=true
      - --log.level=DEBUG
      - --accesslog=true
      - --providers.docker=true
      - --providers.docker.exposedByDefault=false
      - --providers.docker.network=proxy

      - --entrypoints.web.address=:80
      - --entrypoints.web.http.redirections.entrypoint.to=websecure
      - --entrypoints.web.http.redirections.entrypoint.scheme=https

      - --entrypoints.websecure.address=:443
      - --entrypoints.websecure.asDefault=true
      - --entrypoints.websecure.http.tls=true
      - --entrypoints.websecure.http.tls.certresolver=myresolver
      - --entrypoints.websecure.http.tls.domains[0].main=local.example.com
      - --entrypoints.websecure.http.tls.domains[0].sans=*.local.example.com

      - --certificatesresolvers.myresolver.acme.dnschallenge=true
      - --certificatesresolvers.myresolver.acme.dnschallenge.provider=cloudflare
      - --certificatesresolvers.myresolver.acme.email=example@mail.com
      - --certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json
      - --certificatesresolvers.myresolver.acme.caServer=https://dv.acme-v02.api.pki.goog/directory

      - --certificatesresolvers.myresolver.acme.eab=true
      - --certificatesresolvers.myresolver.acme.eab.kid=${ACME_EAB_KID}
      - --certificatesresolvers.myresolver.acme.eab.hmacEncoded=${ACME_EAB_HMAC_KEY}
    environment:
      - CF_DNS_API_TOKEN=${CF_DNS_API_TOKEN}
    labels:
      - traefik.enable=true
      - traefik.docker.network=proxy
      - traefik.http.routers.mydashboard.entrypoints=websecure
      - traefik.http.routers.mydashboard.tls.certresolver=myresolver
      - traefik.http.routers.mydashboard.rule=Host(`traefik.local.example.com`)
      - traefik.http.routers.mydashboard.service=api@internal
      - traefik.http.routers.mydashboard.middlewares=myauth
      - traefik.http.services.mydashboard.loadbalancer.server.port=1337
      - traefik.http.middlewares.myauth.basicauth.users=${AUTH_USERS}:${AUTH_PASSWORD}
    networks:
      - proxy
    ports:
      - 80:80
      - 443:443
    restart: unless-stopped
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./certs:/letsencrypt

networks:
  proxy:
    external: true
```

---

## 🔒 ACME EAB (External Account Binding) for Google Public CA

This configuration allows Traefik to issue certificates via **Google Public CA** using EAB credentials:

```yaml
--certificatesresolvers.myresolver.acme.eab.kid=${ACME_EAB_KID}
--certificatesresolvers.myresolver.acme.eab.hmacEncoded=${ACME_EAB_HMAC_KEY}
```

### How to Obtain `ACME_EAB_KID` and `ACME_EAB_HMAC_KEY`:

1. Enable the Public CA API:

```bash
gcloud services enable publicca.googleapis.com
```

2. Generate EAB credentials:

```bash
gcloud publicca external-account-keys create
```

Example output:

```json
{
  "keyId": "chureeth7ooj0bip9jeghaivootaib2H",
  "b64MacKey": "caiYaemah3chahvaiQuaof1eeWeeph6me4uod5ahei2kahHoolaijooshaig1tao"
}
```

Use:

* `keyId` → `ACME_EAB_KID`
* `b64MacKey` → `ACME_EAB_HMAC_KEY`

📚 Reference: [Google Cloud Public CA Tutorial](https://cloud.google.com/certificate-manager/docs/public-ca-tutorial?hl=en)

---

## ☁️ Cloudflare DNS Challenge

Traefik uses **Cloudflare DNS-01 challenge** for wildcard certificate generation.

Required environment variable:

```dotenv
CF_DNS_API_TOKEN=your_cloudflare_dns_token
```

This token requires:

* Zone / Zone / Read
* Zone / DNS / Edit

📚 Reference: [lego Cloudflare Provider Docs](https://go-acme.github.io/lego/dns/cloudflare/)

---

## 🌐 Access Traefik Dashboard

* URL: `https://traefik.local.example.com`
* Authentication: Use credentials defined in the `.env` file.

---

## 🚀 Running Traefik

```bash
docker compose up -d
```

Check logs:

```bash
docker logs -f traefik
```

---

## ⚠️ Notes & Recommendations

* Replace:

  * `local.example.com` with your own domain.
  * Email (`example@mail.com`) with your valid contact email.
* Ensure your **local DNS server or `/etc/hosts` file** resolves `*.local.example.com` to your Traefik host if testing in a local/internal environment.
* ACME DNS Challenge requires a valid Cloudflare-managed domain.
* ACME EAB credentials are valid for 7 days and for first-time ACME account registration only.
* Apply firewall and Docker security best practices in production environments.

---

## 📝 References

* [Traefik Official Docs](https://doc.traefik.io/traefik/)
* [Google Cloud Public CA](https://cloud.google.com/certificate-manager/docs/public-ca-overview)
* [lego DNS Cloudflare Docs](https://go-acme.github.io/lego/dns/cloudflare/)
* [ACME External Account Binding RFC](https://datatracker.ietf.org/doc/html/rfc8555#section-7.3.4)

---
