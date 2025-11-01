# Solución: Aumentar límite de subida de archivos en Laravel + Easypanel + Nixpacks

## Problema
Al subir archivos mayores a 1MB en Laravel desplegado en Easypanel con Nixpacks, se obtiene el error **413 Request Entity Too Large**.

## Causa
Laravel en Easypanel usa Nixpacks, que configura automáticamente PHP y Nginx. Por defecto, ambos tienen límites bajos:
- PHP: `upload_max_filesize = 2M`, `post_max_size = 2M`
- Nginx: `client_max_body_size` por defecto (1M)

## Solución Completa

### 1. Configurar PHP (php.ini)

Crear archivo `php.ini` en la raíz del proyecto Laravel:

```ini
upload_max_filesize = 50M
post_max_size = 50M
memory_limit = 256M
max_execution_time = 300
max_input_time = 300
max_file_uploads = 20
```

### 2. Configurar Nginx (.nixpacks/nginx.conf)

Crear carpeta `.nixpacks` y archivo `nginx.conf`:

```bash
mkdir -p .nixpacks
```

Contenido de `.nixpacks/nginx.conf`:

```nginx
client_max_body_size 50M;
client_body_buffer_size 128k;
```

### 3. Configurar variables de entorno en Easypanel

Agregar estas variables en el panel de Environment Variables:

```bash
# Configuración de Nixpacks
NIXPACKS_PHP_ROOT_DIR=/app/public
NIXPACKS_PHP_FALLBACK_PATH=/index.php

# Configuración de PHP
PHP_INI_SCAN_DIR=/app:/nix/store/a0qcy2m5mqwrlkr3xkza0n2wdmbq8myq-php-with-extensions-8.2.27/lib

# Configuración de Nginx
NIXPACKS_NGINX_EXTRA_CONFIG=client_max_body_size 50M;
NGINX_CLIENT_MAX_BODY_SIZE=50M
```

**Nota:** La ruta del store de Nix puede variar. Obtén la correcta ejecutando `php --ini` dentro del contenedor.

### 4. Crear nixpacks.toml para persistencia

Crear archivo `nixpacks.toml` en la raíz del proyecto:

```toml
[start]
cmd = "bash -c 'if ! grep -q \"client_max_body_size\" /nginx.conf; then sed -i \"/http {/a \\    client_max_body_size 50M;\" /nginx.conf; sed -i \"/server {/a \\        client_max_body_size 50M;\" /nginx.conf; fi && node /assets/scripts/prestart.mjs /assets/nginx.template.conf /nginx.conf && (php-fpm -y /assets/php-fpm.conf & nginx -c /nginx.conf)'"
```

Este script:
- Verifica que no exista ya la configuración (evita duplicados)
- Agrega `client_max_body_size 50M;` en los bloques `http` y `server` de nginx.conf
- Ejecuta el comando de inicio normal de Nixpacks

### 5. Estructura final del proyecto

```
proyecto-laravel/
├── .nixpacks/
│   └── nginx.conf           # Configuración de Nginx
├── app/
├── public/
├── php.ini                  # Configuración de PHP
├── nixpacks.toml            # Script de inicio personalizado
├── composer.json
└── ...
```

### 6. Desplegar cambios

```bash
# Agregar archivos al repositorio
git add php.ini .nixpacks/ nixpacks.toml
git commit -m "Configurar límite de subida de archivos a 50MB"
git push

# En Easypanel: Click en "Redeploy"
```

## Verificación

### Verificar configuración de PHP

Conectarse al contenedor y ejecutar:

```bash
php --ini
# Debe mostrar: Additional .ini files parsed: /app/php.ini

php -r "echo 'upload_max_filesize: ' . ini_get('upload_max_filesize') . PHP_EOL;"
# Debe mostrar: upload_max_filesize: 50M

php -r "echo 'post_max_size: ' . ini_get('post_max_size') . PHP_EOL;"
# Debe mostrar: post_max_size: 50M
```

### Verificar configuración de Nginx

```bash
grep "client_max_body_size" /nginx.conf
# Debe mostrar:
#     client_max_body_size 50M;
#         client_max_body_size 50M;
```

### Verificar que el contenedor esté corriendo

Desde el VPS:

```bash
docker ps | grep psiconet
# Debe mostrar el contenedor con estado "Up"
```

## Solución de problemas

### Error: Contenedor se cierra inmediatamente (Exited)

Verificar logs:
```bash
docker service logs nombre_servicio --tail 100
```

Causa común: Variable `NIXPACKS_START_CMD` mal configurada o script sin permisos de ejecución.

### Error 413 persiste después de configurar

1. Verificar que Nginx cargó la configuración:
   ```bash
   docker exec -it <container_id> grep "client_max_body_size" /nginx.conf
   ```

