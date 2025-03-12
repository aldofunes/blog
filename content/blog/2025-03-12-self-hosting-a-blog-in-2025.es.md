+++
title = "Autoalojamiento de un Blog en 2025"
description = "Guía paso a paso para alojar tu propio sitio web usando Zola, Caddy y Gitea."
date = 2025-03-12
updated = 2025-03-12

[taxonomies]
tags = ["Autoalojamiento", "CI/CD", "Linux", "Caddy", "Zola"]
categories = ["Tutoriales"]

[extra]
social_media_card = "img/social_cards/blog_self_hosting_a_blog_in_2025.jpg"
+++

## Introducción

[Zola](https://www.getzola.org/) es un generador de sitios estáticos escrito en Rust que es rápido, simple y fácil de usar. Este blog está
construido con Zola y alojado en un servidor Linux con [Caddy](https://caddyserver.com/) como servidor web.

Esta guía te guiará a través de la configuración de Zola y Caddy para autoalojar tu sitio web de manera eficiente.

## Requisitos previos

Antes de comenzar, asegúrate de tener lo siguiente:

1. **Un servidor Linux** – Estoy usando [Pop!_OS 24.04 LTS alpha](https://system76.com/cosmic/).
2. **Un nombre de dominio** apuntando a tu servidor – Yo uso [Cloudflare](https://www.cloudflare.com/).
3. **Un proxy inverso** – [Caddy](https://caddyserver.com/) cumple esta función.
4. **Una plataforma CI/CD** – Uso [Gitea](/blog/gitea-open-source-github-alternative) para despliegues automatizados.
5. **Una herramienta de análisis enfocada en la privacidad** – Uso [Plausible](https://plausible.io/).

## Instalación

### Paso 1: Instalar Zola

Como no hay un paquete precompilado para Pop!_OS 24.04 LTS alpha, instalaremos Zola desde el código fuente:

```bash
git clone https://github.com/getzola/zola.git
cd zola
cargo install --path . --locked
zola --version
```

### Paso 2: Crear un Nuevo Sitio

Inicializa un nuevo sitio con Zola:

```bash
zola init blog
cd blog
git init
echo "public" > .gitignore
```

### Paso 3: Instalar un Tema de Zola

Uso el tema [tabi](https://github.com/welpo/tabi.git). Para instalarlo:

```bash
git submodule add https://github.com/welpo/tabi.git themes/tabi
```

### Paso 4: Configurar Zola y Tabi

Zola usa un archivo `config.toml` para la configuración. Aquí hay un ejemplo:

```toml
base_url = "https://www.aldofunes.com"
title = "Aldo Funes"
description = "Ser humano en formación"
default_language = "es"
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

### Paso 5: Agregar Contenido

Zola usa Markdown para la creación de contenido, y su estructura de directorios es intuitiva. Usa tu editor de texto favorito para empezar a
escribir artículos.

### Paso 6: Desplegar tu Sitio

Para servir el sitio con Caddy, coloca los archivos generados en `/www/blog` y configura Caddy con el siguiente `Caddyfile`:

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

### Paso 7 (Opcional): Configurar un CDN

Usar Cloudflare como CDN mejora el rendimiento y la seguridad. Configura un registro DNS y habilita la protección de Cloudflare para
aprovechar la caché y la protección contra DDoS.

### Paso 8: Automatizar el Despliegue con CI/CD

Para automatizar los despliegues con Gitea, crea `.gitea/workflows/deploy.yaml`:

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

Configura estas variables de entorno en Gitea Actions:

- `ATLAS_SSH_HOST`
- `ATLAS_SSH_USERNAME`
- `ATLAS_SSH_PORT`

Y agrega la clave secreta:

- `ATLAS_SSH_KEY`

Estas credenciales permiten un despliegue seguro mediante SCP.

## Conclusión

Ahora tienes un sitio web completamente autoalojado con Zola y Caddy. Con CI/CD automatizado mediante Gitea, puedes concentrarte en escribir
contenido mientras Gitea maneja el despliegue. ¡Disfruta de tu blog autoalojado!
