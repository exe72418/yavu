# 🤖 Guía Completa para Administrar Jenkins en Docker Compose

Este manual es una referencia completa para interactuar, configurar, personalizar y respaldar el servicio de Jenkins que se ejecuta como un contenedor en nuestro entorno de `docker-compose`.

---
## El Corazón de Jenkins: El Volumen `jenkins_home`

Al igual que la base de datos, la clave para que Jenkins no pierda información es la **persistencia de datos a través de un volumen de Docker**.

- **El Contenedor es Desechable:** El contenedor `jenkins` puede ser eliminado, actualizado o recreado en cualquier momento.
- **El Volumen es Permanente:** El volumen `jenkins_home` es donde Jenkins guarda absolutamente todo:
    - Jobs y Pipelines
    - Plugins instalados
    - Configuraciones globales
    - Credenciales y secretos
    - Historial de builds

La conexión se define en el `docker-compose.yml`:
```yaml
services:
  jenkins:
    # ...
    volumes:
      # Conecta el volumen 'jenkins_home' con la carpeta de datos interna de Jenkins
      - jenkins_home:/var/jenkins_home
```

---
## 🔌 Interactuando con el Contenedor de Jenkins

### 1. Acceder a la Shell del Contenedor

Si necesitas explorar los archivos internos de Jenkins o ejecutar comandos de diagnóstico, puedes acceder a su terminal (shell).

```bash
sudo docker-compose exec jenkins bash
```
**Desglose:**
- `exec`: Ejecuta un comando en un servicio en marcha.
- `jenkins`: El nombre de nuestro servicio.
- `bash`: El comando para iniciar una sesión de terminal interactiva.

Tu prompt cambiará a algo como `jenkins@<id_contenedor>:/$`, indicando que estás dentro del contenedor. Para salir, simplemente escribe `exit`.

### 2. Obtener la Contraseña de Administrador Inicial

Cuando Jenkins arranca por primera vez, genera una contraseña de administrador que necesitas para la configuración inicial. Hay dos formas de obtenerla:

**Método A: Revisando los Logs**
Este método es ideal justo después de levantar el contenedor por primera vez.
```bash
sudo docker-compose logs jenkins
```
Busca en la salida un bloque de texto rodeado de asteriscos que contiene la contraseña.

**Método B: Leyendo el Archivo Directamente (Más Rápido)**
Si el contenedor ya lleva un tiempo funcionando, este comando es más directo:
```bash
sudo docker-compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
Esto mostrará la contraseña directamente en tu terminal.

---
## ⚙️ Configuración y Personalización (Infraestructura como Código)

La mejor manera de configurar Jenkins en Docker no es haciendo clic en la interfaz, sino definiendo la configuración como código. Así, tu instancia de Jenkins será siempre reproducible.

### 1. Instalación de Plugins Automática

Puedes decirle a Jenkins qué plugins instalar automáticamente al arrancar.

**Paso 1: Crear un archivo `plugins.txt`**
En tu carpeta `Servidores/jenkins/`, crea un archivo `plugins.txt` con la lista de plugins que deseas. Puedes encontrar el "ID corto" de cada plugin en su página oficial.

*Ejemplo de `servidores/jenkins/plugins.txt`:*
```
# Plugins esenciales
git:latest
pipeline-stage-view:latest
workflow-aggregator:latest

# Para una mejor interfaz de pipelines
blueocean:latest

# Para integrar con Docker
docker-workflow:latest
```

**Paso 2: Crear un `Dockerfile` para Jenkins**
En la misma carpeta `Servidores/jenkins/`, crea un archivo llamado `Dockerfile`.
```dockerfile
# servidores/jenkins/Dockerfile
FROM jenkins/jenkins:lts-jdk17

# Copia el archivo de plugins al lugar donde Jenkins lo buscará al iniciar
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt

# Ejecuta la herramienta de instalación de plugins
RUN jenkins-plugin-cli -f /usr/share/jenkins/ref/plugins.txt
```

**Paso 3: Modificar `docker-compose.yml`**
Finalmente, ajusta el servicio `jenkins` en tu `docker-compose.yml` para que construya esta imagen personalizada en lugar de usar la oficial directamente.

```yaml
# En docker-compose.yml
services:
  jenkins:
    build: ./jenkins  # <-- CAMBIO: Ahora construye desde la carpeta local
    # image: jenkins/jenkins:lts-jdk17 <-- Esta línea se comenta o elimina
    container_name: jenkins
    ports:
      - "8081:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
    networks:
      - servidores-net
```
La próxima vez que ejecutes `docker-compose up --build`, se creará una imagen de Jenkins con todos tus plugins ya instalados.

---
## 💾 Respaldo y Restauración de Jenkins

El proceso es idéntico al de la base de datos, pero apuntando al volumen de Jenkins. El respaldo contendrá **toda la configuración** de Jenkins.

### Crear un Respaldo (Backup)

1.  **Detén el servicio de Jenkins** para asegurar que no se escriban datos durante el respaldo.
    ```bash
    sudo docker-compose stop jenkins
    ```
2.  **Encuentra la ruta física del volumen** (opcional, pero útil saberlo).
    ```bash
    sudo docker volume inspect servidores_jenkins_home
    ```
3.  **Comprime todo el contenido del volumen** en un archivo.
    ```bash
    # Reemplaza la ruta con la que obtuviste en el paso anterior
    sudo tar -czvf backup_jenkins_$(date +%F).tar.gz /var/lib/docker/volumes/servidores_jenkins_home/_data
    ```
4.  **Vuelve a iniciar Jenkins.**
    ```bash
    sudo docker-compose start jenkins
    ```

### Restaurar un Respaldo

1. Detén el servicio: `sudo docker-compose stop jenkins`.
2. Borra (o renombra) el contenido actual del volumen.
3. Descomprime tu archivo de respaldo (`.tar.gz`) en la carpeta del volumen.
4. Inicia el servicio: `sudo docker-compose start jenkins`.
