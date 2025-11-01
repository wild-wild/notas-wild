# Error al subir imágenes en Laravel usando disco personalizado `uploads`

## Problema

Al intentar mostrar o subir imágenes usando el disco `uploads` definido en `config/filesystems.php`:

```php
<img src="{{ Storage::disk('uploads')->url($post->imagenPost) }}" alt="Post">
```

Laravel lanza el siguiente error:

```
League\Flysystem\UnableToCreateDirectory
Unable to create a directory at /home/easypanel/volumes/sis-pisconet360-uploads.
```

## Causa

El error ocurre porque Laravel intenta crear subdirectorios dentro de:

```
/home/easypanel/volumes/sis-pisconet360-uploads
```

pero la carpeta donde se guardarán las imágenes (`imagenes_posts`) **no existe** o los permisos no permiten que el usuario del servidor web (`www-data`) pueda escribir en ella.

Aunque el disco esté bien configurado, Flysystem requiere que la ruta exista y tenga permisos de escritura.

## Solución

1. Crear la carpeta que se va a usar para almacenar las imágenes.
2. Asignarle el propietario correcto (`www-data`).
3. Dar permisos de lectura/escritura/ejecución para el grupo y propietario.

### Comandos:

```bash
mkdir -p /home/easypanel/volumes/sis-pisconet360-uploads/imagenes_posts
chown -R www-data:www-data /home/easypanel/volumes/sis-pisconet360-uploads/imagenes_posts
chmod -R 775 /home/easypanel/volumes/sis-pisconet360-uploads/imagenes_posts
```

Con esto:

- Laravel podrá crear subdirectorios y guardar imágenes.
- Las imágenes subidas se podrán mostrar usando `Storage::disk('uploads')->url($post->imagenPost)`.
- Se evita el error `UnableToCreateDirectory`.

## Notas

- Siempre verifica que el usuario que corre PHP-FPM o Apache/Nginx tenga permisos de escritura en el directorio de almacenamiento.
- En entornos de producción con contenedores o volúmenes, asegúrate de mapear correctamente el volumen para que persistan los archivos subidos.