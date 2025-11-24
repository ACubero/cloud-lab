📘 DOCUMENTO 1: La Guía "Happy Path" (El Camino Ideal)
Si tuvieras que volver a hacerlo mañana desde cero, sin errores y en 5 minutos, estos serían los pasos exactos. Guárdate esto como tu "Cheat Sheet" (Chuleta) personal.

1. Aprovisionamiento y Acceso
Crear VPS: Hetzner Cloud, Ubuntu 24.04 (ARM64).

Firewall Cloud: Crear un firewall en el panel de Hetzner permitiendo entrada (Inbound) en puertos 22 (SSH), 80 (HTTP) y 9443 (HTTPS). Aplicarlo al servidor.

Acceso: Usar PowerShell o Terminal (no la consola web) para evitar errores de teclado: ssh root@TU_IP.

2. Preparación del Entorno
Actualizar el sistema y asegurarse de que Docker será compatible con Portainer (el fix de la API).

Bash

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
Instalar el orquestador con permisos suficientes para manejar el socket de Docker en Ubuntu 24.04.

Bash

docker volume create portainer_data

docker run -d -p 8000:8000 -p 9443:9443 \
  --name portainer \
  --restart=always \
  --privileged \
  -v /var/run/docker.sock:/var/run/docker.sock:Z \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
4. Despliegue de la Aplicación
Subida de archivos: Usar WinSCP (protocolo SFTP) para subir tu index.html a /home/usuario/web.

Despliegue: En Portainer, crear un Stack con este docker-compose:

YAML

version: '3'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - /home/usuario/web:/usr/share/nginx/html
    restart: always
