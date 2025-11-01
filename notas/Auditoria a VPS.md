# Opción 1: du + sort + head (más portable)
sudo du -h -d1 | sort -hr | head -n20

# Opción 2: si tu du no soporta -d1, usa --max-depth=1 sin pegarlo a sort
sudo du -h --max-depth=1 | sort -hr | head -n20

# Opción 3: si aún falla, separa el comando
sudo du -h --max-depth=1 | sort -rh | head -n20

# Top-20 carpetas del disco completo
sudo du -hx / | sort -hr | head -n20

# O si quieres empezar por /var
sudo du -hx /var | sort -hr | head -n20

docker system df          # resumen por tipo
docker images -a          # imágenes (intermedias incluidas)
docker ps -a              # contenedores parados/existentes
docker volume ls          # volúmenes huérfanos


2. Limpia seguro (solo lo que no se use)
bash
Copy
# 1) Para ver lo que se borraría (dry-run)
docker system prune --all --volumes

# 2) Si el listado te convence, confirma:
docker system prune --all --volumes --force

5. Comprueba el resultado
bash
Copy
docker system df
du -sh /var/lib/docker