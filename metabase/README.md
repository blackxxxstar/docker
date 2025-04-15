# 🚀 Metabase with Docker Compose Setup

This guide shows you how to deploy **Metabase** and a PostgreSQL database using Docker Compose. It includes dummy configurations for easy setup and testing.

---

## 📦 Prerequisites

- **Docker** & **Docker Compose** installed on your machine
- Domain name for Metabase dashboard (e.g., `dummydomain.com`)

---

## 🔧 Setup Instructions

### 1. Clone the Repository (if applicable)

First, clone the repository containing the setup:

```bash
git clone https://github.com/your-repository-name/your-project.git
cd your-project/metabase
```

---

### 2. Create the `.env` File

Create a `.env` file with the following content, which contains the database connection details (dummy data used here):

```env
DB_HOST=metabase_db
DB_TYPE=postgres
DB_DBNAME=metabase
DB_PORT=5432
DB_USER=metabase
DB_PASS=metabase
```

Make sure to place this file in the same directory as your `docker-compose.yml`.

---

### 3. Docker Compose Configuration

Here’s the `docker-compose.yml` configuration for **Metabase** and its **PostgreSQL database**:

```yaml
services:
  metabase:
    image: metabase/metabase
    container_name: metabase
    hostname: metabase
    volumes:
      - /dev/urandom:/dev/random:ro
    environment:
      MB_DB_HOST: ${DB_HOST}
      MB_DB_TYPE: ${DB_TYPE}
      MB_DB_DBNAME: ${DB_DBNAME}
      MB_DB_PORT: ${DB_PORT}
      MB_DB_USER: ${DB_USER}
      MB_DB_PASS: ${DB_PASS}
    networks:
      - proxy
      - backend
    healthcheck:
      test: curl --fail -I http://localhost:3000/api/health || exit 1
      interval: 15s
      timeout: 5s
      retries: 5
    labels:
      - traefik.enable=true
      - traefik.docker.network=proxy
      - traefik.http.services.metabase.loadbalancer.server.port=3000
      - traefik.http.routers.metabase.rule=Host(`dummydomain.com`)
      - traefik.http.routers.metabase.entrypoints=websecure
      - traefik.http.routers.metabase.tls.certresolver=myresolver

  metabase_db:
    image: postgres
    container_name: metabase_db
    hostname: metabase_db
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_DB: ${DB_DBNAME}
      POSTGRES_PASSWORD: ${DB_PASS}
    networks:
      - backend

networks:
  proxy:
    external: true
  backend:
    external: true
```

---

### 4. Start the Services

Run the following command to pull the necessary images and start the services:

```bash
docker-compose pull
docker-compose up -d
```

This will start both **Metabase** and the **PostgreSQL database** containers.

---

### 5. Access the Metabase Dashboard

Once the containers are up and running, you can access the Metabase dashboard via the domain you specified in the labels (in this case, `dummydomain.com`).

---

## 🧩 Traefik Integration

This setup assumes that you're using **Traefik** as a reverse proxy. The **Metabase service** is exposed on port `3000`, and it is protected by **TLS** with **Let’s Encrypt** certificates.

Make sure the following Traefik labels are included in your `docker-compose.yml`:

```yaml
labels:
  - traefik.enable=true
  - traefik.docker.network=proxy
  - traefik.http.services.metabase.loadbalancer.server.port=3000
  - traefik.http.routers.metabase.rule=Host(`dummydomain.com`)
  - traefik.http.routers.metabase.entrypoints=websecure
  - traefik.http.routers.metabase.tls.certresolver=myresolver
```

This allows the Metabase service to be routed properly via Traefik using HTTPS.

---

## 📁 File Structure

```
metabase/
├── docker-compose.yml
├── .env
└── other-configuration-files/
```

---

## 🔒 Security Considerations

- **Environment Variables**: Ensure that your `.env` file is secured and not exposed to unauthorized users.
- **Default Credentials**: It's recommended to change the default username and password for the Metabase service once the setup is complete.

---
