# 🔎 Guía Completa para Administrar SonarQube en Docker Compose

Este documento es un manual de referencia sobre el servicio de **SonarQube**, una herramienta fundamental para el **análisis estático de código y la calidad del software**. En nuestro entorno de Docker, SonarQube se configura como un servicio persistente que trabaja en conjunto con Jenkins y PostgreSQL.

---

## El Concepto Clave: El Control de Calidad Continuo

SonarQube escanea tu código fuente y te proporciona un informe detallado sobre su calidad, detectando:

- **Bugs y vulnerabilidades**: Errores que podrían comprometer la seguridad o la estabilidad de tu aplicación.
    
- **`Code Smells`**: Patrones de código que, aunque no son errores, indican un diseño pobre y dificultan el mantenimiento.
    
- **Deuda técnica**: Una estimación del tiempo y esfuerzo necesarios para corregir los problemas encontrados.
    

Su propósito en nuestro ecosistema es proporcionar un **`feedback` automático y continuo** sobre la salud de tus proyectos, permitiéndote mantener altos estándares de calidad a lo largo del tiempo.

### ¿Cómo Funciona la Configuración en Docker?

El servicio de SonarQube depende de una base de datos para funcionar, por lo que su configuración en

`docker-compose.yml` es vital1.

YAML

```
# docker-compose.yml
services:
  sonarqube:
    image: sonarqube:lts-community
    container_name: sonarqube
    ports:
      - "9000:9000"
    environment:
      # Conexión a la base de datos
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    depends_on:
      - db
    networks:
      - servidores-net
```

- **Dependencia de la base de datos**: La línea `depends_on: - db` es crucial, ya que SonarQube no puede arrancar si la base de datos no está lista.
    
- **Variables de entorno**: Las variables `SONAR_JDBC_URL`, `SONAR_JDBC_USERNAME` y `SONAR_JDBC_PASSWORD` le indican a SonarQube cómo conectarse a la base de datos `db`.
    
- **Volúmenes persistentes**: Usamos tres volúmenes para asegurarnos de que los datos de SonarQube (proyectos analizados, configuraciones, etc.), los plugins y los registros no se pierdan cuando el contenedor se detiene o se elimina2.
    

---

## 🚀 Uso Básico de SonarQube

### 1. Acceso a la Interfaz Web

Una vez que el servicio esté activo, puedes acceder a la interfaz web de SonarQube en tu navegador en:

http://localhost:9000

Las credenciales por defecto para iniciar sesión son:

- **Usuario**: `admin`
    
- **Contraseña**: `admin`
    

Es altamente recomendable cambiar esta contraseña en el primer uso por seguridad.

### 2. Creación de un Proyecto para un Análisis

Para analizar un proyecto, necesitas crearlo en la interfaz de SonarQube y obtener un

`token` de autenticación3.

1. Inicia sesión como `admin`.
    
2. Haz clic en **"Crear un proyecto"** (o en la versión más reciente, navega a **`Projects > Create`**).
    
3. Ingresa un nombre para tu proyecto. Este nombre se usará en el futuro para identificarlo.
    
4. Sigue los pasos para generar un `token`. Este `token` se usará para autenticar el escaneo del código.
    

### 3. Integración con Jenkins

La integración de SonarQube con Jenkins es la forma en que automatizas el análisis. En un

`pipeline` de Jenkins, ejecutarás un comando para escanear tu código y enviar los resultados a SonarQube4.

Groovy

```
// Ejemplo de un paso en un Jenkinsfile
stage('Análisis de Calidad de Código') {
  steps {
    withSonarQubeEnv('tu-servidor-sonar') { // Configurado en Jenkins
      sh 'sonar-scanner -Dsonar.projectKey=nombre-de-tu-proyecto -Dsonar.sources=.'
    }
  }
}
```

- `sonar-scanner`: Es la herramienta que se usa para escanear el código5. Debes asegurarte de que el agente de Jenkins tenga acceso a ella.
    
- `sonar.projectKey`: Debe coincidir con el nombre de tu proyecto en SonarQube6.
    
- **Otras variables**: El escáner de SonarQube buscará la URL y el `token` de autenticación de tu servidor SonarQube, que normalmente se configuran como variables de entorno en Jenkins7.
    

---

## ⚙️ Configuración Avanzada y Mantenimiento

### Respaldo de Datos (Backup)

Para respaldar los datos de SonarQube, la mejor práctica es **respaldar la base de datos de PostgreSQL** a la que se conecta. La información clave de SonarQube reside en el volumen `postgres_data`.

Usa el comando de respaldo de tu guía de PostgreSQL:

Bash

```
sudo docker-compose exec -T db pg_dump -U sonar -d sonar > backup_sonar_$(date +%F).sql
```

Este comando garantiza que todos los datos analizados, los proyectos, y la configuración de SonarQube se guarden de forma segura.

### Actualización de la Versión de SonarQube

Para actualizar a una nueva versión, edita la línea `image: sonarqube:lts-community` en tu `docker-compose.yml` y cambia la etiqueta de la imagen (ej. a `sonarqube:9.9.4-community`). Después de guardar, reinicia el servicio con:

Bash

```
sudo docker-compose up -d --force-recreate sonarqube
```

Gracias al volumen persistente (`sonarqube_data`), tus datos de proyectos y configuración se mantendrán intactos.

---

### 🔗 Integración Avanzada con Jenkins y Backstage

Para un flujo de trabajo de CI/CD más profesional, es importante que SonarQube se comunique de forma bidireccional con tus otras herramientas.

- **Webhooks de SonarQube**: Los webhooks permiten que SonarQube notifique a Jenkins cuando un análisis de código ha terminado. Si un proyecto no pasa la  **"Quality Gate"** (las reglas de calidad que definas), el webhook puede notificar a Jenkins para que detenga el `pipeline` de forma automática8.
    
- **Plugin en Backstage**: Mencionas que Backstage se puede integrar con SonarQube para ver métricas de calidad de código. El **plugin de SonarQube en Backstage** se conecta directamente a la API de SonarQube para obtener métricas9. Esta conexión se define a través de una anotación en el archivo  `catalog-info.yaml` del proyecto10. De esta forma, un desarrollador puede ver el estado de calidad de su código sin salir del portal de Backstage11.
