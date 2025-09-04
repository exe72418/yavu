
````markdown
# Carpeta del proyecto principal

Esta carpeta contiene el código fuente de la aplicación principal, incluyendo los servicios de backend, el frontend (usando Backstage) y la configuración de las dependencias.

## Estructura del proyecto

El código de la aplicación se organiza en una estructura monorepo, con las siguientes subcarpetas principales:

-   `packages/app`: Contiene el código fuente de la interfaz de usuario de Backstage.
-   `packages/backend`: Contiene la lógica del servidor, APIs y los servicios de backend.
-   `app-config.*.yaml`: Archivos de configuración para distintos entornos (desarrollo, producción, etc.).

## Cómo empezar

Para ejecutar este proyecto, no es necesario hacer un "build" manual, ya que los archivos se generan automáticamente dentro del contenedor de Docker.

**1. Instala las dependencias:**

Usa `yarn` para instalar las dependencias del proyecto. Este paso es crucial para que Docker pueda construir la imagen correctamente.

```bash
yarn install
````

**2. Inicia los servicios con Docker Compose:**

Asegúrate de estar en el directorio raíz del proyecto (`../`) donde se encuentra el archivo `docker-compose.yml`. El comando a continuación levantará todos los servicios de la aplicación (incluyendo PostgreSQL, Jenkins, etc.). La primera vez, usá la bandera `--build` para generar los archivos compilados que Git ignora.

```bash
docker-compose up --build
```

**3. Accede a la aplicación:**

Una vez que los servicios estén listos, puedes acceder a la aplicación en la URL que se indica en la documentación principal.

