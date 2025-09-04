# 🧪 Guía Completa para Administrar Cypress en Docker Compose

Este manual detalla el propósito y uso del servicio de Cypress en nuestro entorno de Docker Compose. A diferencia de Jenkins o SonarQube, este servicio no es un servidor que corre constantemente, sino un **entorno de pruebas aislado y bajo demanda**.

---
## El Concepto Clave: Un Entorno de Pruebas Consistente

El principal objetivo de tener Cypress en un contenedor es garantizar que las pruebas se ejecuten siempre en el mismo entorno, sin importar en qué máquina se lancen.

- **Consistencia:** Todos los miembros del equipo y el propio Jenkins usarán la misma versión de Node.js, Cypress y los navegadores, eliminando el clásico problema de "en mi máquina sí funciona".
- **Aislamiento:** Las pruebas no se ven afectadas por las herramientas o configuraciones de la PC anfitriona.
- **Preparado para CI/CD:** Jenkins podrá orquestar las pruebas de la misma manera que tú lo haces localmente.

### ¿Cómo Funciona la Configuración en Docker?

Dos líneas en el `docker-compose.yml` son fundamentales para entender su comportamiento:

1.  **El Contenedor "Dormido" (`entrypoint`)**
    ```yaml
    # Mantenemos el contenedor vivo para poder ejecutar comandos en él
    entrypoint: tail -f /dev/null
    ```
    Esto hace que el contenedor se inicie y se quede en un estado de espera, listo para recibir órdenes. No ejecuta ninguna prueba al arrancar, solo "existe".

2.  **La Carpeta Compartida (`volumes`)**
    ```yaml
    volumes:
      - ./cypress:/e2e
    ```
    Esto es un **"bind mount"**. Es como crear un portal o una ventana mágica entre tu PC y el contenedor. La carpeta `servidores/cypress` de tu máquina es la misma que la carpeta `/e2e` dentro del contenedor.
    
    > **Ventaja:** Puedes usar tu editor de código favorito (como VS Code) para escribir y modificar las pruebas en la carpeta `cypress` de tu PC, y el contenedor verá los cambios instantáneamente.

---
## 🚀 Ejecutando Pruebas con Cypress

Todas las pruebas deben vivir dentro de la carpeta `servidores/cypress/` de tu proyecto.

### 1. Modo Interactivo (Para Escribir y Depurar Pruebas)

Este modo abre la interfaz gráfica de Cypress, permitiéndote ver el navegador, seleccionar qué pruebas correr e inspeccionar cada paso. Es ideal para cuando estás desarrollando un nuevo test.

**Requisito previo (Solo para Linux):**
Para que la ventana del contenedor se muestre en tu escritorio, primero debes darle permiso con este comando en tu PC:
```bash
xhost +local:docker
```

**Comando para iniciar la UI de Cypress:**
```bash
sudo docker-compose exec cypress npx cypress open
```

### 2. Modo Headless (Para Automatización y CI/CD)

Este modo ejecuta todas las pruebas en segundo plano, sin abrir una interfaz gráfica. Es más rápido y es el que usarás en tus pipelines de Jenkins. Los resultados se muestran directamente en la terminal.

**Comando para ejecutar todas las pruebas:**
```bash
sudo docker-compose exec cypress npx cypress run
```

**Ejemplos de uso avanzado:**
```bash
# Correr todas las pruebas usando el navegador Chrome dentro del contenedor
sudo docker-compose exec cypress npx cypress run --browser chrome

# Correr solo un archivo de prueba específico
sudo docker-compose exec cypress npx cypress run --spec "cypress/e2e/login.cy.js"
```

---
### 🔗 Interactuando con los Otros Contenedores

Esta es la parte más importante. Cuando tus pruebas de Cypress se ejecutan dentro de su contenedor, no pueden acceder a `http://localhost:7007` para probar Backstage, porque `localhost` dentro de un contenedor se refiere a sí mismo.

Para que un contenedor se comunique con otro, debes usar el **nombre del servicio** definido en el `docker-compose.yml` como si fuera un nombre de dominio.

- Para acceder a **Backstage**, la URL en tu test debe ser: `http://backstage:7007`
- Para acceder a **Jenkins**, la URL en tu test debe ser: `http://jenkins:8080`
- Para acceder a **SonarQube**, la URL en tu test debe ser: `http://sonarqube:9000`

*Ejemplo de un archivo de prueba `cypress/e2e/backstage.cy.js`:*
```javascript
describe('Prueba de Backstage', () => {
  it('La página principal de Backstage carga correctamente', () => {
    // Usamos el nombre del servicio 'backstage' como hostname
    cy.visit('http://backstage:7007');
    
    // Verificamos que el título esté visible
    cy.contains('My App').should('be.visible');
  });
});
```

---
## 💾 Respaldo y Datos de Cypress

A diferencia de Jenkins y la base de datos, Cypress no maneja datos persistentes que necesiten un respaldo complejo.

- **El Código de las Pruebas:** Tu "respaldo" es el propio **repositorio de Git**. Al subir la carpeta `cypress/` a Git, ya tienes todo tu trabajo versionado y seguro.
- **Artefactos (Videos y Screenshots):** Cuando las pruebas se ejecutan, Cypress genera videos y capturas de pantalla. Gracias al "bind mount" (`./cypress:/e2e`), estos archivos aparecerán automáticamente en la carpeta `cypress/` de tu PC, listos para que los revises. Generalmente, estos artefactos se ignoran en Git a través del archivo `.gitignore`.
