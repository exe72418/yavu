# Entorno de Desarrollo Integral con Docker 🚀

Este proyecto configura un entorno de desarrollo y CI/CD completo utilizando Docker Compose. Con un solo comando, puedes levantar los siguientes servicios interconectados:

* **Jenkins:** Para automatización y pipelines de integración continua.
* **SonarQube:** Para análisis estático de código y control de calidad.
* **Backstage:** Un portal para desarrolladores autocontenido.
* **Cypress:** Para ejecutar pruebas End-to-End.
* **PostgreSQL:** Como base de datos para SonarQube y Backstage.

---

## ✅ Requisitos Previos

Antes de empezar, asegúrate de tener instalado el siguiente software en tu PC:

1.  **Git:** Para clonar el repositorio.
2.  **Docker:** El motor para ejecutar los contenedores.
3.  **Docker Compose:** La herramienta para orquestar los contenedores (generalmente viene incluida con Docker Desktop).

---

## ⚙️ Configuración Inicial (Solo la primera vez)

Antes de levantar el entorno, necesitas realizar dos configuraciones iniciales.

### 1. Configuración de Memoria para SonarQube (Solo en Linux)

SonarQube requiere un ajuste en la configuración de memoria virtual del sistema operativo anfitrión (host).

```bash
sudo sysctl -w vm.max_map_count=262144
```
**Nota:** Este comando debe ejecutarse cada vez que reinicies tu PC.

### 2. Instalación de Dependencias de Backstage

El proyecto de Backstage (`yavu/`) necesita que sus dependencias sean instaladas antes de poder ser empaquetado para Docker.

```bash
# Navega a la carpeta de Backstage, instala y vuelve a salir
cd yavu && yarn install && cd ..
```

---

## 🚀 Cómo Levantar el Entorno

Sigue estos pasos desde la carpeta raíz del proyecto (`Servidores/`).

### Paso 1: Clonar el Repositorio (En una nueva PC)

Si estás en una máquina nueva, clona el repositorio desde GitHub:

```bash
git clone [https://github.com/exe72418/yavu.git](https://github.com/exe72418/yavu.git)
cd yavu # O el nombre que le hayas dado a la carpeta
```

### Paso 2: Preparar el Paquete de Backstage

Este es el paso más importante. Genera el paquete de producción de Backstage, lo que también crea su `Dockerfile` automáticamente.

```bash
# Estando en la raíz del proyecto (Servidores/)
cd yavu && yarn workspace backend bundle && cd ..
```

### Paso 3: Levantar Todos los Servicios

Con todo preparado, ejecuta Docker Compose. Este comando construirá las imágenes necesarias y levantará todos los contenedores en segundo plano.

```bash
sudo docker-compose up -d --build
```
La primera vez puede tardar varios minutos mientras se descargan y construyen las imágenes.

---

## 🖥️ Acceso a los Servicios

Una vez que todo esté corriendo, puedes acceder a tus herramientas desde el navegador:

* **Jenkins:** ➡️ **`http://localhost:8081`**
    * *Nota:* La primera vez, busca la contraseña inicial en los logs con: `sudo docker-compose logs jenkins`.

* **SonarQube:** ➡️ **`http://localhost:9000`**
    * *Nota:* Puede tardar unos minutos en estar completamente operativo.
    * *Credenciales por defecto:* `admin` / `admin`.

* **Backstage:** ➡️ **`http://localhost:7007`**

---

## 🧪 Usando Cypress

El contenedor de Cypress está diseñado para ejecutar pruebas de dos maneras:

### 1. Modo Headless (Recomendado para Automatización)

Este modo ejecuta las pruebas en segundo plano sin abrir una interfaz gráfica. Es la forma ideal para la integración continua (CI/CD) y la que usarías en un pipeline de Jenkins. Los resultados se muestran directamente en la terminal.

```bash
sudo docker-compose exec cypress npx cypress run
```

### 2. Modo Interactivo / Gráfico (Para Desarrollo Local)

Este modo abre la interfaz de Cypress, permitiéndote ver los navegadores, seleccionar pruebas y depurar visualmente.

**Requisito previo (Solo para Linux):**
Para que la ventana del contenedor se muestre en tu escritorio, primero debes darle permiso con este comando en tu terminal (en la PC, no dentro de Docker):

```bash
xhost +local:docker
```

Una vez otorgado el permiso, puedes ejecutar el siguiente comando para abrir la interfaz:
```bash
sudo docker-compose exec cypress npx cypress open
```
**Nota:** Para otros sistemas operativos como macOS o Windows, se requieren configuraciones adicionales (como instalar XQuartz o VcXsrv).

---

## 🛠️ Comandos Útiles de Mantenimiento

* **Ver el estado de los contenedores:**
    ```bash
    sudo docker-compose ps
    ```

* **Ver los logs de un servicio en tiempo real (ej. backstage):**
    ```bash
    sudo docker-compose logs -f backstage
    ```

* **Detener y eliminar todos los contenedores:**
    ```bash
    sudo docker-compose down
    ```
