# 🐳 Dockman Docker - Gestor visual de Docker Compose autohospedado

[![GitHub](https://img.shields.io/badge/GitHub-RA341%2Fdockman-181717?logo=github)](https://github.com/RA341/dockman)
[![Docker](https://img.shields.io/badge/Docker-ghcr.io%2Fra341%2Fdockman-2496ED?logo=docker)](https://ghcr.io/ra341/dockman)
[![License](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](https://www.gnu.org/licenses/agpl-3.0.html)

## 📋 Descripción general

Dockman es una interfaz web moderna y accesible para gestionar **docker-compose** de forma visual, diseñada específicamente para **homelabs** que quieren control total sobre sus archivos `.yaml` sin perder la comodidad de una UI intuitiva. A diferencia de Portainer (que abstrae Compose), Dockman mantiene tus archivos Compose como **fuente de verdad** mientras proporciona gestión visual completa.

## ✨ Características principales

- **Gestor visual de docker-compose files** – Editor YAML integrado con formatting (Alt+L)
- **Smart compose detection** – Promociona compose files únicos a nivel top; carpetas con múltiples files como directorios
- **Control total de contenedores** – Start, stop, restart, logs, shell, rm desde la UI
- **Gestión de imágenes** – Pull, remove, tags, search
- **Gestión de volúmenes y networks** – Crear, eliminar, inspeccionar
- **Multi-host vía SSH** – Gestiona múltiples servidores desde una UI central
- **Búsqueda instantánea + atajos teclado** – Encuentra containers/imágenes al momento
- **Dark mode + Responsive** – UI moderna (React + TypeScript) para desktop, tablet, móvil
- **Configuración centralizada** – `.dockman.yml` con aliases para directorios externos
- **Persistencia en BD SQLite** – Hosts, SSH keys, settings guardados localmente
- **AGPL-3.0 open source** – Código accesible, comunidad activa, desarrollo continuo

## 📋 Requisitos del sistema

- Docker instalado y funcionando
- **Docker Socket** (`/var/run/docker.sock`) accesible
- **100–500 MB RAM** (ligero, NodeJS)
- **100 MB espacio disco** (imagen + BD config)
- Puerto **8866** (o personalizable)
- Volumen persistente para `/config` (BD, SSH keys, settings)
- Variable de entorno `DOCKMAN_COMPOSE_ROOT` (ruta absoluta a tus compose files)
- Navegador moderno (Chrome, Firefox, Safari, Edge)

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

### Acceder

- **Dashboard**: http://localhost:8866
- **Verificar estado**:
  ```bash
  docker logs dockman
  docker ps | grep dockman
  ```

## ⚙️ Configuración

1. **`DOCKMAN_COMPOSE_ROOT`** – Ruta absoluta a la carpeta que contiene tus archivos `docker-compose.yml` (ej: `/ruta/absoluta/stacks`). **Debe coincidir exactamente** con el bind mount host↔container.
2. **`DOCKMAN_LOG_AUTH_WARNING`** – `false` para desactivar el aviso de autenticación si no usas auth.
3. **Volumen `/config`** – Persistencia de BD SQLite, SSH keys y settings (`/ruta/absoluta/dockman/config:/config`).
4. **Docker socket** – `/var/run/docker.sock:/var/run/docker.sock` para gestión de contenedores.
5. **Puerto** – `8866:8866` (modificable en `ports`).

## 🚀 Primeros pasos

1. **Acceder por primera vez** – Abre `http://localhost:8866`. Verás el dashboard con file browser a la izquierda y editor a la derecha; auto-detecta tus compose files desde `DOCKMAN_COMPOSE_ROOT`.
2. **Crear primer stack** – File browser → “+ Add File” → Nombre: `docker-compose.yml` → Editor abre con template vacío → Copia-pega tu YAML → Auto-save.
3. **Gestionar contenedores** – Pestaña “Containers” → Verás todos los containers corriendo → Click en uno → Opciones: logs, shell, restart, stop, rm.
4. **Ver logs en tiempo real** – Containers → Click container → “Logs” → Terminal con streaming, scroll automático, búsqueda rápida.
5. **Shell interactivo** – Containers → Click container → “Shell” → Terminal interactiva, ejecuta comandos, cierra al terminar.
6. **Gestionar imágenes** – Pestaña “Images” → Ver imágenes locales → “Pull” para descargar nuevas → Remove, inspect, tags.
7. **Editor YAML con formato** – File browser → Click compose file → Editor abre → **Alt+L** para auto-format YAML.
8. **Búsqueda rápida** – Barra superior → Escribe nombre de container o imagen → Resultados instantáneos + atajos teclado.
9. **Agregar host remoto (SSH multi-host)** – Settings (engranaje) → “Hosts” → “Add Host” → Nombre, IP, usuario SSH, puerto (22 default) → Copia/ genera SSH key → Dockman gestiona múltiples servidores desde UI central.
10. **Gestión volúmenes/networks** – Pestañas “Volumes” / “Networks” → Ver todos, crear nuevo, remove, inspect.
11. **Cambiar tema (Dark mode)** – Settings → Theme → Toggle Dark/Light → Preferencia guardada en BD local.

## 💡 Casos de uso

- **Homelabs (self-hosted)** – Gestor visual para tu stack de servicios, control total, zero cloud.
- **DevOps engineers** – Manage múltiples servidores vía SSH, Compose files como source of truth.
- **Startups/pequeños equipos** – Dashboard compartida, sin complejidad de Kubernetes, Compose suficiente.
- **Learning Docker** – Aprende Compose visualmente, entiende YAML sin abstracciones.
- **Migración desde Portainer** – Si quieres acceso directo a Compose files pero UI moderna.

## 🔒 Acceso remoto seguro

```text
# Caddyfile
docker.tudominio.com {
    reverse_proxy localhost:8866
}
```

- Acceso: `https://docker.tudominio.com` con HTTPS automático (Let’s Encrypt).
- **IMPORTANTE**: Dockman **no incluye auth por defecto**. Usa Caddy con `basic_auth` o HTTPS + firewall si expones a red pública.

## 🛠️ Gestión y mantenimiento

```bash
# Ver logs en tiempo real
docker logs -f dockman

# Backup de configuración (BD SQLite, SSH keys, settings)
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
# Típicamente: 100-300 MB RAM, bajo CPU

# Verificar COMPOSE_ROOT setup
docker exec dockman sh -c 'echo $DOCKMAN_COMPOSE_ROOT'
# Debe mostrar tu ruta absoluta configurada
```

## 📝 Licencia

AGPL-3.0 – Código abierto, comunidad activa, desarrollo continuo.

> ✨ **Nota**: Este repositorio contiene la configuración Docker y documentación
> extraída del tutorial de Genbyte: <a href="https://genbyte.blogspot.com/2026/08/como-instalar-dockman-gestor-de-docker.html" target="_blank" rel="noopener noreferrer">Cómo instalar Dockman en Docker - Gestor de Docker Compose autohospedado en Docker</a>