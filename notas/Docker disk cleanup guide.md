# Guía Rápida de Análisis y Limpieza de Docker

Esta guía sirve para revisar el uso de espacio en disco y limpiar Docker de manera segura, liberando espacio sin afectar contenedores o imágenes en uso.

---

## 1. Revisar el espacio usado

### Opciones para listar carpetas más grandes

**Opción 1: Más portable**
```bash
sudo du -h -d1 | sort -hr | head -n20
```

**Opción 2: Si tu `du` no soporta `-d1`**
```bash
sudo du -h --max-depth=1 | sort -hr | head -n20
```

**Opción 3: Separando pasos**
```bash
sudo du -h --max-depth=1 | sort -rh | head -n20
```

**Top 20 carpetas de todo el disco**
```bash
sudo du -hx / | sort -hr | head -n20
```

**Top 20 carpetas comenzando desde `/var`**
```bash
sudo du -hx /var | sort -hr | head -n20
```

### Comandos Docker para ver uso de espacio
```bash
docker system df        # Resumen por tipo

docker images -a        # Lista de imágenes (intermedias incluidas)
docker ps -a            # Contenedores parados/existentes
docker volume ls        # Volúmenes huérfanos
```

---

## 2. Limpieza segura de Docker

### Paso 1: Ver qué se borraría (dry-run)
```bash
docker system prune --all --volumes
```

### Paso 2: Confirmar limpieza si todo está correcto
```bash
docker system prune --all --volumes --force
```

---

## 3. Comprobar resultados
```bash
docker system df        # Resumen post-limpieza
du -sh /var/lib/docker  # Tamaño actual de la carpeta de Docker
```

---

**Consejo:** Revisa siempre primero con el dry-run antes de confirmar la limpieza completa para no eliminar datos necesarios accidentalmente.