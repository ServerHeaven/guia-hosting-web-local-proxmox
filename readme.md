<p align="center">
  <img src="https://img.shields.io/badge/Proxmox_VE-E57000?style=for-the-badge&logo=proxmox&logoColor=white" alt="Proxmox">
  <img src="https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white" alt="Nginx">
  <img src="https://img.shields.io/badge/Debian_13-A81D33?style=for-the-badge&logo=debian&logoColor=white" alt="Debian">
  <img src="https://img.shields.io/badge/LXC-333333?style=for-the-badge&logo=linux&logoColor=white" alt="LXC">
  <img src="https://img.shields.io/badge/100%25_Local-22c55e?style=for-the-badge" alt="Local">
</p>

<h1 align="center">🖥️ Hostea webs HTML en Proxmox LXC<br>Guía completa</h1>

<p align="center">
  <strong>Sirve sitios estáticos desde un contenedor Linux ultra ligero en tu red local.<br>Sin cloud · Sin costes · Sin límites · Rendimiento instantáneo.</strong>
</p>

<p align="center">
  <a href="https://github.com/ServerHeaven/guia-hosting-web-local-proxmox/issues">🐛 Reportar error / Report a bug</a> ·
  <a href="https://github.com/ServerHeaven/guia-hosting-web-local-proxmox/issues">💡 Sugerir mejora / Suggest an improvement</a> ·
  <a href="#english">🇬🇧 English</a>
</p>

-----

## 📖 ¿Qué es esto?

Una **guía web interactiva y bilingüe ES/EN** (un solo archivo HTML, selector **ES / EN** o `?lang=en`; solo carga tipografías de Google Fonts) que explica paso a paso cómo montar un servidor web local en Proxmox VE usando un contenedor LXC con Nginx. Cubre desde la creación del contenedor hasta HTTPS, multi-sitio, auto-deploy y seguridad.

Ideal si ya tienes un servidor Proxmox en casa y quieres hostear dashboards, documentación, guías, portafolios o cualquier sitio estático sin depender de servicios externos.

## ⚡ ¿Por qué no usar GitHub Pages u otro hosting gratuito?

|                 |GitHub Pages                   |Hosting en LXC                         |
|-----------------|:-----------------------------:|:-------------------------------------:|
|Coste            |Gratis                         |Gratis (tu hardware)                   |
|Privacidad       |Público obligatorio (plan free)|100% privado en tu red                 |
|Ancho de banda   |100 GB/mes soft limit          |Sin límite                             |
|Tamaño del sitio |1 GB máx.                      |Lo que tu disco permita                |
|Latencia         |~50-200ms                      |<1ms (red local)                       |
|Requiere internet|Sí                             |No                                     |
|Server-side      |No                             |Sí (Nginx config, redirects, headers)  |
|Dominios custom  |`.github.io` o custom          |Lo que quieras (`.lan`, `.local`, etc.)|

**TL;DR:** GitHub Pages es genial para sitios públicos. LXC local es mejor para todo lo que quieres mantener privado, sin restricciones y con acceso instantáneo.

## 🗂️ Contenido de la guía

|Sección                 |Qué aprenderás                                             |
|------------------------|-----------------------------------------------------------|
|**01 · Introducción**   |Ventajas de hostear localmente en Proxmox                  |
|**02 · Requisitos**     |Hardware mínimo: 1 core, 64 MB RAM, 1 GB disco             |
|**03 · Crear LXC**      |Contenedor Debian 13 no-privilegiado con IP fija           |
|**04 · Instalar Nginx** |Servidor web con gzip, caché y config optimizada           |
|**05 · Subir tu web**   |4 métodos: SCP, rsync, Git clone, ZIP                      |
|**06 · Multi-sitio**    |Hostear varias webs: por subdirectorios, puertos o dominios|
|**07 · Dominio local**  |DNS con `/etc/hosts`, Pi-hole o AdGuard Home               |
|**08 · HTTPS local**    |Certificados con mkcert (sin warnings) o auto-firmados     |
|**09 · Auto-deploy**    |Cron + git pull, webhooks de GitHub, aliases rsync         |
|**10 · Seguridad**      |UFW, headers Nginx, SSH hardening                          |
|**11 · Troubleshooting**|Tabla de errores comunes + comandos de diagnóstico         |

## ✅ Requisitos mínimos

|Componente       |Mínimo             |Recomendado|
|-----------------|-------------------|-----------|
|Proxmox VE       |8.x                |Última 8.x o 9.x|
|CPU              |1 core             |1 core     |
|RAM              |64 MB              |128–256 MB |
|Disco            |1 GB               |2–4 GB     |
|SO del contenedor|Debian 12/13 o Alpine|Debian 13  |


> Un contenedor LXC con Nginx consume ~50-80 MB de RAM. Puedes tener decenas corriendo simultáneamente sin problemas.

