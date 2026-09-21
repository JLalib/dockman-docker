# 🐳 Dockman - Gestor de Docker Compose Autohospedado

[![GitHub Stars](https://img.shields.io/github/stars/RA341/dockman?style=for-the-badge&logo=github)](https://github.com/RA341/dockman)
[![Docker Pulls](https://img.shields.io/docker/pulls/ra341/dockman?style=for-the-badge&logo=docker)](https://hub.docker.com/r/ra341/dockman)
[![License](https://img.shields.io/github/license/RA341/dockman?style=for-the-badge)](https://github.com/RA341/dockman/blob/main/LICENSE)
[![GitHub Release](https://img.shields.io/github/v/release/RA341/dockman?style=for-the-badge&logo=github)](https://github.com/RA341/dockman/releases)

## 📋 Descripción general

**Dockman** es una interfaz web moderna y accesible para gestionar `docker-compose` de forma visual, diseñada específicamente para **homelabs** que quieren control total sobre sus archivos `.yaml` sin perder la comodidad de una UI intuitiva.

A diferencia de Portainer (que abstrae Compose), Dockman mantiene tus archivos Compose como **fuente de verdad** mientras proporciona gestión visual completa. Está construido con **TypeScript + React**, es **open source (AGPL-3.0)** y ofrece soporte **multi-host vía SSH** para gestionar múltiples servidores desde una UI central.

> 📖 **Artículo original**: [Cómo instalar Dockman en Docker - Gestor de Docker Compose autohospedado en Docker](https://genbyte.blogspot.com/2026/08/como-instalar-dockman-gestor-de-docker.html)

## ✨ Características principales

- 🎨 **Gestor visual de docker-compose files** - Editor YAML integrado con formatting (Alt+L)
- 🔍 **Smart compose detection** - Promociona compose files únicos a nivel top; folders con múltiples files como directorios
- 📦 **Control total de contenedores** - Start, stop, restart, logs, shell, rm desde la UI
- 🖼️ **Gestión de imágenes** - Pull, remove, tags, search, inspect
- 💾 **Gestión de volúmenes y networks** - Crear, eliminar, inspeccionar
- 🌐 **Multi-host vía SSH** - Gestiona múltiples servidores desde UI central
- ⚡ **Búsqueda instantánea + atajos teclado** - Encuentra containers/imágenes al instante
- 🌙 **Dark mode + Responsive** - UI moderna, desktop, tablet, móvil
- ⚙️ **Configuración centralizada** - `.dockman.yml` para aliases, hosts, SSH keys
- 🗄️ **Persistencia en BD SQLite** - Configuración, hosts, claves guardadas localmente
- 🔓 **AGPL-3.0 open source** - Código accesible, comunidad activa, desarrollo continuo

## 📋 Requisitos del sistema

- ✅ **Docker** instalado y funcionando
- ✅ **Docker Socket** (`/var/run/docker.sock`) accesible
- ✅ **100 MB - 500 MB RAM** (ligero, NodeJS)
- ✅ **100 MB espacio disco** (imagen + BD config)
- ✅ **Puerto 8866** (o personalizable)
- ✅ **Volumen persistente** para `/config` (BD, SSH keys, settings)
- ✅ **Variable `DOCKMAN_COMPOSE_ROOT`** (ruta absoluta a compose files)
- ✅ **Navegador moderno** (Chrome, Firefox, Safari, Edge)

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

1. **DOCKMAN_COMPOSE_ROOT** - Ruta absoluta a tus archivos docker-compose (obligatoria)
2. **DOCKMAN_LOG_AUTH_WARNING** - `false` para desactivar warning de autenticación
3. **Volumen stacks** - Host y container path **idénticos** (ej: `/ruta/absoluta/stacks:/ruta/absoluta/stacks`)
4. **Volumen config** - Persistencia BD SQLite, SSH keys, settings (`/ruta/absoluta/dockman/config:/config`)
5. **Docker socket** - `/var/run/docker.sock:/var/run/docker.sock` para gestión de contenedores
6. **Puerto** - `8866:8866` (personalizable)
7. **Restart policy** - `always` para producción

## 🚀 Primeros pasos

1. **Acceder por primera vez** - Abre `http://localhost:8866`, verás dashboard con file browser (izquierda) y editor (derecha)
2. **Crear primer stack** - File browser → "+ Add File" → Nombre: `docker-compose.yml` → Pega tu YAML → Auto-save
3. **Gestionar contenedores** - Tab "Containers" → Click container → Logs, shell, restart, stop, rm
4. **Ver logs en tiempo real** - Containers → Click container → "Logs" → Terminal con streaming
5. **Shell interactivo** - Containers → Click container → "Shell" → Ejecuta comandos directamente
6. **Gestionar imágenes** - Tab "Images" → Ver locales, "Pull" para nuevas, remove, inspect, tags
7. **Editor YAML con format** - Click compose file → Editor abre → **Alt+L** para auto-format YAML
8. **Búsqueda rápida** - Search bar arriba → Escribe nombre container/imagen → Resultados instantáneos
9. **Agregar host remoto (SSH)** - Settings (engranaje) → "Hosts" → "Add Host" → Nombre, IP, usuario, puerto, SSH key
10. **Gestión volúmenes/networks** - Tabs "Volumes" / "Networks" → Crear, eliminar, inspeccionar
11. **Cambiar tema** - Settings → Theme → Toggle Dark/Light (guardado en BD local)

## 💡 Casos de uso

- 🏠 **Homelabs (self-hosted)** - Gestor visual para tu stack de servicios, control total, zero cloud
- 👨‍💻 **DevOps engineers** - Maneja múltiples servidores vía SSH, Compose files como source of truth
- 🚀 **Startups/pequeños equipos** - Dashboard compartida, sin complejidad de Kubernetes
- 📚 **Learning Docker** - Aprende Compose visualmente, entiende YAML sin abstracciones
- 🔄 **Migración desde Portainer** - Si quieres acceso directo a Compose files pero UI moderna

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

### Ver logs
```bash
docker logs -f dockman
```

### Backup de configuración
```bash
cp -r /ruta/absoluta/dockman/config ./dockman-config-backup-$(date +%Y%m%d)
# Contiene: BD SQLite, SSH keys, settings
```

### Backup de compose files
```bash
cp -r /ruta/absoluta/stacks ./stacks-backup-$(date +%Y%m%d)
```

### Restore configuración
```bash
docker stop dockman
rm -rf /ruta/absoluta/dockman/config
cp -r ./dockman-config-backup-YYYYMMDD /ruta/absoluta/dockman/config
docker start dockman
```

### Reiniciar container
```bash
docker compose restart dockman
```

### Actualizar a versión más reciente
```bash
docker compose pull
docker compose up -d
```

### Monitorear consumo
```bash
docker stats dockman
# Típicamente: 100-300MB RAM, bajo CPU
```

### Verificar COMPOSE_ROOT setup
```bash
docker exec dockman sh -c 'echo $DOCKMAN_COMPOSE_ROOT'
# Debe mostrar tu ruta absoluta configurada
```

## 📝 Licencia

**AGPL-3.0** - Código abierto, desarrollo activo, comunidad en GitHub.

## 📌 Nota final

> Documentación generada basada en el artículo: **[Cómo instalar Dockman en Docker - Gestor de Docker Compose autohospedado en Docker](https://genbyte.blogspot.com/2026/08/como-instalar-dockman-gestor-de-docker.html)**

---

**Referencias oficiales:**
- 📦 [GitHub Repository - RA341/dockman](https://github.com/RA341/dockman)
- 🌐 [Official Website - dockman.radn.dev](https://dockman.radn.dev)
- 📚 [Official Documentation](https://github.com/RA341/dockman#readme)
- 🐳 [Docker Hub - ra341/dockman](https://hub.docker.com/r/ra341/dockman)
- 📦 [GitHub Container Registry - ghcr.io/ra341/dockman](https://github.com/RA341/dockman/pkgs/container/dockman)