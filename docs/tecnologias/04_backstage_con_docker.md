# 🌟 Guía Completa para Administrar Backstage en Docker Compose

Este manual se enfoca en el servicio de **Backstage**, el corazón de nuestro entorno. Backstage actúa como un **portal para desarrolladores**, un único lugar donde los equipos pueden encontrar, crear y administrar todo su software.

Esta guía cubre sus funcionalidades clave, con un foco especial en las **Plantillas de Software (Software Templates)** y cómo Backstage se integra con Jenkins, SonarQube y Cypress para crear un ecosistema de desarrollo unificado.

---
## El Concepto Clave: El Portal Centralizado

El propósito de Backstage es centralizar y estandarizar las herramientas y recursos de un equipo de desarrollo. Sus pilares son:

* **Catálogo de Software:** Un inventario de todos tus servicios, sitios web, librerías y otros componentes de software. Cada componente tiene un dueño, documentación y metadatos importantes.
* **Plantillas de Software:** Permite crear nuevos componentes de software a partir de esqueletos predefinidos, asegurando que todos los nuevos proyectos sigan las mejores prácticas desde el primer día.
* **TechDocs:** Una solución de "docs-as-code" que permite a los desarrolladores escribir documentación en Markdown junto a su código, y Backstage la renderiza y la muestra en el portal.
* **Ecosistema de Plugins:** La funcionalidad de Backstage se puede expandir enormemente a través de plugins para integrar casi cualquier herramienta de desarrollo.

---
## 🏗️ Deep Dive: Plantillas de Software (Software Templates)

Esta es una de las funcionalidades más poderosas de Backstage y el foco de esta guía.

### ¿Qué son?

Son "recetas" para crear nuevos proyectos. En lugar de clonar un repositorio viejo, borrar código y reconfigurar todo a mano, un desarrollador simplemente va a Backstage, elige una plantilla (ej. "Nuevo Microservicio en Spring Boot"), llena un formulario y Backstage se encarga de todo el trabajo pesado.

### ¿Cómo Funcionan?

El proceso tiene tres partes principales:

1.  **La Definición (`template.yaml`):** Es un archivo YAML que describe la plantilla. Define los **parámetros** que se le pedirán al usuario (como el nombre del componente) y los **pasos** que el backend de Backstage ejecutará.
2.  **El Esqueleto (`skeleton/`):** Una carpeta que contiene la estructura de archivos y el código base del nuevo proyecto. Utiliza una sintaxis especial para las variables que serán reemplazadas.
3.  **Las Acciones:** Son las tareas que el backend realiza, como:
    * `fetch:template`: Descarga el esqueleto.
    * `template`: Reemplaza las variables en el esqueleto con los valores del formulario.
    * `publish:github`: Crea un nuevo repositorio en GitHub y sube el código.
    * `catalog:register`: Registra el nuevo componente en el catálogo de Backstage.

### Ejemplo Práctico

Para agregar una nueva plantilla a tu instancia de Backstage (`yavu`):

1.  **Crea la estructura de archivos** en tu proyecto, por ejemplo, en `yavu/templates/mi-nueva-plantilla/`.
    ```
    yavu/
    └── templates/
        └── mi-nueva-plantilla/
            ├── template.yaml       <-- La definición
            └── skeleton/           <-- El código base
                ├── src/
                └── catalog-info.yaml
    ```

2.  **Define el `template.yaml`**:
    ```yaml
    # yavu/templates/mi-nueva-plantilla/template.yaml
    apiVersion: scaffolder.backstage.io/v1beta3
    kind: Template
    metadata:
      name: mi-plantilla-ejemplo
      title: Plantilla de Ejemplo
      description: Una plantilla simple para crear un nuevo componente.
    spec:
      owner: team-a
      type: service
      parameters:
        - title: Detalles del Componente
          required:
            - component_id
            - owner
          properties:
            component_id:
              title: Nombre
              type: string
              description: Nombre único del componente.
            owner:
              title: Dueño
              type: string
              description: Dueño del componente, ej. team-a.
      steps:
        - id: template
          name: Generando el esqueleto del código
          action: template
          input:
            values:
              component_id: ${{ parameters.component_id }}
              owner: ${{ parameters.owner }}
        - id: register
          name: Registrando en el catálogo
          action: catalog:register
          input:
            repoContentsUrl: ${{ steps.template.output.repoContentsUrl }}
    ```

