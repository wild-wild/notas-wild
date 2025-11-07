# Limpieza de Docker y análisis de espacio en Ubuntu 22.04.5 LTS

**Descripción:**  
Este documento registra el procedimiento realizado en un servidor Ubuntu 22.04.5 LTS para liberar espacio en disco utilizando Docker. Se incluyen los comandos ejecutados, el estado del sistema antes y después de la limpieza, y la información de espacio en disco.

---

## Información del Sistema

```
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-157-generic x86_64)

* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/pro

System information as of Sat Nov 1 16:21:19 UTC 2025

System load:  0.03               Processes: 171
Usage of /:   15.1% of 96.73GB   Users logged in: 0
Memory usage: 23%                IPv4 address for eth0: 72.61.217.85
Swap usage:   0%                 IPv6 address for eth0: 2a02:4780:66:aeff::1

*** System restart required ***
```

---

## Comando para analizar espacio ocupado por directorios

```bash
sudo du -hx / | sort -hr | head -n20
```

**Salida:**

```
15G     /
13G     /var/lib
13G     /var
12G     /var/lib/docker/overlay2
12G     /var/lib/docker
1.8G    /usr
679M    /usr/lib
...
461M    /var/lib/snapd
```

---

## Estado de Docker antes de la limpieza

```bash
docker system df
```

**Salida:**

```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          21        7         10.18GB   7.196GB (70%)
Containers      9         5         53MB      36.95MB (69%)
Local Volumes   0         0         0B        0B
Build Cache     108       0         21.56MB   21.56MB
```

---

## Limpieza completa de Docker

```bash
docker system prune --all --volumes --force
```

**Resultados:**
- Containers eliminados
- Imágenes eliminadas
- Caché de build eliminada
- Total reclaimable: 7.766GB  

---

## Estado de Docker después de la limpieza

```bash
docker system df
```

**Salida:**

```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          5         5         3.397GB   7.834MB (0%)
Containers      5         5         16.06MB   0B (0%)
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B
```

---

## Verificación del tamaño de la carpeta de Docker

```bash
du -sh /var/lib/docker
```

**Salida:**

```
7.0G    /var/lib/docker
```

---

**Resumen:**
- Se liberaron aproximadamente **7.7 GB** de espacio en disco.  
- La carpeta `/var/lib/docker` pasó de ~12GB a 7GB tras la limpieza.  
- Se eliminó caché, contenedores e imágenes no utilizadas.  

---

**Referencias:**
- [Documentación oficial Ubuntu](https://help.ubuntu.com)
- [Docker System Prune](https://docs.docker.com/engine/reference/commandline/system_prune/)

