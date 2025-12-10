# Configuración de n8n con Docker

Esta configuración despliega n8n con PostgreSQL y lo conecta a nginx-proxy-manager.

## Requisitos previos

- Docker y Docker Compose instalados
- Nginx Proxy Manager ejecutándose en la red `nginx-proxy-manager_default`
- Dominio configurado apuntando a tu servidor

## Configuración inicial

### 1. Copiar el archivo de variables de entorno

```bash
cp .env.example .env
```

### 2. Editar el archivo `.env`

Edita el archivo `.env` y cambia los valores por defecto:

```bash
# Base de datos - Usa una contraseña segura
POSTGRES_PASSWORD=tu_password_seguro_aqui
N8N_DB_PASSWORD=tu_password_seguro_aqui

# Autenticación básica de n8n
N8N_BASIC_AUTH_USER=tu_usuario
N8N_BASIC_AUTH_PASSWORD=tu_password_admin

# Configuración
N8N_HOST=n8n.julian-mbp.pro
LETSENCRYPT_EMAIL=julian.bastidasmp@gmail.com
```

### 3. Configurar Nginx Proxy Manager

En tu panel de Nginx Proxy Manager, crea un nuevo **Proxy Host**:

- **Domain Names**: `n8n.julian-mbp.pro`
- **Scheme**: `http`
- **Forward Hostname/IP**: `n8n`
- **Forward Port**: `5678`
- **Cache Assets**: ✓ (opcional)
- **Block Common Exploits**: ✓
- **Websockets Support**: ✓ (importante)

En la pestaña **SSL**:
- **SSL Certificate**: Request a new SSL Certificate
- **Force SSL**: ✓
- **HTTP/2 Support**: ✓
- **HSTS Enabled**: ✓

### 4. Iniciar los contenedores

```bash
docker-compose up -d
```

### 5. Verificar los logs

```bash
# Ver todos los logs
docker-compose logs -f

# Ver solo logs de n8n
docker logs n8n -f

# Ver solo logs de postgres
docker logs n8n-postgres -f
```

### 6. Acceder a n8n

Abre tu navegador y accede a: `https://n8n.julian-mbp.pro`

Usa las credenciales configuradas en el archivo `.env`:
- Usuario: valor de `N8N_BASIC_AUTH_USER`
- Contraseña: valor de `N8N_BASIC_AUTH_PASSWORD`

## Comandos útiles

### Detener los servicios

```bash
docker-compose down
```

### Reiniciar los servicios

```bash
docker-compose restart
```

### Ver el estado de los contenedores

```bash
docker-compose ps
```

### Verificar la red

```bash
docker network inspect nginx-proxy-manager_default
```

## Backup y Restauración

### Crear backup de la base de datos

```bash
docker exec n8n-postgres pg_dump -U n8n n8n > n8n_backup_$(date +%Y%m%d).sql
```

### Restaurar backup

```bash
docker exec -i n8n-postgres psql -U n8n -d n8n < n8n_backup_20241210.sql
```

### Backup de los datos de n8n

Los datos de n8n se almacenan en el volumen `n8n_data`. Para hacer backup:

```bash
docker run --rm -v n8n_n8n_data:/data -v $(pwd):/backup alpine tar czf /backup/n8n_data_backup.tar.gz -C /data .
```

## Troubleshooting

### n8n no se conecta a PostgreSQL

Verifica que PostgreSQL esté saludable:

```bash
docker exec n8n-postgres pg_isready -U n8n -d n8n
```

### No puedo acceder al dominio

1. Verifica que ambos contenedores estén en la red correcta:
```bash
docker network inspect nginx-proxy-manager_default
```

2. Verifica los logs de n8n:
```bash
docker logs n8n -f
```

3. Verifica la configuración en Nginx Proxy Manager

### Cambiar contraseñas

1. Edita el archivo `.env`
2. Reinicia los servicios:
```bash
docker-compose down
docker-compose up -d
```

## Mantenimiento

### Limpiar ejecuciones antiguas

n8n está configurado para limpiar automáticamente las ejecuciones antiguas:
- `EXECUTIONS_DATA_MAX_AGE=336` (14 días)
- `EXECUTIONS_DATA_PRUNE_MAX_COUNT=10000`

Puedes ajustar estos valores en el archivo `docker-compose.yml`.

### Actualizar n8n

```bash
docker-compose pull
docker-compose up -d
```

## Configuración de red

Este docker-compose se conecta a la red externa `nginx-proxy-manager_default`. 

Si tu red de Nginx Proxy Manager tiene un nombre diferente, actualiza la sección `networks` en `docker-compose.yml`.

## Seguridad

- ✓ Autenticación básica habilitada
- ✓ HTTPS forzado mediante Nginx Proxy Manager
- ✓ Base de datos con contraseña
- ✓ Variables de entorno en archivo `.env` (no versionado)

**Importante**: Nunca subas el archivo `.env` al repositorio. Usa `.env.example` como plantilla.
