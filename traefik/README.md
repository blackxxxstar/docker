# Traefik Reverse Proxy Setup

This guide helps you set up **Traefik** as a reverse proxy using Docker. Follow these steps to get started.

## Prerequisites

- Docker and Docker Compose installed on your system.
- Basic understanding of how reverse proxies and Docker networking work.

---

## Setup Instructions

### 1. Create a Docker Network

This allows Traefik to communicate with other containers via a shared network:

```bash
docker network create proxy
```

> The network must be named `proxy` so that Traefik can automatically detect other containers connected to it.

---

### 2. Generate HTTP Basic Auth Credentials

To secure the Traefik dashboard, generate an `.htpasswd` file with bcrypt encryption.

1. Go to [https://www.hostingcanada.org/htpasswd-generator](https://www.hostingcanada.org/htpasswd-generator)
2. Enter your desired username and password.
3. Copy the output (starting with `$2y$...`) and save it.

---

### 3. Clone the Repository

```bash
git clone https://github.com/blackxxxstar/docker.git
cd docker/traefik
```

---

### 4. Set Environment Variables

Copy the example `.env` file and modify it with your custom values, including the `HTTP_BASIC_AUTH` you generated earlier.

```bash
cp .env.example .env
```

Open `.env` in your editor and configure:

- `CF_DNS_API_TOKEN`: Cloudflare DNS API Token

---

### 5. Start Traefik

Pull the latest Docker images:

```bash
docker compose pull
```

Then start Traefik in detached mode:

```bash
docker compose up -d
```

---

## Notes

- The Traefik dashboard will be accessible at `https://traefik.<your-domain>` if DNS and SSL are correctly configured.
- All your future services (e.g., apps, APIs) should:
  - Be attached to the same `proxy` network.
  - Have appropriate Traefik labels for routing.
- You can use this Traefik instance with other `docker-compose` stacks by adding:
  ```yaml
  networks:
    - proxy
  ```

---

## License

This setup is provided as-is under the MIT License.

---
