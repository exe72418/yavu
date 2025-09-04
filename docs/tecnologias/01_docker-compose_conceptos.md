# 🐳 Guía Definitiva de Docker y Docker Compose

Este documento es un manual conceptual y práctico sobre Docker y Docker Compose, utilizando nuestro proyecto (`Jenkins`, `SonarQube`, `Backstage`, etc.) como ejemplo principal.

---
## Parte 1: Docker - El Contenedor Individual 📦

### ¿Qué es Docker y por qué es tan popular?

Imagina que tu aplicación (por ejemplo, Backstage) es un objeto frágil que necesitas enviar a otra ciudad (otro PC o un servidor).

- **Antes de Docker:** Tenías que conseguir una caja, envolver la aplicación con mucho cuidado, y luego asegurarte de que el destinatario tuviera exactamente el mismo tipo de papel de burbujas, cinta adhesiva y herramientas para abrirlo (la misma versión de Node.js, las mismas librerías del sistema, etc.). Si algo era diferente, el objeto llegaba roto.

- **Con Docker:** Docker te da un **contenedor de envío estandarizado**. Metes tu aplicación y **absolutamente todo lo que necesita para funcionar** (Node.js, librerías, dependencias, configuraciones) dentro de este contenedor. Ahora, puedes enviar este contenedor a cualquier lugar que tenga Docker instalado, y tienes la **garantía del 100% de que funcionará exactamente igual**.

> **Beneficio Principal:** Soluciona para siempre el clásico problema de **"en mi máquina sí funciona"**. La consistencia entre el entorno de desarrollo, pruebas y producción es total.

### Conceptos Fundamentales de Docker

#### 1. Imagen (La Receta o el Molde)
Una imagen es un paquete ligero, inmutable y ejecutable que incluye todo lo necesario para correr una aplicación: el código, el entorno de ejecución (como Node.js o Java), las librerías, las variables de entorno y los archivos de configuración.

- **Es una plantilla de solo lectura.** No se puede modificar.
- Se construye a partir de un archivo de instrucciones llamado `Dockerfile`.
- **Analogía:** Es el **plano de una casa prefabricada**.

#### 2. Contenedor (La Instancia en Ejecución)
Un contenedor es una **instancia en ejecución de una imagen**. Es la materialización de la plantilla.

- Puedes crear, iniciar, detener, mover y eliminar contenedores.
- Están aislados entre sí y del sistema anfitrión, pero comparten el kernel del sistema operativo.
- **Analogía:** Es la **casa real y funcional** que construiste a partir del plano. Puedes construir muchas casas idénticas con el mismo plano.

#### 3. Volumen (La Caja de Seguridad)
Por defecto, los datos dentro de un contenedor se pierden cuando este se elimina. Un volumen es un mecanismo para **persistir los datos** fuera del ciclo de vida del contenedor.

- Se gestionan por Docker y se almacenan en una parte del disco duro del anfitrión.
- **Analogía:** Es la **cimentación y el sótano de la casa**. Aunque demuelas y reconstruyas la casa, la cimentación y todo lo que guardaste en el sótano permanece.

#### 4. Red (La Red Vecinal Privada)
Docker permite crear redes virtuales para que los contenedores puedan comunicarse entre sí de forma aislada y segura.

- Dentro de la misma red, los contenedores pueden llamarse usando el nombre del servicio (ej. `backstage` puede conectarse a `db`).
- **Analogía:** Es la **red de telefonía interna** de un barrio cerrado. Todos los vecinos (contenedores) pueden llamarse entre sí fácilmente, pero están aislados del exterior.

---
## Parte 2: Docker Compose - El Director de Orquesta 🎼

### ¿Por qué necesitamos Docker Compose?

Una aplicación moderna rara vez es un solo servicio. Nuestro proyecto es el ejemplo perfecto: necesitamos un servidor CI/CD (Jenkins), un analizador de código (SonarQube), un portal (Backstage) y una base de datos (PostgreSQL).

Gestionar 4 o 5 contenedores por separado con comandos `docker run` sería increíblemente complejo y propenso a errores.

**Docker Compose** soluciona esto. Es una herramienta que utiliza un único archivo de configuración (`docker-compose.yml`) para definir, configurar y ejecutar una aplicación multi-contenedor completa con un solo comando.

### Anatomía de un Archivo `docker-compose.yml`

Este archivo es el **plano maestro de todo nuestro entorno**. Analicemos sus partes clave usando nuestro propio archivo como ejemplo:

```yaml
version: '3.8' # Versión de la sintaxis de Docker Compose

services: # Aquí se definen todos nuestros contenedores
  jenkins: # Nombre del servicio. También es su "hostname" en la red interna.
    image: jenkins/jenkins:lts-jdk17 # Usa una imagen pre-hecha de Docker Hub.
    build: ./yavu # Alternativa a 'image'. Construye una imagen desde un Dockerfile local.
    container_name: jenkins # Un nombre amigable para el contenedor.
    ports:
      - "8081:8080" # Mapea el puerto 8081 de tu PC al puerto 8080 del contenedor.
    volumes:
      - jenkins_home:/var/jenkins_home # Conecta el volumen 'jenkins_home' a la carpeta de datos de Jenkins.
    environment:
      - SONAR_JDBC_URL=... # Define variables de entorno dentro del contenedor.
    depends_on:
      - db # No inicies este servicio hasta que el servicio 'db' esté en marcha.
    networks:
      - servidores-net # Conecta este servicio a nuestra red privada.

volumes: # Declaración de todos los volúmenes "caja de seguridad" que usaremos.
  jenkins_home:
  postgres_data:
  # ...

networks: # Declaración de nuestras redes privadas.
  servidores-net:
    driver: bridge
```

