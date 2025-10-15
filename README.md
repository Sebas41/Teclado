# Proyecto: ansible-pipeline (Teclado)

## Descripción general

La versión inicial del repositorio contenía únicamente los archivos esenciales para la ejecución local del simulador de teclado:

* index.html: estructura básica de la aplicación

* script.js: lógica del teclado interactivo

* Carpeta css/: estilos base y animaciones

Esta versión permitía ejecutar el proyecto manualmente, pero no incluía automatización, control de calidad ni despliegue continuo, por lo que solo funcionaba de forma local. En la nueva versión se implementaron mejoras de infraestructura, mantenibilidad y despliegue, manteniendo intacta la lógica principal del teclado (script.js) y los estilos (css/).

A continuación se detallan los cambios realizados:

### 1. Creación del archivo Jenkinsfile

**Objetivo:** automatizar el proceso de integración, análisis y despliegue del proyecto mediante Jenkins.
| Etapa del Pipeline | Descripción   | Justificación   |
| ------------------ | ------------- | --------------- |
| **Checkout**       | Clona el repositorio automáticamente en el agente Jenkins | Permite obtener la versión más reciente del código para cada ejecución |
| **SonarQube analysis** | Ejecuta un análisis estático con SonarQube usando credenciales seguras | Permite detectar errores, vulnerabilidades y malas prácticas en el código antes de desplegar |
| **Build static files** | Copia `index.html`, `script.js` y la carpeta `css` dentro de un directorio `dist/` | Facilita la organización y empaquetado del proyecto antes del despliegue |
| **Deploy to Nginx (via SSH)** | Automatiza la transferencia de archivos hacia un servidor remoto con Nginx mediante `sshpass` | Implementa un flujo de **CD (Continuous Deployment)**, reduciendo errores humanos y asegurando disponibilidad inmediata |

Resultado: El proyecto ahora se analiza, empaqueta y despliega automáticamente cada vez que se actualiza la rama `main`. Además, se muestran mensajes de éxito o fallo en Jenkins para un seguimiento claro del pipeline.

### 2. Creación del archivo sonar-project.properties

Objetivo: Configurar el análisis estático del código con SonarQube.

Cambios clave:

* Definición de claves del proyecto (sonar.projectKey, sonar.projectName, sonar.sources)

* Inclusión de exclusiones para directorios no relevantes

* Soporte para análisis de JavaScript y CSS.

Permite que SonarQube identifique métricas de calidad y mantenimiento del código, haciendo visible la salud técnica del proyecto.

### 3. Creación del archivo package.json

Objetivo: formalizar el proyecto como un paquete estándar compatible con Node.js.

Contenido agregado:

* Nombre, versión y descripción del proyecto

* Scripts útiles (start para ejecutar servidor local)

* Definición del repositorio

Estandariza la estructura del proyecto, permitiendo su ejecución con herramientas (como npm o yarn) y facilitando su instalación o mantenimiento por terceros.

### 4. Modificaciones en index.html

Cambios realizados:
| Cambio                                  | Descripción                                                       | Justificación                                                                                                  |
| --------------------------------------- | ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Título actualizado**                  | `Document` → `Keyboard Training`                                  | Da un nombre descriptivo y profesional a la aplicación.                                                        |
| **Etiqueta `<base href="/keyboard/">`** | Define una ruta base para los recursos.                           | Permite servir la aplicación desde una subruta en el servidor (por ejemplo: `http://130.131.27.80/keyboard/`). |
| **Corrección del id del carácter `\`**  | Se reemplazó `id="\"` por `id="BACKSLASH"`.                       | Evita conflictos con caracteres especiales en atributos HTML y mejora la compatibilidad del DOM.               |
| **Rutas relativas en CSS y JS**         | Se ajustaron a `css/style.css` y `script.js` bajo el `base href`. | Garantiza que los archivos se carguen correctamente durante el despliegue en Nginx.                            |

Se adaptó el archivo para funcionar correctamente en entornos de producción y servidores web reales, sin modificar la funcionalidad original.

### 5. Código JavaScript y estilos sin cambios

El archivo script.js y los estilos en css/ permanecieron idénticos, dado que su funcionalidad y presentación ya eran correctas. El enfoque principal fue la automatización del flujo DevOps, no la modificación del comportamiento visual o lógico.

## Despliegue actual

La aplicación se encuentra disponible en:
http://130.131.27.80/keyboard/