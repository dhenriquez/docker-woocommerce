## Pila de Servidor WooCommerce de Alto Rendimiento con Docker
Repositorio del Proyecto: https://github.com/dhenriquez/docker-woocommerce

Este proyecto proporciona una configuración de Docker Compose para desplegar una pila de servidor optimizada para WooCommerce, diseñada para ofrecer alto rendimiento, escalabilidad y un manejo impecable de las llamadas AJAX. La arquitectura se basa en las mejores prácticas de la industria y en una investigación exhaustiva sobre los componentes más eficientes para tiendas de comercio electrónico de alto tráfico.

## Arquitectura
La pila de software ha sido seleccionada cuidadosamente para maximizar la velocidad y la eficiencia en cada capa, desde el servidor web hasta la base de datos y el almacenamiento en caché.

## Servidor Web: Nginx
Se eligió Nginx por su arquitectura asíncrona y dirigida por eventos, que es fundamentalmente superior para manejar una gran cantidad de conexiones concurrentes con un bajo consumo de memoria. Esto es ideal para las numerosas y rápidas peticiones AJAX de una tienda WooCommerce activa.

## Procesador PHP: PHP-FPM
Utilizamos la imagen oficial de WordPress con PHP-FPM (FastCGI Process Manager) para que Nginx actúe como un proxy inverso de alto rendimiento. Se ha seleccionado una versión reciente de PHP (8.3) para aprovechar las últimas mejoras de rendimiento y seguridad.

## Base de Datos: MariaDB
MariaDB es un reemplazo directo y altamente compatible de MySQL, a menudo con ligeras ventajas de rendimiento. Es una opción robusta y recomendada oficialmente para WordPress y WooCommerce.   

## Caché de Objetos: Redis
Esta es la optimización más crítica para la velocidad de las operaciones dinámicas. Redis actúa como un caché de objetos persistente, almacenando los resultados de las consultas a la base de datos en la memoria RAM. Esto reduce drásticamente la carga sobre la base de datos y acelera significativamente las respuestas de la API y las llamadas AJAX, como la actualización del carrito.

## Requisitos Previos
Asegúrate de tener instaladas las siguientes herramientas en tu sistema:

(https://docs.docker.com/get-docker/)

(https://docs.docker.com/compose/install/)

## Estructura de Archivos
El proyecto tiene la siguiente estructura de directorios:
docker-woocommerce/
├── docker-compose.yml
├── nginx/
│   └── default.conf
└── README.md


## Instalación y Configuración

Sigue estos pasos para poner en marcha tu entorno:

1.  **Clona el repositorio:**
    Abre tu terminal y clona el repositorio del proyecto desde GitHub:
    ```bash
    git clone [https://github.com/dhenriquez/docker-woocommerce.git](https://github.com/dhenriquez/docker-woocommerce.git)
    cd docker-woocommerce
    ```

2.  **Personaliza las credenciales:**
    Abre el archivo `docker-compose.yml` y **cambia las contraseñas por defecto** por valores seguros y únicos. Busca y reemplaza:

    *   `tu_password_root_muy_seguro`
    *   `tu_password_db_seguro`

3.  **Configura tu dominio:**
    Abre el archivo `nginx/default.conf` y reemplaza `tu-dominio.com www.tu-dominio.com` con tu dominio real. Para desarrollo local, puedes usar `localhost`.

4.  **Inicia los servicios:**
    En el directorio raíz del proyecto, ejecuta el siguiente comando para construir e iniciar los contenedores en segundo plano:

    ```bash
    docker-compose up -d
    ```

## Configuración Post-Instalación en WordPress

Una vez que los contenedores estén en funcionamiento, completa la configuración desde el panel de WordPress:

1.  **Instala WordPress:**
    Abre tu navegador y navega a `http://localhost` (o tu dominio). Sigue las instrucciones del asistente de instalación de WordPress.

2.  **Instala el plugin de Caché de Redis:**
    *   En el panel de administración de WordPress, ve a **Plugins > Añadir nuevo**.
    *   Busca, instala y activa el plugin **"Redis Object Cache"**.

3.  **Habilita el Caché de Objetos:**
    *   Ve a **Ajustes > Redis**.
    *   Haz clic en el botón **"Enable Object Cache"**.
    *   Debería conectarse correctamente y mostrar el estado "Connected". Las credenciales de conexión ya están preconfiguradas a través de las variables de entorno en el archivo `docker-compose.yml`.

¡Listo! Tu tienda WooCommerce ahora está funcionando sobre una pila de servidor optimizada para alto rendimiento.

## Gestión de los Contenedores

*   **Para detener los servicios:**
    ```bash
    docker-compose down
    ```
*   **Para ver los logs en tiempo real (ej. para Nginx):**
    ```bash
    docker-compose logs -f nginx
    ```
*   **Para reconstruir las imágenes después de un cambio:**
    ```bash
    docker-compose up -d --build
    ```

## Consideraciones para Producción

*   **SSL/HTTPS:** La configuración de Nginx proporcionada incluye una sección comentada para HTTPS. Para producción, es **esencial** que habilites SSL. Deberás obtener certificados SSL (por ejemplo, usando Let's Encrypt / Certbot), colocarlos en una carpeta (`ssl/`) y descomentar la sección del servidor en el puerto 443 en `nginx/default.conf`.

*   **Copia de Seguridad (Backups):** Esta configuración utiliza volúmenes de Docker para persistir los datos de la base de datos (`db_data`) y los archivos de WordPress (`wordpress_data`). Es **crítico** que implementes una estrategia de copia de seguridad robusta para estos volúmenes para evitar la pérdida de datos.

*   **Afinación de PHP:** Para un control más granular, puedes crear un archivo `php.ini` personalizado y montarlo en el contenedor de WordPress para ajustar directivas como `memory_limit`, `upload_max_filesize`, y `max_execution_time` según las necesidades específicas de tus plugins.[1, 3]

*   **Correo Electrónico:** Por defecto, este entorno no está configurado para enviar correos electrónicos transaccionales (confirmaciones de pedido, etc.). Deberás configurar un servicio SMTP a través de un plugin de WordPress (como WP Mail SMTP) para asegurar una entrega de correo fiable.

## Licencia

Este proyecto se distribuye bajo la licencia MIT.