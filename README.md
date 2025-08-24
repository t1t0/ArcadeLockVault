# ArcadeLockVault(API

ArcadeLockVault es una aplicación self-hosted diseñada para organizar y gestionar contraseñas de sitios web de equipos en un entorno seguro y amigable. Permite a los equipos almacenar, compartir y administrar credenciales de manera centralizada con altos estándares de seguridad.

## Características

- 🔐 Gestión segura de contraseñas para equipos
- 🏠 Self-hosted: control total sobre tus datos
- 👥 Colaboración en equipo
- 🔒 Cifrado de extremo a extremo
- 🌐 Interfaz web intuitiva y amigable
- 🚀 Construido con AdonisJS y TypeScript

## Requisitos del Sistema

- Node.js (versión 18 o superior)
- npm o yarn
- Base de datos (SQLite para desarrollo, PostgreSQL/MySQL para producción)

## Instalación y Configuración para Desarrollo

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd ArcadeLockVault
```

### 2. Instalar dependencias

```bash
npm install
```

### 3. Configurar variables de entorno

Copia el archivo de ejemplo y configura las variables necesarias:

```bash
cp .env.example .env
```

Edita el archivo `.env` con tus configuraciones:

```env
# Configuración de la aplicación
TZ=UTC
PORT=3333
HOST=localhost
LOG_LEVEL=info
APP_KEY=your-secret-app-key-here
NODE_ENV=development

# Configuración de base de datos
DB_CONNECTION=sqlite
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASSWORD=
DB_DATABASE=arcade_lock_vault

# Configuración de sesión
SESSION_DRIVER=cookie
```

### 4. Generar clave de aplicación

```bash
node ace generate:key
```

### 5. Ejecutar migraciones

```bash
node ace migration:run
```

### 6. Iniciar el servidor de desarrollo

```bash
npm run dev
```

La aplicación estará disponible en `http://localhost:3333`

## Comandos de Desarrollo

```bash
# Iniciar servidor de desarrollo con hot reload
npm run dev

# Ejecutar tests
npm test

# Linting
npm run lint

# Formatear código
npm run format

# Construir para producción
npm run build

# Ejecutar migraciones
node ace migration:run

# Revertir migraciones
node ace migration:rollback

# Crear nueva migración
node ace make:migration nombre_de_la_migracion

# Crear nuevo modelo
node ace make:model NombreDelModelo

# Crear nuevo controlador
node ace make:controller NombreDelControlador
```

## Configuración para Producción

### 1. Variables de entorno de producción

Configura las siguientes variables en tu servidor de producción:

```env
# Configuración de la aplicación
TZ=UTC
PORT=3333
HOST=0.0.0.0
LOG_LEVEL=info
APP_KEY=your-production-secret-key-here
NODE_ENV=production

# Configuración de base de datos (PostgreSQL recomendado)
DB_CONNECTION=pg
DB_HOST=your-db-host
DB_PORT=5432
DB_USER=your-db-user
DB_PASSWORD=your-secure-db-password
DB_DATABASE=arcade_lock_vault_prod

# Configuración de sesión
SESSION_DRIVER=cookie

# Configuraciones adicionales de seguridad
SECURE_COOKIES=true
SAME_SITE=strict
```

### 2. Preparar la aplicación

```bash
# Instalar dependencias de producción
npm ci --only=production

# Construir la aplicación
npm run build

# Ejecutar migraciones en producción
node ace migration:run --force
```

### 3. Configuración del servidor web

#### Nginx (Recomendado)

Crea un archivo de configuración para Nginx:

```nginx
server {
    listen 80;
    server_name tu-dominio.com;
    
    # Redirigir HTTP a HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name tu-dominio.com;
    
    # Configuración SSL
    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;
    
    # Configuraciones de seguridad SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    location / {
        proxy_pass http://localhost:3333;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### 4. Configuración de Process Manager (PM2)

Instala PM2 globalmente:

```bash
npm install -g pm2
```

Crea un archivo `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [{
    name: 'arcade-lock-vault',
    script: './build/bin/server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3333
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true
  }]
}
```

Inicia la aplicación con PM2:

```bash
# Iniciar la aplicación
pm2 start ecosystem.config.js

# Guardar configuración de PM2
pm2 save

# Configurar PM2 para iniciarse al arrancar el sistema
pm2 startup
```

### 5. Configuración de base de datos

#### PostgreSQL (Recomendado para producción)

```sql
-- Crear base de datos y usuario
CREATE DATABASE arcade_lock_vault_prod;
CREATE USER arcade_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE arcade_lock_vault_prod TO arcade_user;
```

### 6. Backup y Monitoreo

#### Script de backup automático

```bash
#!/bin/bash
# backup.sh
DATE=$(date +"%Y%m%d_%H%M%S")
BACKUP_DIR="/path/to/backups"
DB_NAME="arcade_lock_vault_prod"
DB_USER="arcade_user"

# Crear backup de la base de datos
pg_dump -U $DB_USER -h localhost $DB_NAME > $BACKUP_DIR/backup_$DATE.sql

# Mantener solo los últimos 7 backups
find $BACKUP_DIR -name "backup_*.sql" -mtime +7 -delete
```

Configura un cron job para ejecutar backups automáticos:

```bash
# Ejecutar backup diario a las 2:00 AM
0 2 * * * /path/to/backup.sh
```

## Seguridad

- ✅ Usa HTTPS en producción
- ✅ Configura firewalls apropiados
- ✅ Mantén las dependencias actualizadas
- ✅ Usa contraseñas seguras para la base de datos
- ✅ Configura backups regulares
- ✅ Monitorea logs de seguridad
- ✅ Implementa rate limiting

## Contribuir

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/nueva-caracteristica`)
3. Commit tus cambios (`git commit -am 'Agregar nueva característica'`)
4. Push a la rama (`git push origin feature/nueva-caracteristica`)
5. Abre un Pull Request

## Licencia

Este proyecto está bajo la licencia MIT. Ver el archivo `LICENSE` para más detalles.

## Soporte

Si encuentras algún problema o tienes preguntas, por favor abre un issue en el repositorio de GitHub.

---

**¡Mantén tus contraseñas seguras con ArcadeLockVault! 🔐**