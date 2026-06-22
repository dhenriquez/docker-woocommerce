## Stack para WooCommerce Multi-Sitio de Alto Rendimiento con Docker

Este proyecto proporciona una configuración de Docker Compose diseñada para desplegar **múltiples** tiendas WooCommerce de alto rendimiento en un único servidor VPS, aislando completamente las bases de datos y orquestando el tráfico a través de un Proxy Inverso Maestro. La arquitectura se basa en las mejores prácticas de la industria y cuenta con preconfiguraciones avanzadas para soportar alto tráfico de forma eficiente.

## Arquitectura Multi-Sitio
El stack de software ha sido seleccionado y optimizado cuidadosamente para aislar los entornos de los clientes sin sacrificar la velocidad:

1.  **Proxy Inverso Maestro (`nginx-proxy`):** Se encarga de escuchar los puertos 80 y 443 del servidor. Lee las peticiones entrantes y las enruta automáticamente al contenedor Nginx interno correcto basándose en la variable de entorno `VIRTUAL_HOST`.
2.  **Servidor Web Aislado (Nginx Interno con Microcaching y Gzip):** Cada sitio tiene su propio contenedor Nginx privado. Sirve archivos estáticos directamente con compresión Gzip y cuenta con **FastCGI Microcaching (de 1 segundo)** para páginas públicas de usuarios no autenticados, protegiendo al backend de saturaciones ante picos de tráfico.
3.  **Procesador PHP (PHP-FPM 8.3 con OPcache y Sesiones en RAM):** Ejecuta la imagen oficial de WordPress con PHP-FPM. Cuenta con optimizaciones de **OPcache** para acelerar el procesamiento y almacena las sesiones PHP directamente en Redis, evitando la sobreescritura del disco y de la base de datos.
4.  **Bases de Datos y Caché Aisladas (MariaDB y Redis Optimizados):** Cada sitio levanta su propia instancia de MariaDB (con 1GB preasignado al pool de memoria de InnoDB para consultas ultrarrápidas) y Redis para caché de objetos transitorios (limitado a 256MB con descarte automático LRU).

## Requisitos Previos
Asegúrate de tener instaladas las siguientes herramientas en tu servidor:
*   [Docker](https://docs.docker.com/get-docker/)
*   [Docker Compose (v2+)](https://docs.docker.com/compose/install/)

## Estructura de Archivos
El proyecto tiene la siguiente estructura orientada a plantillas:
```text
docker-woocommerce/
├── proxy/
│   └── docker-compose.yml   <-- El proxy maestro global
├── nginx/
│   └── default.conf         <-- Nginx interno preconfigurado con Microcaching y Gzip
├── php-conf/
│   └── custom.ini           <-- Configuración PHP (OPcache y sesiones en Redis)
├── docker-compose.yml       <-- Plantilla optimizada de la tienda
├── .env.example             <-- Plantilla de credenciales y dominio
└── README.md                <-- Este archivo
```

## Instalación y Configuración

Sigue estos pasos para poner en marcha tu entorno multi-sitio:

### Paso 1: Levantar el Proxy Maestro
Este paso **sólo se realiza una vez** en todo tu servidor VPS.
```bash
cd proxy
docker compose up -d
```
El proxy maestro se quedará a la espera de que levantes nuevas tiendas.

### Paso 2: Crear una Nueva Tienda
Por cada tienda que quieras alojar en tu VPS, debes crear un clon o copiar los archivos base.
1.  **Copia los archivos:** Crea una carpeta nueva en tu servidor (ej. `/var/www/tienda-1`) y copia dentro todos los archivos base (`docker-compose.yml`, `.env.example`, la carpeta `nginx/` y la carpeta `php-conf/`).
2.  **Configura el entorno (`.env`):**
    Copia la plantilla y edítala:
    ```bash
    cp .env.example .env
    nano .env
    ```
    Configura el dominio (`VIRTUAL_HOST`) y las credenciales de base de datos (`DB_ROOT_PASSWORD`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`).
3.  **Inicia los servicios de la tienda:**
    ```bash
    docker compose up -d
    ```
    *Nota: Docker automáticamente prefijará los contenedores y volúmenes con el nombre de tu carpeta, evitando colisiones de nombre con otras tiendas.*

## Configuración Post-Instalación en WordPress

Una vez que los contenedores estén en funcionamiento, completa la instalación:

1.  **Instala WordPress:**
    Abre tu navegador en el dominio que definiste en `VIRTUAL_HOST`. Si tus DNS apuntan a la IP del VPS, Nginx-Proxy te mostrará el asistente de WordPress de inmediato.
2.  **Plugin de Redis:**
    *   Instala y activa el plugin **"Redis Object Cache"**.
    *   Ve a **Ajustes > Redis** y habilita la caché. Gracias a las variables de entorno inyectadas en Docker, el plugin se conectará automáticamente.

## Gestión de los Contenedores

*   **Para detener una tienda:** (ejecutar dentro de su carpeta respectiva)
    ```bash
    docker compose down
    ```
*   **Para ver los logs de una tienda:**
    ```bash
    docker compose logs -f
    ```

## Consideraciones para Producción

*   **SSL/HTTPS:** Nginx-Proxy soporta Let's Encrypt usando un contenedor compañero llamado `nginx-proxy-acme`. Puedes agregarlo a la carpeta `proxy` para automatizar los certificados SSL.
*   **Copias de Seguridad:** Implementa rutinas para respaldar los volúmenes de Docker (`db_data` y `wordpress_data`) en cada tienda.
*   **Cron Real de WordPress (Recomendado para Alto Tráfico):**
    Por defecto, WordPress ejecuta tareas programadas cada vez que recibe visitas, lo que degrada el rendimiento. Para desactivarlo y usar un cron del sistema:
    1. Agrega `define('DISABLE_WP_CRON', true);` en las variables de entorno (`WORDPRESS_CONFIG_EXTRA`) en `docker-compose.yml`.
    2. Ejecuta `crontab -e` en tu VPS y programa la ejecución real cada 5 minutos:
       ```bash
       */5 * * * * docker compose -f /var/www/tu-tienda/docker-compose.yml exec -T wordpress php /var/www/html/wp-cron.php > /dev/null 2>&1
       ```
*   **Correo Electrónico:** Por defecto, PHP en Docker no envía correos. Usa plugins como WP Mail SMTP.

## Licencia
Este proyecto se distribuye bajo la licencia MIT.