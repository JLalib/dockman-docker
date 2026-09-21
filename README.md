# 🐳 Dockman en Docker - Gestor de Docker Compose Autohospedado

[![GitHub](https://img.shields.io/badge/GitHub-RA341%2Fdockman-181717?logo=github)](https://github.com/RA341/dockman)
[![Docker](https://img.shields.io/badge/Docker-ghcr.io%2Fra341%2Fdockman-2496ED?logo=docker)](https://github.com/RA341/dockman/pkgs/container/dockman)
[![License](https://img.shields.io/badge/License-AGPL--3.0-orange)](https://github.com/RA341/dockman/blob/main/LICENSE)

## 📋 Descripción general

**Dockman** es una interfaz web moderna y accesible para gestionar `docker-compose` de forma visual, diseñada específicamente para **homelabs** que quieren control total sobre sus archivos `.yaml` sin perder la comodidad de una UI intuitiva.

A diferencia de Portainer (que abstrae Compose), Dockman mantiene tus archivos Compose como **fuente de verdad** mientras proporciona gestión visual completa: editor YAML, control de contenedores, imágenes, volúmenes, networks, multi-host vía SSH, búsqueda instantánea y atajos de teclado.

> **Propuesta clave**: Acceso directo a archivos `docker-compose` en tu filesystem. Editor visual YAML (no oculta nada). Gestión de stacks por carpetas o archivos individuales. Multi-host support via SSH.

## ✨ Características principales

- **Gestor visual de docker-compose files** — Interfaz split-panel (file browser + editor)
- **Smart compose detection** — Promociona compose files únicos a nivel top; folders con múltiples files como directorios
- **Editor YAML con formatting** — `Alt+L` para auto-format YAML, edita sin salir de la UI
- **Control total de contenedores** — Start, stop, restart, logs (streaming), shell interactivo, rm
- **Gestión de imágenes** — Pull, remove, tags, search, inspect
- **Gestión de volúmenes y networks** — Crear, eliminar, inspeccionar
- **Multi-host vía SSH** — Gestiona múltiples servidores desde una UI central
- **Configuración centralizada** — `.dockman.yml` con aliases para directorios externos
- **Persistencia en BD SQLite** — Hosts, SSH keys, settings guardados localmente
- **Dark mode + Responsive** — UI moderna React + TypeScript, mobile-friendly
- **Búsqueda instantánea + atajos** — Encuentra containers/imágenes al instante, navegación rápida por teclado
- **AGPL-3.0 open source** — Código accesible, comunidad activa, desarrollo continuo

## 📋 Requisitos del sistema

- **Docker** instalado y funcionando
- **Docker Socket** (`/var/run/docker.sock`) accesible
- **100 MB - 500 MB RAM** (ligero, NodeJS)
- **100 MB espacio disco** (imagen + BD config)
- **Puerto 8866** (o custom, configurable)
- **Volumen persistente** para `/config` (BD, SSH keys, settings)
- **Variable `DOCKMAN_COMPOSE_ROOT`** (ruta absoluta a compose files)
- **Navegador moderno** (Chrome, Firefox, Safari, Edge)

> ⚠️ **Importante**: La ruta `COMPOSE_ROOT` debe ser **ABSOLUTA e idéntica** en: variable de entorno + volumen host + volumen container.

## 🐳 Instalación

### Paso 1: Preparar estructura de directorios

```bash
mkdir -p /ruta/absoluta/stacks
mkdir -p /ruta/absoluta/dockman/config
# /ruta/absoluta/stacks contendrá tus docker-compose.yml files
```

### Paso 2: Docker Compose (recomendado)

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  dockman:
    container_name: dockman
    image: ghcr.io/ra341/dockman:latest
    restart: always
    ports:
      - "8866:8866"
    environment:
      # 1️⃣ Ruta absoluta a tus compose files
      - DOCKMAN_COMPOSE_ROOT=/ruta/absoluta/stacks
      # Desactivar warning auth (si no usas auth)
      - DOCKMAN_LOG_AUTH_WARNING=false
    volumes:
      # 2️⃣ Host side y 3️⃣ Container side DEBEN SER IDÉNTICAS
      - /ruta/absoluta/stacks:/ruta/absoluta/stacks
      # Config dir (BD, SSH keys, etc)
      - /ruta/absoluta/dockman/config:/config
      # Docker socket para gestionar containers
      - /var/run/docker.sock:/var/run/docker.sock
EOF

docker compose up -d
```

### Paso 3: Docker run simple

```bash
docker run -d \
  --name dockman \
  --restart always \
  -p 8866:8866 \
  -e DOCKMAN_COMPOSE_ROOT=/ruta/absoluta/stacks \
  -e DOCKMAN_LOG_AUTH_WARNING=false \
  -v /ruta/absoluta/stacks:/ruta/absoluta/stacks \
  -v /ruta/absoluta/dockman/config:/config \
  -v /var/run/docker.sock:/var/run/docker.sock \
  ghcr.io/ra341/dockman:latest
```

### Acceder y verificar

```bash
# Acceder a la UI
http://localhost:8866

# Verificar estado
docker logs dockman
docker ps | grep dockman
```

## ⚙️ Configuración

1. **`DOCKMAN_COMPOSE_ROOT`** — Ruta absoluta a tus archivos compose (ej: `/opt/stacks`)
2. **`DOCKMAN_LOG_AUTH_WARNING`** — `false` para desactivar warning de auth si no usas autenticación
3. **Volumen compose** — Host y container path **idénticos** (requisito crítico)
4. **Volumen config** — Persistencia BD SQLite, SSH keys, settings (`/config`)
5. **Docker socket** — `/var/run/docker.sock:/var/run/docker.sock` para gestión de containers
6. **Puerto** — `8866:8866` (configurable en `ports`)
7. **Restart policy** — `always` para producción

## 🚀 Primeros pasos

1. **Acceder por primera vez** — Abre `http://localhost:8866`; verás dashboard con file browser (izquierda) y editor (derecha); auto-detecta compose files desde `DOCKMAN_COMPOSE_ROOT`
2. **Crear primer stack** — File browser → `+ Add File` → Nombre: `docker-compose.yml` → Editor abre con template → Copia-pega tu YAML → Auto-save
3. **Gestionar contenedores** — Tab "Containers" → Ver todos los containers → Click container → Opciones: logs, shell, restart, stop, rm
4. **Ver logs en tiempo real** — Containers → Click container → "Logs" → Terminal con streaming, scroll automático, búsqueda
5. **Shell interactivo** — Containers → Click container → "Shell" → Terminal interactiva, ejecuta comandos, cierra al terminar
6. **Gestionar imágenes** — Tab "Images" → Ver imágenes locales → "Pull" para descargar → Remove, inspect, tags
7. **Editor YAML con format** — File browser → Click compose file → Editor → `Alt+L` para auto-format YAML
8. **Búsqueda rápida** — Search bar arriba → Escribe nombre container/imagen → Resultados instantáneos + atajos teclado
9. **Agregar host remoto (SSH multi-host)** — Settings (engranaje) → "Hosts" → "Add Host" → Nombre, IP, usuario SSH, puerto (22) → Copia/ genera SSH key
10. **Gestión volúmenes/networks** — Tabs "Volumes" / "Networks" → Ver todos → Crear, remove, inspect
11. **Cambiar tema (Dark mode)** — Settings → Theme → Toggle Dark/Light → Preferencia guardada en BD local

## 💡 Casos de uso

- **Homelabs (self-hosted)** — Gestor visual para tu stack de servicios, control total, zero cloud
- **DevOps engineers** — Manage múltiples servidores vía SSH, compose files como source of truth
- **Startups/pequeños equipos** — Dashboard compartida, sin complejidad de Kubernetes, Compose suficiente
- **Learning Docker** — Aprende Compose visualmente, entiende YAML sin abstracciones
- **Migración desde Portainer** — Si quieres acceso directo a Compose files pero UI moderna

## 🔒 Acceso remoto seguro

### HTTPS con Caddy (producción)

```caddyfile
docker.tudominio.com {
    reverse_proxy localhost:8866
}
```

Acceso: `https://docker.tudominio.com` con HTTPS automático.

> ⚠️ **IMPORTANTE**: Dockman **no incluye auth por defecto**. Usar Caddy con `basic_auth` o HTTPS + firewall si expones a red pública.

## 🛠️ Gestión y mantenimiento

```bash
# Ver logs en tiempo real
docker logs -f dockman

# Backup de configuración (BD, SSH keys, settings)
cp -r /ruta/absoluta/dockman/config ./dockman-config-backup-$(date +%Y%m%d)

# Backup de compose files
cp -r /ruta/absoluta/stacks ./stacks-backup-$(date +%Y%m%d)

# Restore configuración
docker stop dockman
rm -rf /ruta/absoluta/dockman/config
cp -r ./dockman-config-backup-YYYYMMDD /ruta/absoluta/dockman/config
docker start dockman

# Reiniciar container
docker compose restart dockman

# Actualizar a versión más reciente
docker compose pull
docker compose up -d

# Monitorear consumo
docker stats dockman
# Típicamente: 100-300MB RAM, bajo CPU

# Verificar COMPOSE_ROOT setup
docker exec dockman sh -c 'echo $DOCKMAN_COMPOSE_ROOT'
# Debe mostrar tu ruta absoluta configurada
```

## 📝 Licencia

**AGPL-3.0** — Código abierto, uso libre, modificaciones deben compartirse bajo misma licencia.

[Ver licencia completa](https://github.com/RA341/dockman/blob/main/LICENSE)

---

> 📖 **Guía completa**: [Cómo instalar Dockman en Docker - Gestor de Docker Compose autohospedado en Docker](https://genbyte.blogspot.com/2026/08/como-instalar-dockman-gestor-de-docker.html)