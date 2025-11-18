# AutoYa-Deployment
Este documento describe cómo desplegar todo el entorno de Autoya utilizando Docker, Nginx, MinIO, Redis, PostgreSQL y el pipeline automatizado en Jenkins.

---

Deployment — Autoya (Backend + Frontend + Infraestructura)
## 1. Requisitos previos: 
Antes de desplegar, asegúrate de tener instalado:
- Docker ≥ 20.x
- Docker Compose ≥ v2.x
- Jenkins (si usarás CI/CD)
- Git
- Acceso al servidor donde se hará el deploy

## 2. Variables de entorno
El deploy usa el archivo:
- .env.dev

Contiene:
- Config DB (Postgres)
- MinIO (almacenamiento de imágenes)
- Redis
- URL pública del bucket
- API URL para el frontend

---

## 3. Servicios que se despliegan
- **Backend**: Construido desde /backend y expuesto por Docker.
- **Frontend**: Construido desde /frontend y servido con Nginx.
- **PostgreSQL**: Base de datos persistente.
- **Redis**: Cache y colas.
- **MinIO**: Almacenamiento S3-compatible para imágenes de vehículos.
- **MinIO Setup**: Crea el bucket automáticamente según el branch -> autoya-dev

---

## 4. Nginx — Reverse Proxy + SPA Router
Ubicado en: docker/nginx.conf

Incluye:
- Server estático del frontend
- Proxy hacia el backend (/api)
- Proxy websockets (/ws)
- SPA routing (try_files)

---

## 5. Deployment manual (sin Jenkins)
1. Bajar contenedores previos
docker compose -f docker-compose.dev.yml --env-file .env.dev down -v
2. Construir todo
docker compose -f docker-compose.dev.yml --env-file .env.dev build
3. Levantar entorno completo
docker compose -f docker-compose.dev.yml --env-file .env.dev up -d
4. Ver logs
- docker logs autoya-backend -f
- docker logs autoya-frontend -f

---

## 6. Puertos del Deployment
Servicio	Puerto:
- Frontend	10081 → 80
- Backend	10080 → 8080
- PostgreSQL	10543 → 5432
- Redis	10679 → 6379
- MinIO API	10090 → 9000
- MinIO Console	10091 → 9001

## Enlaces del proyecto
- **Frontend AutoYa**: https://github.com/AlvaroV19/AutoYa-Frontend
- **Backend AutoYa**: https://github.com/AlvaroV19/AutoYa-Backend
- **Docs AutoYa**: https://github.com/AlvaroV19/AutoYa-Docs
