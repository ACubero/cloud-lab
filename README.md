# 🚀 Despliegue de Infraestructura Cloud & Container Orchestration

Este repositorio documenta el desafío técnico de desplegar un entorno de producción para una aplicación web estática, gestionando todo el ciclo de vida de la infraestructura: desde el aprovisionamiento del servidor hasta la orquestación de contenedores y la resolución avanzada de problemas.

![Estado del Proyecto](https://img.shields.io/badge/Estado-Completado-success)
![Docker](https://img.shields.io/badge/Docker-v29-blue)
![Portainer](https://img.shields.io/badge/Portainer-CE-orange)

## 🛠️ Arquitectura y Tecnologías

* **Infraestructura:** VPS en Hetzner Cloud (Arquitectura ARM64 / Ubuntu 24.04 LTS).
* **Orquestación:** Portainer CE gestionando Docker Engine.
* **Servidor Web:** Nginx (Alpine Linux) contenerizado.
* **Seguridad:** Estrategia de defensa en profundidad (Cloud Firewall + UFW + SSH Hardening).
* **Despliegue:** Docker Compose (Stacks) y gestión de volúmenes persistentes (Bind Mounts).

## 💡 Habilidades Demostradas

### 1. Administración de Sistemas Linux (SysAdmin)
* Gestión de usuarios y permisos (Root vs Sudoers).
* Diagnóstico de servicios con `systemctl` (reparación de servicio SSH inactivo).
* Configuración de red y diagnóstico de conectividad (`ip addr`, `Test-NetConnection`, `ufw`).
* Gestión de seguridad SSH (`sshd_config`, `PermitRootLogin`, gestión de llaves).

### 2. Docker & Containerización
* Instalación y configuración del Docker Daemon en Linux.
* **Solución de problemas complejos:** Resolución de incompatibilidad entre la API de Docker v29 y Portainer mediante configuración personalizada del `daemon.json` (`min-api-version`).
* Manejo de contextos de seguridad en contenedores (`--privileged`, etiquetas SELinux `:Z`).
* Creación y gestión de Stacks mediante `docker-compose.yml`.

### 3. Redes y Seguridad
* Implementación de Firewalls en capas (Nivel Proveedor y Nivel SO).
* Gestión de puertos (22 SSH, 80 HTTP, 9443 HTTPS).
* Transferencia segura de archivos mediante SFTP (WinSCP).

## 🐛 "War Story": El desafío de la API
Durante la implementación, el entorno de Portainer fallaba al conectar con el socket de Docker local. Tras analizar los logs del contenedor, identifiqué un *breaking change* en la versión 29 de Docker Engine.

**Solución implementada:**
Modificación del `daemon.json` para mantener retrocompatibilidad:
```json
{
  "min-api-version": "1.24"
}



📘 Historia 1: La Guía del Éxito (El Camino Ideal)
Si tuviera que repetir este despliegue mañana en 5 minutos, estos son los pasos exactos sin dar vueltas:

1. Aprovisionamiento y Acceso
Crear VPS: Usar Hetzner Cloud con imagen Ubuntu 24.04 (Arquitectura ARM64).

Firewall Cloud: Antes de nada, crear un firewall en el panel web permitiendo entrada (Inbound) en los puertos 22 (SSH), 80 (HTTP) y 9443 (HTTPS) y, lo más importante, aplicarlo al servidor.

Acceso: Conectar siempre vía PowerShell o Terminal (ssh root@TU_IP) para evitar problemas de codificación de teclado de la consola web.

2. Preparación del Entorno
Actualizar el sistema y aplicar el parche preventivo para la compatibilidad de Docker:
# Actualizar sistema
apt update && apt upgrade -y

# Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# CONFIGURACIÓN CRÍTICA (Parche compatibilidad Docker 29)
sudo tee /etc/docker/daemon.json <<EOF
{
  "min-api-version": "1.24"
}
EOF

# Reiniciar Docker para aplicar cambios
systemctl restart docker

3. Despliegue de Portainer
Instalar el orquestador con permisos suficientes para manejar el socket de Docker en Ubuntu 24.04:
docker volume create portainer_data

docker run -d -p 8000:8000 -p 9443:9443 \
  --name portainer \
  --restart=always \
  --privileged \
  -v /var/run/docker.sock:/var/run/docker.sock:Z \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

4. Despliegue de la Aplicación
Subida de archivos: Usar WinSCP (protocolo SFTP) para subir el index.html a la carpeta del usuario (/home/usuario/web).

Despliegue: En Portainer, crear un Stack apuntando a esa ruta:
docker volume create portainer_data

docker run -d -p 8000:8000 -p 9443:9443 \
  --name portainer \
  --restart=always \
  --privileged \
  -v /var/run/docker.sock:/var/run/docker.sock:Z \
  -v portainer_data:/data \
  portainer/portainer-ce:latest


📙 Historia 2: Diario de Guerra (Troubleshooting)
Esta es la bitácora técnica de los obstáculos encontrados y cómo se solucionó cada uno mediante ingeniería y diagnóstico.
Problema,Síntoma,Solución Técnica Aplicada
Named Pipes en Windows,Docker Desktop en Windows fallaba al conectar con Portainer usando rutas de Linux (/var/run/docker.sock).,Migración a Linux: Se decidió mover la infraestructura a un VPS nativo Linux para evitar capas de emulación y problemas de sockets propietarios.
Inyección de caracteres,"Al pegar comandos en la consola VNC del navegador, los caracteres : y / se cambiaban por ñ o -.",Cambio a SSH: Se configuró el acceso remoto vía SSH (PowerShell) para utilizar la codificación de teclado local correcta.
Bloqueo de Red (Capa 1),Test-NetConnection fallaba (Timeout) en el puerto 22.,Hetzner Firewall: Se detectó que el firewall de nube no estaba aplicado al recurso (servidor). Se vinculó correctamente el firewall al VPS.
Bloqueo de Red (Capa 2),WinSCP rechazaba la conexión tras abrir el firewall de nube.,UFW (Linux Firewall): El firewall interno del SO bloqueaba SSH. Se ejecutó ufw allow 22/tcp desde la consola de emergencia.
Servicio SSH Muerto,El puerto estaba abierto pero nadie respondía (Connection refused).,Systemd: Se diagnosticó el servicio como inactive (dead). Se forzó el arranque con systemctl start ssh y se habilitó PermitRootLogin en sshd_config.
AppArmor / Socket,"Portainer mostraba el entorno local como ""Down"" o ""Unreachable"" a pesar de estar instalado.",Privilegios: Se reinstaló el contenedor con las banderas --privileged y el sufijo :Z en el volumen del socket para saltar las restricciones de SELinux/AppArmor.
Incompatibilidad API,Error Failed loading environment persistente en arquitectura ARM64.,Downgrade API: Se identificó que Docker v29 rompe compatibilidad con Portainer v2.21. Se forzó min-api-version: 1.24 en /etc/docker/daemon.json.
