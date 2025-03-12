+++
title = "Gitea - Open Source GitHub Alternative"
description = "Self-hosting Gitea, a lightweight GitHub alternative written in Go."
date = 2025-02-17
updated = 2025-03-12

[taxonomies]
tags = ["Self-Hosting", "CI/CD", "Linux", "Gitea", "Go"]

[extra]
social_media_card = "img/social_cards/blog_gitea_open_source_github_alternative.jpg"
+++

## Introduction

Gitea is a lightweight, self-hosted alternative to GitHub, providing Git repository hosting, CI/CD, package management, and team
collaboration features. If you value privacy, control, and flexibility over your development workflow, self-hosting Gitea can be a great
choice.

This guide will walk you through setting up Gitea using Docker and Caddy as a reverse proxy, along with configuring a runner for CI/CD
pipelines.

## Prerequisites

Before we start, ensure you have the following:

1. **A Linux server** - I am using [Pop!_OS 24.04 LTS alpha](https://system76.com/cosmic/).
2. **A domain name** pointing to your server - I am using [Cloudflare](https://www.cloudflare.com/).
3. **A container runtime** - I am using [Docker](https://www.docker.com/).
4. **A reverse proxy** - I am using [Caddy](https://caddyserver.com/).

## Installation

### Step 1: Create a Storage Location

To store repository files and application data, I will create a dataset in my ZFS pool at `/data` (ZFS is optional; use any directory if ZFS
is unavailable):

```bash
sudo mkdir -p /data/gitea
```

### Step 2: Create a Gitea User

We need a dedicated user for running Gitea:

```bash
adduser \
    --system \
    --shell /bin/bash \
    --gecos 'Gitea' \
    --group \
    --disabled-password \
    --home /home/gitea \
    gitea
```

### Step 3: Set Up Docker Compose

Create a `docker-compose.yaml` file for Gitea:

```yaml
# ~/gitea/compose.yaml
services:
  server:
    image: docker.gitea.com/gitea:latest
    environment:
      USER: gitea
      USER_UID: 122  # Ensure this matches the Gitea user ID
      USER_GID: 126  # Ensure this matches the Gitea group ID
      GITEA__database__DB_TYPE: postgres
      GITEA__database__HOST: db:5432
      GITEA__database__NAME: gitea
      GITEA__database__USER: gitea
      GITEA__database__PASSWD: __REDACTED__
      GITEA__server__DOMAIN: gitea.example.com
      GITEA__server__HTTP_PORT: 9473
      GITEA__server__ROOT_URL: https://gitea.example.com/
      GITEA__server__DISABLE_SSH: false
      GITEA__server__SSH_PORT: 22022
    restart: always
    volumes:
      - /data/gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "9473:9473"
      - "22022:22"
    depends_on:
      - db

  db:
    image: docker.io/library/postgres:17
    restart: always
    environment:
      POSTGRES_USER: gitea
      POSTGRES_PASSWORD: __REDACTED__
      POSTGRES_DB: gitea
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
    driver: local
```

### Step 4: Start Gitea

Run the following command to start Gitea:

```bash
docker compose up --detach
```

> **Note:** Gitea's SSH server is set to port `22022` because port `22` is already in use by my existing SSH setup. Adjust this as needed.

## Setting Up the Reverse Proxy

We will use Caddy to handle HTTPS and reverse proxy requests for Gitea. Add the following to your `Caddyfile`:

```plaintext
gitea.example.com {
    tls {
        dns cloudflare __CLOUDFLARE_TOKEN__
        resolvers 1.1.1.1
    }
    reverse_proxy localhost:9473
}
```

Reload Caddy to apply the changes:

```bash
sudo systemctl reload caddy
```

## Configuration

Now, open your browser and navigate to `https://gitea.example.com` (or `http://localhost:9473` if testing locally). Complete the setup by
filling in:

- Admin account details
- Database configuration
- SMTP settings (if needed)

Once configured, click **Install Gitea**.

## Setting Up CI/CD

Gitea has built-in CI/CD capabilities. We will deploy an [Act Runner](https://docs.gitea.com/usage/actions/act-runner) to run pipelines.

### Step 1: Create Runner Configuration

The following `runner-config.yaml` addresses an issue where the runner cannot cache due to job containers being on different networks:

```yaml
# ~/gitea/runner-config.yaml
cache:
  enabled: true
  dir: "/data/cache"
  host: "192.168.1.10"  # Replace with your server's private IP
  port: 9012  # Port Gitea listens on
```

### Step 2: Update Docker Compose

Modify `docker-compose.yaml` to add the runner service:

```yaml
# ~/gitea/compose.yaml
services:
  runner:
    image: docker.io/gitea/act_runner:latest
    environment:
      CONFIG_FILE: /config.yaml
      GITEA_INSTANCE_URL: https://gitea.example.com/
      GITEA_RUNNER_REGISTRATION_TOKEN: __REDACTED__
      GITEA_RUNNER_NAME: runner-1
    ports:
      - "9012:9012"
    volumes:
      - ./runner-config.yaml:/config.yaml
      - gitea-runner-data:/data
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  gitea-runner-data:
    driver: local
```

### Step 3: Start the Runner

Run the following command:

```bash
docker compose up --detach
```

## Conclusion

Self-hosting Gitea is relatively straightforward and provides full control over your development workflow. However, setting up CI/CD and
reverse proxying may require some tweaking to fit your setup.

### Next Steps

- Configure **OAuth authentication** (e.g., GitHub, GitLab, LDAP).
- Set up **automated backups** to avoid data loss.
- Enable **Gitea Webhooks** for integration with external services.

If you have any questions, feel free to reach out. Happy coding!

