# COMANDOS A EJECUTAR DESPUES DE HACER DEPLOY 
Ejecutar en la tarminal de la app (Laravel)

> 1.- Comando para limpiar cache

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

> 2.- Comando para crear enlace simbolico

```bash
php artisan storage:link
```


---

## **II. Shell y Nginx**

> Comandos usados para inspeccionar, modificar la configuración del servidor web y recargar el servicio.

```bash
cat /nginx.conf
sed -i '/http {/a \    client_max_body_size 50M;' /nginx.conf
sed -i '/server {/a \        client_max_body_size 50M;' /nginx.conf
grep "client_max_body_size" /nginx.conf
pkill -HUP nginx
```

