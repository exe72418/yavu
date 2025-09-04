# 🐘 Guía Completa para Administrar PostgreSQL en Docker Compose

Este documento es un manual de referencia para conectar, manipular, configurar y respaldar la base de datos PostgreSQL que se ejecuta como un servicio dentro de nuestro entorno de `docker-compose`.

---
## Principio Fundamental: Contenedores y Datos Persistentes

Antes de ejecutar cualquier comando, es crucial entender cómo Docker maneja los datos de nuestra base de datos.

- **El Contenedor es Temporal:** El contenedor `db` que ejecuta PostgreSQL es efímero. Si se borra, todo lo que está *dentro* de su sistema de archivos se pierde.
- **El Volumen es Permanente:** Para evitar la pérdida de datos, usamos un **volumen de Docker** (`postgres_data`). Piensa en él como un disco duro externo que se conecta al contenedor. Los datos de la base de datos se escriben en este volumen, que reside de forma segura en tu máquina (ya sea tu PC o el servidor).

El archivo `docker-compose.yml` define esta conexión:

```yaml
services:
  db:
    # ...
    volumes:
      # Conecta el volumen 'postgres_data' con la carpeta de datos interna de PostgreSQL
      - postgres_data:/var/lib/postgresql/data

# Aquí se declara que Docker debe gestionar el volumen 'postgres_data'
volumes:
  postgres_data:
```

---
## 🔌 Conexión y Manipulación de Datos

Hay dos formas principales de interactuar con la base de datos: a través de la terminal o con un cliente gráfico.

### 1. Acceder a la Consola `psql` (Vía Terminal)

`psql` es la herramienta de línea de comandos oficial de PostgreSQL. Para usarla, ejecutamos un comando "dentro" del contenedor `db`.

**Comando de Conexión:**
```bash
sudo docker-compose exec db psql -U sonar
```

**Desglose:**
- `sudo docker-compose exec`:  Indica que vamos a ejecutar un comando en un servicio que ya está corriendo.
- `db`: Es el nombre de nuestro servicio de base de datos.
- `psql -U sonar`: Es el comando a ejecutar dentro del contenedor. Inicia la consola `psql` como el usuario (`-U`) `sonar`.

Una vez ejecutado, tu terminal cambiará a `sonar=#`, indicando que estás conectado y listo para enviar comandos SQL.

### 2. "Chuleta" de Comandos `psql`

Una vez dentro de `sonar=#`, estos son los comandos más útiles:

| Comando | Descripción | Ejemplo |
| :--- | :--- | :--- |
| `\l` | **L**ista todas las bases de datos. | `\l` |
| `\c <db>` | Se **c**onecta a otra base de datos. | `\c backstage` |
| `\dt` | Muestra las **t**ablas de la base de datos actual. | `\dt` |
| `\d <table>`| **D**escribe la estructura de una tabla específica. | `\d users` |
| `SELECT` | Consulta datos. **¡Recuerda el punto y coma!** | `SELECT * FROM users;` |
| `\q` | **Q**uit (Salir) de `psql` y volver a la terminal. | `\q` |

### 3. Conexión con un Cliente Gráfico (DBeaver, DataGrip, etc.)

Puedes usar tu herramienta de base de datos preferida para conectarte, ya que el puerto `5432` del contenedor está disponible para tu máquina.

Usa los siguientes parámetros de conexión:
- **Host:** `localhost`
- **Puerto:** `5432` (o el que hayas definido en la sección `ports` de `db` si lo añades)
- **Base de Datos:** `sonar` o `backstage`
- **Usuario:** `sonar`
- **Contraseña:** `sonar`

---
## ⚙️ Configuración Avanzada del Servidor PostgreSQL

Para cambiar la configuración del motor de PostgreSQL (ej. `postgresql.conf`), la mejor práctica es montar un archivo de configuración personalizado desde tu máquina local, en lugar de modificar archivos dentro del contenedor.

**Paso 1: Crear un Archivo de Configuración Local**

En tu carpeta `servidores/`, crea un archivo `postgres_custom.conf`.

*Ejemplo de `postgres_custom.conf`:*
```ini
# Aumenta el número máximo de conexiones
max_connections = 250

# Aumenta la memoria compartida para mejorar el rendimiento
shared_buffers = 512MB

# Habilita el registro de todas las consultas (útil para depurar)
log_statement = 'all'
```

**Paso 2: Modificar `docker-compose.yml` para Usar el Archivo**

Edita el servicio `db` para que monte este nuevo archivo y le indique a PostgreSQL que lo use al arrancar.

```yaml
# En docker-compose.yml
services:
  db:
    image: postgres:13
    container_name: postgres_db
    environment:
      - POSTGFRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
    volumes:
      - postgres_data:/var/lib/postgresql/data
      # Montamos nuestro archivo de configuración personalizado
      - ./postgres_custom.conf:/etc/postgresql/custom.conf
    # Le decimos a postgres que use nuestro archivo al arrancar
    command: postgres -c config_file=/etc/postgresql/custom.conf
    networks:
      - servidores-net
```

**Paso 3: Aplicar los Cambios**

Para que el contenedor de la base de datos cargue la nueva configuración, debes forzar su recreación:

```bash
sudo docker-compose up -d --force-recreate db
```
Tus datos no se perderán, ya que están seguros en el volumen `postgres_data`.

---
## 💾 Respaldo y Restauración de Datos (Backup & Restore)

Esta es una de las tareas más críticas. Los comandos se ejecutan desde tu PC/servidor.

### Crear un Respaldo (`pg_dump`)

`pg_dump` es la herramienta estándar de PostgreSQL para crear un archivo de respaldo (`.sql` o `.dump`) de una base de datos completa.

```bash
# Comando para respaldar la base de datos de SonarQube
sudo docker-compose exec -T db pg_dump -U sonar -d sonar > backup_sonar_$(date +%F).sql

# Comando para respaldar la base de datos de Backstage
sudo docker-compose exec -T db pg_dump -U sonar -d backstage > backup_backstage_$(date +%F).sql
```
- La opción `-T` es importante para que Docker no cree una terminal interactiva, lo que permite que la salida se redirija a un archivo correctamente.

### Restaurar un Respaldo

Para restaurar, hacemos el proceso inverso: leemos un archivo `.sql` y lo "tubería" (`|`) hacia el comando `psql` dentro del contenedor.

> **Advertencia:** Esto puede sobrescribir datos existentes en la base de datos. Asegúrate de restaurar en una base de datos limpia o de saber lo que estás haciendo.

```bash
# Comando para restaurar la base de datos de SonarQube desde un archivo
cat backup_sonar_2025-09-03.sql | sudo docker-compose exec -T db psql -U sonar -d sonar
```