2. Si no aparece, el `nixpacks.toml` no se ejecutó correctamente. Verificar sintaxis.

3. Recargar Nginx manualmente:
   ```bash
   docker exec -it <container_id> pkill -HUP nginx
   ```

### PHP no carga php.ini personalizado

Verificar que `PHP_INI_SCAN_DIR` incluya `/app`:
```bash
docker exec -it <container_id> php --ini
```

Debe mostrar: `Scan for additional .ini files in: /app:...`

## Notas importantes

- **No usar `localStorage` o `sessionStorage`** en artifacts de Claude.ai (no están disponibles)
- Los archivos en `/nix/store/` son inmutables, por eso se modifica `/nginx.conf` generado dinámicamente
- Easypanel usa Docker Swarm, no Docker Compose
- El límite de 50MB es configurable, ajustar según necesidades
- Para archivos muy grandes (>50MB), considerar usar almacenamiento S3 o chunked uploads

## Variables eliminadas (no funcionan con Nixpacks)

Estas variables NO tienen efecto y pueden eliminarse:

```bash
# ❌ No funcionan
PHP_UPLOAD_MAX_FILESIZE=50M
PHP_POST_MAX_SIZE=50M
PHP_MEMORY_LIMIT=256M
PHP_MAX_EXECUTION_TIME=300
NIXPACKS_PHP_INI_OVERWRITE=/app/php.ini
```

Usar en su lugar:
- ✅ Archivo `php.ini` + variable `PHP_INI_SCAN_DIR`
- ✅ Archivo `.nixpacks/nginx.conf` + `nixpacks.toml`

## Caso de estudio: Diagnóstico y solución paso a paso

### Síntoma inicial
Error 413 al intentar subir archivo de 4.5 MB a través de Livewire:

```
2025/11/01 14:10:11 [error] 41#41: *43 client intended to send too large body: 4511672 bytes
POST /livewire/upload-file HTTP/1.1" 413 585
```

### Diagnóstico

#### 1. Verificar configuración de PHP
```bash
# Conectarse al contenedor
docker exec -it <container_id> sh

# Verificar límites de PHP
php -r "echo 'upload_max_filesize: ' . ini_get('upload_max_filesize') . PHP_EOL;"
php -r "echo 'post_max_size: ' . ini_get('post_max_size') . PHP_EOL;"
```

**Resultado:** PHP configurado correctamente (50M) ✅

#### 2. Identificar el problema real
Como PHP estaba bien configurado, el error 413 indica que **Nginx** está bloqueando la subida.

#### 3. Localizar archivo de configuración de Nginx

```bash
# Buscar todos los archivos nginx.conf
find / -name "nginx.conf" 2>/dev/null
```

**Resultado encontrado:**
```
/nix/store/8lncw1wjkrywc0shnwhpsgwgj9cn4da1-nginx-1.26.2/conf/nginx.conf  # Config por defecto de Nix
/app/.nixpacks/nginx.conf  # Config personalizada (no se estaba usando)
/nginx.conf  # Config generada por Nixpacks (la que se usa realmente)
```

#### 4. Identificar qué archivo usa Nginx

```bash
# Ver procesos de Nginx
ps aux | grep nginx
```

**Resultado:**
```
root  39  nginx: master process nginx -c /nginx.conf
```

**Conclusión:** Nginx usa `/nginx.conf`, no el del store de Nix ni el de `.nixpacks/`

### Solución aplicada

#### Paso 1: Verificar contenido del nginx.conf activo

```bash
cat /nginx.conf
```

Estructura encontrada:
```nginx
worker_processes 5;
daemon off;
events {
  worker_connections  4096;
}
http {
    # ... configuración ...
    server {
        listen 80;
        server_name localhost;
        root /app/public;
        # ... más configuración ...
    }
}
```

#### Paso 2: Agregar client_max_body_size

```bash
# Agregar en el bloque http
sed -i '/http {/a \    client_max_body_size 50M;' /nginx.conf

# Agregar también en el bloque server (redundancia por seguridad)
sed -i '/server {/a \        client_max_body_size 50M;' /nginx.conf

# Verificar que se agregó
grep "client_max_body_size" /nginx.conf
```

**Resultado:**
```
    client_max_body_size 50M;
        client_max_body_size 50M;
```

#### Paso 3: Recargar Nginx

```bash
# Método 1: Usando kill
ps aux | grep "nginx: master"
kill -HUP <PID_DEL_MASTER>

# Método 2: Usando pkill
pkill -HUP nginx
```

#### Paso 4: Verificar recarga

```bash
ps aux | grep nginx
```

Los worker processes deben tener nuevos PIDs, indicando que se recargaron correctamente.

#### Paso 5: Probar subida de archivo

Resultado: ✅ **Archivo de 4.5 MB subido exitosamente**

