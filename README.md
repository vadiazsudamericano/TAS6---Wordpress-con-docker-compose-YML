# Práctica: Wordpress con docker compose YML

## 1. Título
**Despliegue automatizado de una arquitectura de tres capas WordPress, MySQL y phpMyAdmin utilizando redes y volúmenes en Docker Compose.**



## 2. Tiempo de duración
90 minutos


## 3. Fundamentos

Para comprender el desarrollo de esta práctica, es indispensable dominar los conceptos clave de la tecnología de contenedores. A diferencia de la virtualización tradicional, donde cada máquina virtual (VM) requiere un hipervisor y un sistema operativo invitado (*Guest OS*) completo, la contenedorización permite que múltiples aplicaciones aisladas compartan el mismo núcleo (*kernel*) del sistema operativo anfitrión (Turnbull, 2014). De acuerdo con el análisis de Joy (2015), esta característica estructural elimina la sobrecarga de emular hardware físico, lo que se traduce en un consumo de recursos drásticamente menor, arranques casi instantáneos en cuestión de milisegundos y una portabilidad absoluta entre entornos de desarrollo, pruebas y entornos de producción.

### El Ecosistema Docker y sus Componentes

Docker opera bajo una arquitectura distribuida de tipo cliente-servidor. El componente central es el Docker Daemon dockerd, un servicio continuo en segundo plano que escucha las peticiones de la API de Docker y gestiona los objetos principales del sistema, como imágenes, contenedores, redes virtuales y volúmenes persistentes (Docker, s.f.). Una Imagen de Docker se define como una plantilla de solo lectura y de carácter inmutable que contiene las instrucciones y dependencias necesarias para instanciar una aplicación. Estas imágenes se estructuran mediante capas superpuestas basadas en un sistema de archivos de unión Union File System, optimizando el almacenamiento compartido a través de repositorios públicos centralizados como Docker Hub (Turnbull, 2014). Por su parte, un **Contenedor** es la instancia de ejecución viva, aislada y de lectura-escritura creada a partir de dicha imagen base.

### Orquestación Local con Docker Compose

Cuando una aplicación web evoluciona hacia una arquitectura distribuida de múltiples capas (como un frontend, un motor de base de datos relacional y herramientas de administración), la gestión manual e individual de cada contenedor mediante la interfaz de línea de comandos CLI se vuelve ineficiente y propensa a fallos operacionales. Docker Compose resuelve esta problemática al introducir el concepto de infraestructura como código IaC, permitiendo definir y coordinar aplicaciones multi-contenedor a través de un único archivo de configuración estructurado en formato YAML (`docker-compose.yml`) (Docker, s.f.). En este archivo se especifican de forma declarativa las propiedades de cada servicio, los mapeos de puertos de red, las credenciales internas y las directivas de ordenamiento en el arranque mediante la cláusula `depends_on`. En arquitecturas de sistemas de gestión de contenidos CMS como WordPress, esta automatización reduce drásticamente las discrepancias de entorno y los errores humanos en la sincronización de servicios en segundo plano (Warner & Harris, 2020).

### Redes y Volúmenes: Comunicación y Persistencia

Por defecto, el ciclo de vida de los contenedores se rige bajo un principio efímero e indeterminado; si un contenedor se detiene, se corrompe o se destruye, todas las mutaciones de datos generadas en su capa de escritura superior se pierden de forma irreversible. Para mitigar esta limitación estructural, Docker implementa Volúmenes, los cuales corresponden a directorios gestionados de forma nativa por el demonio de Docker que puentean un sector del sistema de archivos interno del contenedor con el almacenamiento físico de la máquina anfitriona, independizando los datos del ciclo de vida del proceso ejecutable (Turnbull, 2014).

En el plano de la conectividad, el aislamiento perimetral se delega a las Redes de Docker utilizando controladores virtuales, de los cuales el controlador tipo bridge es el estándar para despliegues en un mismo nodo. Al asociar múltiples servicios a una misma red privada virtualizada, Docker habilita de forma automática un servidor DNS interno para la resolución de nombres de servicio (Docker, s.f.). Este mecanismo permite que el contenedor de WordPress localice e interactúe con el motor de base de datos apuntando directamente al alias lógico (por ejemplo, `db:3306`) en lugar de depender de direccionamiento IP dinámico. Esto restringe cualquier tipo de exposición innecesaria hacia redes externas, consolidando un esquema robusto de seguridad perimetral dentro del ecosistema (Joy, 2015).
## 4. Conocimientos previos

Para realizar esta práctica de manera óptima, el estudiante necesita tener claros los siguientes temas:

- **Comandos básicos de Linux:** Navegación por directorios (`cd`, `mkdir`), gestión de archivos (`cat`, `ls`) y ejecución de scripts.
- **Formato YAML:** Reglas de indentación y estructuración de datos mediante espacios.
- **Conceptos de Redes:** Mapeo de puertos (`host:contenedor`), protocolos TCP/IP y funcionamiento básico de servicios web (HTTP/puerto 80).
- **Manejo del navegador:** Uso de herramientas de desarrollo y acceso a servicios mediante puertos específicos en entornos locales o virtuales.