3.  **Registra la plantilla en Backstage**: Edita tu archivo `yavu/app-config.yaml` para que Backstage sepa dónde encontrar tus plantillas:
    ```yaml
    catalog:
      locations:
        - type: file
          target: ./templates/mi-nueva-plantilla/template.yaml
    ```

---
## 🔗 Integrando Backstage con Tu Ecosistema

Backstage brilla cuando se conecta con tus otras herramientas. Aquí te explico cómo se relaciona con los otros servicios en nuestro `docker-compose`.

### Backstage + SonarQube (Calidad de Código)

* **Objetivo:** Ver las métricas de calidad de código de SonarQube (bugs, vulnerabilidades, code smells, cobertura de pruebas) directamente en la página de un componente en Backstage.
* **Cómo Funciona:**
    1.  En el archivo `catalog-info.yaml` de tu componente, añades una "anotación" que lo vincula con su proyecto en SonarQube.
    2.  El plugin de SonarQube en Backstage usa esa información para consultar la API de SonarQube (`http://sonarqube:9000`) y mostrar los datos.
* **Resultado:** Una vista centralizada de la salud técnica de tus proyectos sin salir de Backstage.
    

### Backstage + Jenkins (Integración Continua)

* **Objetivo:** Ver el historial de builds de Jenkins (éxito, fallo, duración) para un componente directamente en su página de Backstage.
* **Cómo Funciona:**
    1.  De manera similar, el `catalog-info.yaml` del componente se anota con el nombre del job o pipeline de Jenkins.
    2.  El plugin de Jenkins en Backstage consulta la API de Jenkins (`http://jenkins:8081`) para obtener el historial y el estado de los builds.
* **Resultado:** Los desarrolladores pueden ver el estado del CI/CD de sus servicios sin tener que navegar a la interfaz de Jenkins.
    

### Backstage + Cypress (Resultados de Pruebas)

* **Objetivo:** Mostrar los resultados de las pruebas End-to-End de Cypress en Backstage.
* **Cómo Funciona (Flujo Típico):** La integración aquí es a través de Jenkins.
    1.  **Jenkins** ejecuta las pruebas usando el contenedor de Cypress (`docker-compose exec cypress ...`).
    2.  Jenkins archiva los resultados de las pruebas (reportes, videos, screenshots).
    3.  Un plugin en Backstage (como el de Cobertura de Pruebas o el de Artefactos de CI) se configura para leer esos resultados desde Jenkins y mostrarlos en la página del componente.
* **Resultado:** Visibilidad completa del ciclo de vida del software, desde la calidad del código y los builds hasta los resultados de las pruebas de aceptación, todo en un solo lugar.

---
## 💾 Administración y Datos de Backstage

* **Configuración:** Toda la configuración de Backstage se gestiona a través de los archivos `app-config.yaml` y `app-config.production.yaml`, los cuales deben estar en tu repositorio de Git.
* **Datos Persistentes:** La información principal de Backstage, como el inventario del Catálogo de Software, se almacena en la **base de datos PostgreSQL** que hemos configurado (en la base de datos `backstage`).
* **Respaldo (Backup):** Para respaldar Backstage, simplemente necesitas respaldar su base de datos. Puedes usar el comando de la guía de PostgreSQL:
    ```bash
    sudo docker-compose exec -T db pg_dump -U sonar -d backstage > backup_backstage_$(date +%F).sql
    ```