### Hacer la solución permanente

El problema es que `/nginx.conf` se regenera en cada deploy. La solución temporal se pierde.

#### Solución: Modificar nginx.conf al inicio con nixpacks.toml

Crear archivo `nixpacks.toml` en la raíz del proyecto:

```toml
[start]
cmd = "bash -c 'if ! grep -q \"client_max_body_size\" /nginx.conf; then sed -i \"/http {/a \\    client_max_body_size 50M;\" /nginx.conf; sed -i \"/server {/a \\        client_max_body_size 50M;\" /nginx.conf; fi && node /assets/scripts/prestart.mjs /assets/nginx.template.conf /nginx.conf && (php-fpm -y /assets/php-fpm.conf & nginx -c /nginx.conf)'"
```

**¿Qué hace este comando?**

1. **Verifica** si `client_max_body_size` ya existe (evita duplicados en redeploys)
2. **Modifica** `/nginx.conf` agregando la directiva en bloques `http` y `server`
3. **Ejecuta** el comando de inicio original de Nixpacks

#### Desplegar solución permanente

```bash
git add nixpacks.toml
git commit -m "Fix: Configurar límite de subida de archivos a 50MB permanente"
git push
```

En Easypanel: Click en **"Redeploy"**

### Verificación post-deploy

```bash
# 1. Verificar que el contenedor está corriendo
docker ps | grep psiconet

# 2. Entrar al contenedor
docker exec -it <container_id> sh

# 3. Verificar que nginx.conf tiene la configuración
grep "client_max_body_size" /nginx.conf

# 4. Verificar procesos de Nginx
ps aux | grep nginx
```

**Resultado esperado:**
- Contenedor con estado "Up" ✅
- `/nginx.conf` contiene `client_max_body_size 50M;` ✅
- Nginx corriendo correctamente ✅
- Subida de archivos >1MB funciona ✅

## Por qué otras soluciones no funcionaron

### ❌ Variables de entorno PHP_* no funcionan
```bash
PHP_UPLOAD_MAX_FILESIZE=50M  # Nixpacks no las procesa
PHP_POST_MAX_SIZE=50M
```
**Razón:** Nixpacks no convierte automáticamente estas variables a configuración de PHP.

**Solución correcta:** Usar `php.ini` + `PHP_INI_SCAN_DIR`

### ❌ NIXPACKS_PHP_INI_OVERWRITE no sobrescribe
```bash
NIXPACKS_PHP_INI_OVERWRITE=/app/php.ini
```
**Razón:** Esta variable no funciona como se espera. El archivo no se usa como principal.

**Solución correcta:** Usar `PHP_INI_SCAN_DIR` para agregar directorio adicional de escaneo

### ❌ .nixpacks/nginx.conf ignorado
```nginx
# .nixpacks/nginx.conf
client_max_body_size 50M;
```
**Razón:** Nixpacks genera `/nginx.conf` dinámicamente y no incluye archivos de `.nixpacks/`

**Solución correcta:** Modificar `/nginx.conf` al inicio con `nixpacks.toml`

### ❌ NIXPACKS_NGINX_EXTRA_CONFIG no se aplica
```bash
NIXPACKS_NGINX_EXTRA_CONFIG=client_max_body_size 50M;
```
**Razón:** Esta variable no inyecta configuración en el nginx.conf generado

**Solución correcta:** Script en `nixpacks.toml` que modifica el archivo

### ❌ Script start.sh causaba que el contenedor se cerrara
```bash
NIXPACKS_START_CMD=/app/start.sh
```
**Problema:** El script no ejecutaba correctamente el comando original, causando que el contenedor terminara inmediatamente (estado "Exited").

**Solución correcta:** Usar `nixpacks.toml` con el comando completo incluyendo el inicio de servicios

## Lecciones aprendidas

1. **Nixpacks usa configuración dinámica:** Los archivos se generan en tiempo de ejecución, no en build time
2. **Nginx usa `/nginx.conf` específico:** No el del store de Nix ni archivos en `.nixpacks/`
3. **Variables de entorno no siempre funcionan:** Revisar documentación de Nixpacks para cada caso
4. **Verificar qué archivos se usan realmente:** Usar `ps aux` y `find` para diagnosticar
5. **Soluciones temporales vs permanentes:** Modificaciones manuales se pierden en redeploy
6. **El comando `start` debe ser completo:** Debe iniciar todos los servicios necesarios (php-fpm + nginx)

## Referencias

- [Documentación de Nixpacks](https://nixpacks.com/)
- [Documentación de Easypanel](https://easypanel.io/docs)
- [Configuración de PHP-FPM](https://www.php.net/manual/en/install.fpm.configuration.php)
- [Directivas de Nginx](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)
- [Nginx reload vs restart](http://nginx.org/en/docs/control.html)