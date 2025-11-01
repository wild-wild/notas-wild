# 📄 Documentación de Comandos Usados para Diagnóstico y Configuración (Laravel + Easypanel + Nixpacks)

Este documento recoge los comandos clave que fueron utilizados durante el proceso de diagnóstico y solución de errores relacionados con la configuración de subida de archivos en un despliegue de Laravel utilizando Easypanel y Nixpacks.

---

## 🐳 Comandos relacionados con Docker

### 🔍 Ver contenedores en ejecución
```bash
docker ps
```

### 🔍 Ver todos los contenedores (incluidos los detenidos)
```bash
docker ps -a
```

### 🔎 Filtrar contenedores por nombre (ejemplo: psiconet)
```bash
docker ps -a | grep psiconet
```

### 📜 Ver logs de un contenedor específico
```bash
docker logs <container_id>
```
Ejemplo:
```bash
docker logs 9017015e6f4b
```

### 📜 Ver logs de un servicio de Docker Swarm
```bash
docker service logs <service_name> --tail 100
```
Ejemplo:
```bash
docker service logs psiconet360_sis-pisconet360 --tail 100
```

---

## 🛠️ Comandos dentro del contenedor

### 🔐 Acceder al contenedor en modo interactivo (shell)
```bash
docker exec -it <container_id> sh
```

### 🔍 Buscar procesos relacionados con Nginx
```bash
ps aux | grep nginx
```

### 🔍 Ver el proceso maestro de Nginx
```bash
ps aux | grep "nginx: master"
```

### ❌ Intentar ejecutar Nginx manualmente (no disponible en el host)
```bash
nginx -c /nginx.conf
```
> Este comando falló en el host ya que Nginx no está instalado fuera del contenedor.

---

## 📁 Comandos para revisar o editar configuración

### 🔍 Buscar archivos nginx.conf dentro del contenedor
```bash
find / -name "nginx.conf" 2>/dev/null
```

### 🔍 Ver la configuración de Nginx que se está usando
```bash
ps aux | grep nginx
```

### 🔍 Ver configuraciones de PHP (dentro del contenedor)
```bash
php --ini
php -r "echo 'upload_max_filesize: ' . ini_get('upload_max_filesize') . PHP_EOL;"
php -r "echo 'post_max_size: ' . ini_get('post_max_size') . PHP_EOL;"
```

---

## 🧹 Limpieza y notificación

### 🔋 Ver estado del sistema
```bash
uptime
who
```

### ✉️ Mostrar notificación de reinicio del sistema
```bash
echo "*** System restart required ***"
```

---

## 📌 Notas adicionales
- Los comandos de Docker se ejecutaron desde el servidor Ubuntu donde estaba desplegado el proyecto.
- Algunos comandos fallaron debido a diferencias entre el host y el contenedor (por ejemplo, `nginx` no estaba disponible en el host).
- Se utilizó `docker service logs` para revisar los logs de servicios en modo Swarm.
- Este conjunto de comandos permitió identificar la configuración correcta de Nginx y corregir errores relacionados con la subida de archivos mediante `nixpacks.toml`.

---

✅ **Documento generado automáticamente con fines de documentación técnica**.

