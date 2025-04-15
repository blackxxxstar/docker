# 🐳 Docker Host Setup (Ubuntu)

This guide provides a step-by-step setup for installing **Docker**, **Docker Compose**, and essential CLI tools on a fresh Ubuntu machine. It also includes bash completion setup for better terminal UX.

---

## 📦 Step 1 – Install Docker & Dependencies

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

---

## 📁 Step 2 – Add Docker Repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

---

## 🐋 Step 3 – Install Docker & Compose

```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin bash-completion nano wget curl
```

---

## 👤 Step 4 – Add User to Docker Group

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

> This allows running Docker without `sudo`.

---

## 🔁 Step 5 – Enable Docker Services

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

---

## 🧠 Step 6 – Terminal Helpers & Completion

Install helpers:

```bash
sudo apt install -y bash-completion nano wget curl
```

Add Docker auto-completion to `.bashrc`:

```bash
nano ~/.bashrc
```

Paste this at the bottom:

```bash
# Docker CLI completion
if [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
fi
```

Then apply it:

```bash
source ~/.bashrc
```

Generate completion file for Docker:

```bash
mkdir -p ~/.local/share/bash-completion/completions
docker completion bash > ~/.local/share/bash-completion/completions/docker
```

---

## 🔌 SSH Back into Host (if required)

```bash
exit
ssh user@ubuntu
```

Check if Docker is working:

```bash
docker ps
```

---

## ✅ Done!

Your server is now ready to run containers using Docker and Docker Compose.

---
