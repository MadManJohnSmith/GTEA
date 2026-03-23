# GTEA — Deploy

## Setup local (WSL / Linux)

### 1. Clonar los tres repos en la misma carpeta padre
El contenido mas actual esta en las ramas dev_alan
```bash
mkdir ~/GTEA && cd ~/GTEA
git clone https://github.com/Guillermo1804/GTEA-Infra.git
git clone https://github.com/Guillermo1804/GTEA-Proyect-Api.git
git clone https://github.com/Guillermo1804/GTEA-Proyect-WebApp.git
```

Estructura esperada:

```text
~/GTEA/
├── GTEA-Infra/        ← entras aquí para todos los comandos
├── GTEA-Proyect-Api/
└── GTEA-Proyect-WebApp/
```

### 2. Configurar variables de entorno
```bash
cd ~/GTEA/GTEA-Infra
cp .env.example .env
# Editar .env con tus valores locales
```

### 3. Levantar local
```bash
docker compose build
docker compose up
```

Verificar en browser: http://localhost
