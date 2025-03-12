+++
title = "Self-Hosting a Blog in 2025"
description = "A step-by-step guide to hosting your own website using Zola, Caddy, and Gitea."
date = 2025-03-12
updated = 2025-03-12

[taxonomies]
tags = ["Self-Hosting", "CI/CD", "Linux", "Caddy", "Zola"]
categories = ["Tutorials"]

[extra]
social_media_card = "img/social_cards/blog_self_hosting_a_blog_in_2025.jpg"
+++

## Introduction

[Zola](https://www.getzola.org/) is a static site generator written in Rust that is fast, simple, and easy to use. This blog is built using
Zola and hosted on a Linux server with [Caddy](https://caddyserver.com/) as the web server.

This guide will walk you through setting up Zola and Caddy to self-host your website efficiently.

## Prerequisites

Before starting, ensure you have the following:

1. **A Linux server** – I am using [Pop!_OS 24.04 LTS alpha](https://system76.com/cosmic/).
2. **A domain name** pointing to your server – I use [Cloudflare](https://www.cloudflare.com/).
3. **A reverse proxy** – [Caddy](https://caddyserver.com/) handles this role.
4. **A CI/CD platform** – I use [Gitea](/blog/gitea-open-source-github-alternative) for automated deployments.
5. **A privacy-focused analytics tool** – I use [Plausible](https://plausible.io/).

## Installation

### Step 1: Install Zola

Since there is no precompiled package for Pop!_OS 24.04 LTS alpha, we will install Zola from source:

```bash
git clone https://github.com/getzola/zola.git
cd zola
cargo install --path . --locked
zola --version
```

### Step 2: Create a New Site

Initialize a new Zola site:

```bash
zola init blog
cd blog
git init
echo "public" > .gitignore
```

### Step 3: Install a Zola Theme

I use the [tabi](https://github.com/welpo/tabi.git) theme. To install it:

```bash
git submodule add https://github.com/welpo/tabi.git themes/tabi
```

### Step 4: Configure Zola & Tabi

Zola uses a `config.toml` file for configuration. Below is a sample configuration:

```toml
base_url = "https://www.aldofunes.com"
title = "Aldo Funes"
description = "Human being in the making"
default_language = "en"
theme = "tabi"
compile_sass = false
minify_html = true
author = "Aldo Funes"
taxonomies = [{ name = "tags" }, { name = "categories" }]
build_search_index = true

[markdown]
highlight_code = true
highlight_theme = "css"
highlight_themes_css = [{ theme = "dracula", filename = "css/syntax.css" }]
render_emoji = true
external_links_class = "external"
external_links_target_blank = true
smart_punctuation = true

[search]
index_format = "elasticlunr_json"

[extra]
stylesheets = ["css/syntax.css"]
remote_repository_url = "https://gitea.funes.me/aldo/blog"
remote_repository_git_platform = "gitea"
mermaid = true
show_previous_next_article_links = true
toc = true
favicon_emoji = "👾"
```

### Step 5: Add Content

Zola uses Markdown for content creation, and its directory structure is intuitive. Use your favorite text editor to start writing articles.

### Step 6: Deploy Your Site

To serve the site with Caddy, place the generated files in `/www/blog` and configure Caddy with the following `Caddyfile`:

```
aldofunes.com, www.aldofunes.com {
    tls {
        dns cloudflare __CLOUDFLARE_TOKEN__
        resolvers 1.1.1.1
    }
    root * /www/blog
    file_server
    handle_errors {
        rewrite * /{err.status_code}.html
        file_server
    }
    header           Cache-Control max-age=3600
    header /static/* Cache-Control max-age=31536000
}
```

### Step 7 (Optional): Set Up a CDN

Using Cloudflare as a CDN improves performance and security. Configure a DNS record and enable Cloudflare proxying to benefit from caching
and DDoS protection.

### Step 8: Automate Deployment with CI/CD

To automate deployments with Gitea, create `.gitea/workflows/deploy.yaml`:

```yaml
name: Deploy
on:
  push:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository code
        uses: actions/checkout@v4
        with:
          submodules: true

      - name: Check 🔍
        uses: zolacti/on@check
        with:
          drafts: true

      - name: Build 🛠
        uses: zolacti/on@build

      - name: Deploy 🚀
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ vars.ATLAS_SSH_HOST }}
          username: ${{ vars.ATLAS_SSH_USERNAME }}
          key: ${{ secrets.ATLAS_SSH_KEY }}
          port: ${{ vars.ATLAS_SSH_PORT }}
          source: public
          target: /www/blog
          rm: true
          overwrite: true
          strip_components: 1
```

Set these environment variables in Gitea Actions:

- `ATLAS_SSH_HOST`
- `ATLAS_SSH_USERNAME`
- `ATLAS_SSH_PORT`

And add the secret key:

- `ATLAS_SSH_KEY`

These credentials enable secure deployment via SCP.

## Conclusion

You now have a fully self-hosted website powered by Zola and Caddy. With automated CI/CD using Gitea, you can focus on writing content while
Gitea handles deployment. Enjoy your self-hosted blog!