## 🚀 Inicio rápido

### Ver la guía

```bash
git clone https://github.com/ServerHeaven/guia-hosting-web-local-proxmox.git
cd guia-hosting-web-local-proxmox
open index.html   # macOS; en Linux: xdg-open index.html
```


### Resumen express (si no quieres leer la guía entera)

```bash
# 1. Crear LXC desde el host Proxmox
pct create 200 local:vztmpl/debian-13-standard_13.1-2_amd64.tar.zst \
  --hostname webserver --cores 1 --memory 256 \
  --rootfs local-lvm:4 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.200/24,gw=192.168.1.1 \
  --unprivileged 1 --onboot 1 --start 1

# 2. Entrar al contenedor e instalar Nginx
pct enter 200
apt update && apt install -y nginx

# 3. Subir tu web (desde tu máquina local)
scp -r ./mi-web/* root@192.168.1.200:/var/www/html/

# 4. Listo → http://192.168.1.200
```

## 📁 Estructura del repositorio

```
.
├── index.html     # Guía web completa bilingüe (archivo único)
└── readme.md      # Este archivo
```

## 🧰 Cheatsheet de comandos

```bash
# ─── PROXMOX ────────────────────────────────────
pct start 200                # Arrancar contenedor
pct enter 200                # Entrar al contenedor
pct snapshot 200 pre-update  # Snapshot antes de cambios

# ─── NGINX ──────────────────────────────────────
nginx -t                     # Verificar configuración
systemctl reload nginx       # Aplicar cambios
tail -f /var/log/nginx/error.log  # Ver errores en tiempo real

# ─── DEPLOY ─────────────────────────────────────
scp -r ./web/* root@192.168.1.200:/var/www/html/
rsync -avz --delete ./web/ root@192.168.1.200:/var/www/html/

# ─── PERMISOS ───────────────────────────────────
chown -R www-data:www-data /var/www/
find /var/www -type d -exec chmod 755 {} \;
find /var/www -type f -exec chmod 644 {} \;
```

## 🔗 Recursos relacionados

|Recurso                          |Enlace                                                                          |
|---------------------------------|--------------------------------------------------------------------------------|
|Nginx — Documentación            |[nginx.org/en/docs](https://nginx.org/en/docs/)                                 |
|mkcert — HTTPS local sin warnings|[github.com/FiloSottile/mkcert](https://github.com/FiloSottile/mkcert)          |
|Proxmox VE — Wiki                |[pve.proxmox.com/wiki](https://pve.proxmox.com/wiki/Main_Page)                  |
|Pi-hole — DNS local              |[pi-hole.net](https://pi-hole.net/)                                             |
|AdGuard Home — DNS local         |[github.com/AdguardTeam/AdGuardHome](https://github.com/AdguardTeam/AdGuardHome)|

## 🤝 Contribuir

¿Encontraste un error o quieres mejorar la guía?

1. Fork del repositorio
1. Crea una rama: `git checkout -b fix/mi-mejora`
1. Commit: `git commit -m 'Añade X'`
1. Push: `git push origin fix/mi-mejora`
1. Abre un Pull Request

## 📜 Licencia

Distribuido bajo la licencia MIT (pendiente de añadir el archivo `LICENSE` al repositorio).

-----

<p align="center">
  Hecho con ❤️ para la comunidad de self-hosting en español
  <br>
  <sub>Si te ha sido útil, dale una ⭐ — ayuda a que otros la encuentren</sub>
</p>

-----

## English

An **interactive bilingual (ES/EN) web guide** — a single HTML file with an **ES / EN** switch (or `?lang=en`) — explaining step by step how to host static websites on a local Proxmox VE server using an LXC container with Nginx: container creation, Nginx, uploading sites (SCP, rsync, Git, ZIP), multi-site (subdirectories, ports, domains), local DNS, local HTTPS (mkcert), auto-deploy, security and troubleshooting.

**Requirements:** Proxmox VE 8.x or later (latest 8.x or 9.x recommended), 1 core, 64–256 MB RAM, 1–4 GB disk, Debian 12/13 or Alpine template.

**Quick start**

```bash
git clone https://github.com/ServerHeaven/guia-hosting-web-local-proxmox.git
cd guia-hosting-web-local-proxmox
open index.html   # macOS; on Linux: xdg-open index.html
```

**Notes added in the latest review:** template names change over time (use `pveam available`); `.local` is reserved for mDNS (prefer `.lan` or `home.arpa`); mkcert's root CA must be installed on every client device; GitHub webhooks cannot reach a private LAN IP (use the cron method); the `git pull` cron may need `safe.directory`; `add_header` directives are not inherited by a `location` that defines its own.

The English translations live in the `<script id="i18n-data">` block at the end of `index.html`, keyed by the original Spanish text.

Released under the MIT license (the `LICENSE` file is still to be added to the repository).
