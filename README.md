## Pila de Servidor WooCommerce Multi-Sitio de Alto Rendimiento con Docker
Repositorio del Proyecto: https://github.com/dhenriquez/docker-woocommerce

Este proyecto proporciona una configuración de Docker Compose diseñada para desplegar **múltiples** tiendas WooCommerce de alto rendimiento en un único servidor VPS, aislando completamente las bases de datos y orquestando el tráfico a través de un Proxy Inverso Maestro. La arquitectura se basa en las mejores prácticas de la industria y en una investigación exhaustiva sobre los componentes más eficientes.

## Arquitectura Multi-Sitio
La pila de software ha sido seleccionada cuidadosamente para aislar los entornos de los clientes sin sacrificar la velocidad:

1.  **Proxy Inverso Maestro (`nginx-proxy`):** Se encarga de escuchar los puertos 80 y 443 del servidor. Lee las peticiones entrantes y las enruta automáticamente al contenedor Nginx interno correcto basándose en la variable de entorno `VIRTUAL_HOST`.
2.  **Servidor Web Aislado (Nginx Interno):** Cada sitio tiene su propio contenedor Nginx que actúa de forma privada, sirviendo archivos estáticos y pasando llamadas PHP a su propio contenedor FPM.
3.  **Procesador PHP (PHP-FPM 8.3):** Cada sitio usa una imagen oficial de WordPress con PHP-FPM, aislado por completo del resto de sitios.
4.  **Bases de Datos Aisladas (MariaDB y Redis):** Cada sitio levanta su propia instancia de MariaDB (para base de datos relacional) y Redis (para caché de objetos) mediante volúmenes independientes.

## Requisitos Previos
Asegúrate de tener instaladas las siguientes herramientas en tu servidor:
*   [Docker](https://docs.docker.com/get-docker/)
*   [Docker Compose](https://docs.docker.com/compose/install/)

## Estructura de Archivos
El proyecto tiene la siguiente estructura orientada a plantillas:
```text
docker-woocommerce/
├── proxy/
│   └── docker-compose.yml   <-- El proxy maestro global
├── nginx/
│   └── default.conf         <-- Nginx interno preconfigurado
├── php-conf/
│   └── custom.ini           <-- Configuración PHP
├── wp-config.php            <-- Plantilla wp-config
├── docker-compose.yml       <-- Plantilla de la tienda
├── .env.example             <-- Plantilla de credenciales y dominio
└── README.md                <-- Este archivo
```

## Instalación y Configuración

Sigue estos pasos para poner en marcha tu entorno multi-sitio:

### Paso 1: Levantar el Proxy Maestro
Este paso **sólo se realiza una vez** en todo tu servidor VPS.
```bash
cd proxy
docker-compose up -d
```
El proxy maestro se quedará a la espera de que levantes nuevas tiendas.

### Paso 2: Crear una Nueva Tienda
Por cada tienda que quieras alojar en tu VPS, debes crear un clon o copiar los archivos base.
1.  **Copia los archivos:** Crea una carpeta nueva en tu servidor (ej. `/var/www/tienda-1`) y copia dentro todos los archivos (`docker-compose.yml`, carpeta `nginx/`, `.env.example`, etc.) excluyendo la carpeta `proxy/`.
2.  **Configura el entorno (`.env`):**
    Copia la plantilla y edítala:
    ```bash
    cp .env.example .env
    nano .env
    ```
    Configura el dominio (`VIRTUAL_HOST`) y las credenciales de base de datos (`DB_ROOT_PASSWORD`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`).
3.  **Inicia los servicios de la tienda:**
    ```bash
    docker-compose up -d
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
    docker-compose down
    ```
*   **Para ver los logs de una tienda:**
    ```bash
    docker-compose logs -f
    ```

## Consideraciones para Producción

*   **SSL/HTTPS:** Nginx-Proxy soporta Let's Encrypt usando un contenedor compañero llamado `nginx-proxy-acme`. Puedes agregarlo a la carpeta `proxy` para automatizar los certificados SSL.
*   **Copias de Seguridad:** Implementa rutinas para respaldar los volúmenes de Docker (`db_data` y `wordpress_data`) en cada tienda.
*   **Correo Electrónico:** Por defecto, PHP en Docker no envía correos. Usa plugins como WP Mail SMTP.

## Licencia
Este proyecto se distribuye bajo la licencia MIT.