---
## 🛠️ Comandos Esenciales (La Chuleta Definitiva)

Todos estos comandos se ejecutan desde la carpeta donde está tu `docker-compose.yml` (`Servidores/`).

| Comando                                         | Descripción                                                                                                                    |
| :---------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------- |
| `sudo docker-compose up`                        | Levanta todos los servicios y **ata la terminal** a los logs. Si la cierras, los contenedores se detienen. Ideal para depurar. |
| `sudo docker-compose up -d`                     | Levanta todos los servicios en **modo "detached"** (segundo plano). Tu terminal queda libre. Es el modo normal de operación.   |
| `sudo docker-compose down`                      | **Detiene y elimina** todos los contenedores, redes y volúmenes (si se especifica con `-v`) definidos en el archivo.           |
| `sudo docker-compose ps`                        | **Muestra el estado** actual de todos los servicios del proyecto (si están corriendo, detenidos, etc.).                        |
| `sudo docker-compose logs`                      | Muestra los **logs acumulados** de todos los servicios.                                                                        |
| `sudo docker-compose logs -f <servicio>`        | Muestra los logs de un **servicio específico en tiempo real**. (Ej: `... logs -f jenkins`).                                    |
| `sudo docker-compose exec <servicio> <comando>` | **Ejecuta un comando** dentro de un contenedor que ya está corriendo. (Ej: `... exec db psql -U sonar`).                       |
| `sudo docker-compose build`                     | **Fuerza la reconstrucción** de las imágenes que tienen una sección `build` (como Backstage), si hiciste cambios.              |
| `sudo docker-compose pull`                      | **Descarga las versiones más recientes** de las imágenes que no se construyen localmente (como Jenkins, SonarQube).            |
| `sudo docker-compose stop`                      | **Detiene** los contenedores sin eliminarlos.                                                                                  |
| `sudo docker-compose start`                     | **Inicia** contenedores que estaban detenidos.                                                                                 |


---

## Parte 3: Manipulación Avanzada y Flujo de Desarrollo

Para un flujo de trabajo de desarrollo eficiente y profesional, es clave entender cómo personalizar y depurar tu entorno Docker. Esta sección te da las herramientas para manipular los servicios de manera avanzada.

### 1. Variables de Entorno con `.env`

Para mantener tu `docker-compose.yml` limpio y reutilizable, es una práctica común usar un archivo **`.env`** para definir variables de entorno que el archivo YAML puede leer. Esto te permite cambiar configuraciones importantes, como los puertos o las contraseñas, sin modificar el archivo principal.

**¿Cómo funciona?**

1. Crea un archivo llamado `.env` en la misma carpeta que tu `docker-compose.yml`.
    
2. Agrega tus variables en el formato `CLAVE=VALOR`, por ejemplo:
    
    Ini, TOML
    
    ```
    # .env
    JENKINS_PORT=8081
    ```
    
3. En tu `docker-compose.yml`, referencia la variable con `${...}`:
    
    YAML
    
    ```
    # docker-compose.yml
    services:
      jenkins:
        ...
        ports:
          - "${JENKINS_PORT}:8080"
    ```
    

Ahora, puedes cambiar el puerto de Jenkins modificando solo el archivo `.env`.

### 2. Desarrollo en Vivo con Volúmenes de Código

Para no tener que reconstruir la imagen de tu aplicación (como Backstage) cada vez que haces un cambio, puedes montar tu código local dentro del contenedor. Este **volumen de código** sincroniza los archivos de tu PC con los del contenedor en tiempo real.

YAML

```
# En el servicio de Backstage
services:
  backstage:
    build: ./yavu
    volumes:
      - ./yavu:/app # Monta la carpeta de tu código local en la carpeta /app del contenedor
      - /app/node_modules # **Importante:** Esta línea evita que los archivos de tu PC sobreescriban las dependencias instaladas en el contenedor.
```

### 3. Entendiendo la Comunicación entre Contenedores

Dentro de una red de Docker Compose, cada servicio puede comunicarse con otro usando el **nombre del servicio como un `hostname`**.

Por ejemplo, si el servicio `backstage` necesita conectarse a la base de datos `postgres` (que en tu `docker-compose.yml` tiene el nombre de servicio `db`), la URL de conexión que usaría sería:

`postgresql://db:5432/backstage`

**Recuerda:** No uses `localhost` o `127.0.0.1` para la comunicación entre contenedores, ya que se refieren a la máquina interna de cada contenedor, no a los demás servicios en la red.

---

## 🔍 Comandos Adicionales para la Depuración y Gestión

Esta es una lista de comandos útiles para cuando necesitas ir más allá de levantar y detener los servicios.

|Comando|Descripción|
|---|---|
|`sudo docker-compose exec <servicio> /bin/bash`|**Abre una terminal interactiva** dentro de un contenedor en ejecución. Crucial para depuración. (Ej: `... exec jenkins /bin/bash`).|
|`sudo docker-compose restart <servicio>`|**Reinicia** un servicio específico.|
|`sudo docker inspect <nombre_contenedor>`|**Muestra toda la información** de bajo nivel de un contenedor (variables de entorno, IP, volúmenes, etc.).|
|`sudo docker system prune`|**Limpia** recursos no utilizados de Docker, como imágenes, contenedores y volúmenes. Muy útil para liberar espacio en disco.|
