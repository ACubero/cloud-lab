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