## 5. Objetivos a alcanzar

- Implementar un entorno web multi-contenedor utilizando WordPress, MySQL y phpMyAdmin de forma integrada.
- Manipular archivos de configuración en formato YAML para estructurar de manera lógica la infraestructura.
- Configurar volúmenes independientes para asegurar la persistencia de datos de la aplicación y el motor de base de datos.
- Desplegar una red virtual tipo bridge para el aislamiento y comunicación segura entre los servicios.


## 6. Equipo necesario

- Computador con sistema operativo Windows.
- Conexión estable a Internet.
- Navegador web actualizado.
- Entorno interactivo virtualizado basado en la nube Killercoda con Docker y Docker Compose preinstalados.



## 7. Material de apoyo

- Documentación oficial de Docker:  
  https://docs.docker.com/

- Documentación de Docker Compose:  
  https://docs.docker.com/compose/

- Hoja de referencia de comandos Linux (Linux Cheat Sheet).



# 8. Procedimiento

## Paso 1: Creación del directorio de trabajo

Acceder a la terminal de Linux en el entorno de Killercoda y ejecutar el comando para crear una carpeta dedicada al proyecto con el fin de evitar conflictos de archivos. Posteriormente, ingresar al directorio creado:

```bash
mkdir mi-sitio-wp && cd mi-sitio-wp
```

## Paso 2: Creación del archivo de configuración YAML

Para estructurar el archivo sin depender de un editor gráfico, ejecutar el flujo de entrada `cat << 'EOF'` en la terminal. Copiar y pegar el bloque de configuración completo que define los tres servicios, los volúmenes y la red virtual:

```bash
cat << 'EOF' > docker-compose.yml
version: '3.8'

services:
  db:
    image: mysql:8.0
    container_name: wp_mysql_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 090306
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: 090306
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - wp_network

  wordpress:
    image: wordpress:latest
    container_name: wp_site
    restart: always
    depends_on:
      - db
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: 090306
      WORDPRESS_DB_NAME: wordpress_db
    volumes:
      - wp_data:/var/www/html
    networks:
      - wp_network

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: wp_phpmyadmin
    restart: always
    depends_on:
      - db
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: 090306
    networks:
      - wp_network

volumes:
  db_data:
  wp_data:

networks:
  wp_network:
    driver: bridge
EOF
```


## Paso 3: Verificación de la escritura del archivo

Confirmar que el archivo se guardó íntegramente imprimiendo su contenido en la pantalla de la terminal con el comando:

```bash
cat docker-compose.yml
```


## Paso 4: Despliegue de los contenedores

Ejecutar el comando de Docker Compose adaptado a la sintaxis del entorno de Killercoda, añadiendo el parámetro `-d` para delegar la ejecución al segundo plano:

```bash
docker-compose up -d
```

Esperar a que finalice la descarga de capas de las imágenes desde Docker Hub hasta que la terminal indique el estado iniciado de cada servicio.


## Paso 5: Diagnóstico y verificación del estado operacional

Comprobar el estado físico de los contenedores y el mapeo correcto de los puertos mediante el comando de control de Docker:

```bash
docker ps
```

## Paso 6: Configuración del CMS WordPress

Abrir la utilidad de gestión de puertos expuestos de Killercoda Traffic Ports, ingresar el puerto `8080` y proceder con la selección de idioma y creación del usuario administrador del sitio web dentro de la interfaz gráfica cargada.

## Paso 7: Verificación del motor de base de datos MySQL

Abrir una nueva pestaña de tráfico utilizando el puerto `8081` para cargar phpMyAdmin. Autenticarse utilizando el usuario `root` y la clave establecida para verificar la generación automática de las tablas dentro del esquema `wordpress_db`.


# 9. Resultados esperados

Al finalizar la práctica se espera que el estudiante demuestre el correcto despliegue del ecosistema técnico adjuntando las siguientes capturas:

### Captura 1 (Terminal)
Salida del comando `docker ps` que liste con éxito los tres contenedores en estado activo:
![Captura de terminal de Killercoda](terminal2.png)

### Captura 2 (Navegador - Puerto 8080)

![Captura Dashboard de Wordpress](wordpress.png)
### Captura 3 (Navegador - Puerto 8081)
![Captura php myadmin](php.png)


# 10. Bibliografía

Docker. (s.f.). *Docker Compose overview*. Docker Documentation. https://docs.docker.com/compose/

Joy, A. M. (2015). Performance comparison of OS-level virtualization framework for cloud computing. *2015 International Conference on Control Communication & Computing India (ICCC)*, 466-470. https://doi.org/10.1109/ICCC.2015.7432936

Turnbull, J. (2014). *The Docker Book: Containerization is the new virtualization*. James Turnbull.

Warner, J., & Harris, S. (2020). *WordPress All-in-One For Dummies*. John Wiley & Sons.
