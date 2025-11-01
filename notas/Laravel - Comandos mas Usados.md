# 📄 Comandos más utilizados en Laravel

Este documento recopila los comandos más comunes y útiles al trabajar con proyectos basados en el framework Laravel. Incluye comandos de Artisan, Composer, migraciones, Tinker y más.

---

## 🧱 Comandos básicos de Laravel

### 🔄 Crear un nuevo proyecto Laravel
```bash
laravel new nombre-proyecto
# o
composer create-project laravel/laravel nombre-proyecto
```

### 🚀 Iniciar el servidor de desarrollo
```bash
php artisan serve
```

---

## 🛠️ Artisan: Comandos de consola de Laravel

### 📜 Listar comandos disponibles
```bash
php artisan list
```

### ❗ Mostrar ayuda de un comando
```bash
php artisan help comando
```

### 🧼 Limpiar cachés de configuración y rutas
```bash
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear
```

### 🔧 Generar la clave de la aplicación
```bash
php artisan key:generate
```

---

## 🗃️ Migraciones y Base de Datos

### 📌 Ejecutar migraciones
```bash
php artisan migrate
```

### 📌 Revertir la última migración
```bash
php artisan migrate:rollback
```

### 📌 Refrescar la base de datos (rollback + migrate)
```bash
php artisan migrate:refresh
```

### ⚡ Crear archivo de migración
```bash
php artisan make:migration create_nombre_tabla_table
```

---

## 🧬 Modelos, Controladores y Otros Recursos

### 🧱 Crear un modelo Eloquent
```bash
php artisan make:model NombreModelo
```

### 🧱 Crear un modelo con migración
```bash
php artisan make:model NombreModelo -m
```

### 🎮 Crear un controlador
```bash
php artisan make:controller NombreControlador
```

### 👇 Crear un controlador tipo resource (CRUD)
```bash
php artisan make:controller NombreControlador --resource
```

### 📦 Crear un seeder
```bash
php artisan make:seeder NombreSeeder
```

---

## 🧪 Pruebas y Desarrollo

### ▶️ Ejecutar las pruebas unitarias
```bash
php artisan test
```

### 🛠️ Entrar a Tinker para pruebas interactivas
```bash
php artisan tinker
```

### 🔃 Ejecutar jobs en cola
```bash
php artisan queue:work
```

---

## 🐘 Comandos relacionados con Composer

### 📦 Instalar dependencias del proyecto
```bash
composer install
```

### 🔄 Actualizar dependencias
```bash
composer update
```

### 🧼 Limpiar caché de Composer
```bash
composer clear-cache
```

---

## 📡 Comandos útiles para despliegue

### ⚙️ Optimizar configuración para producción
```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### 🔧 Ejecutar migraciones en producción
```bash
php artisan migrate --force
```

---
### 🔧 Crear el enlace simbolico
```bash
php artisan storage:link
```

---

✅ **Este documento sirve como referencia rápida para desarrolladores Laravel. Puedes agregar tus propios alias, scripts personalizados o flujos de trabajo específicos.**

