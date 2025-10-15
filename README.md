# Proyecto: ansible-pipeline (Teclado)

## Descripción general

La versión inicial del repositorio contenía únicamente los archivos esenciales para la ejecución local del simulador de teclado:

* index.html: estructura básica de la aplicación.

* script.js: lógica del teclado interactivo.

* Carpeta css/: estilos base y animaciones.

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