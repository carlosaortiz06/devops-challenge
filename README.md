🛠️ DevOps Challenge
Este proyecto implementa una aplicación Node.js sencilla con Express, empaquetada en Docker, desplegada en Kubernetes y gestionada mediante un flujo CI/CD automatizado con GitHub Actions. Está diseñado para demostrar buenas prácticas en construcción de imágenes, despliegue continuo, manejo de secretos y salud de servicios.

📦 Dockerfile
El proyecto incluye un Dockerfile optimizado para construir y ejecutar una aplicación Node.js en un entorno liviano y listo para producción. Se utiliza la imagen oficial node:20-alpine, basada en Alpine Linux, ideal por su bajo peso y seguridad. El directorio de trabajo se establece en /app, donde se copian los archivos package.json y package-lock.json para instalar únicamente las dependencias necesarias en producción usando npm install --only=production. Luego se copia el resto del código fuente al contenedor. Se expone el puerto 3000, correspondiente al puerto en el que se ejecuta la aplicación Node.js. Finalmente, el contenedor se lanza ejecutando el archivo index.js con el comando CMD ["node", "index.js"].

📄 index.js
La aplicación está desarrollada con el framework Express.js, lo que permite levantar un servidor HTTP ligero y rápido. El servidor define una única ruta GET /health, utilizada como endpoint de salud o health check, que responde con un JSON { status: "ok" }. Esto permite a herramientas de monitoreo o balanceadores de carga verificar si la aplicación está activa y operativa.

El servidor escucha en el puerto definido por la variable de entorno PORT, y en caso de no estar definida, utiliza el puerto 3000 por defecto. Una vez iniciado, muestra en consola un mensaje indicando que el servidor está corriendo correctamente.

⚙️ Kubernetes Deployment
El archivo deployment.yml define un recurso Deployment en Kubernetes que gestiona la implementación de la aplicación Node.js dentro del espacio de nombres devops-challenge.

Detalles clave del despliegue:

Nombre del deployment: app

Namespace: devops-challenge

Réplicas: 1

Selector y etiquetas: app: node-app

Nombre del contenedor: node-app

Imagen: ghcr.io/carlosaortiz06/node-app:latest desde GitHub Container Registry

Puerto expuesto: 3000

Autenticación: Uso de imagePullSecrets (ghcr-secret)

Liveness y readiness probes: Verifican la salud del servicio accediendo al endpoint /health

Variables de entorno: Se inyecta SECRET_VALUE desde un recurso Secret

Este manifiesto permite desplegar la aplicación de forma segura, controlada y supervisada, integrándose perfectamente con prácticas modernas de DevOps.

🔐 Kubernetes Secret
El archivo secret.yaml define un recurso Secret de tipo Opaque en Kubernetes para almacenar información sensible codificada en base64.

Nombre del secreto: my-secret

Namespace: devops-challenge

Clave: secret-value, con valor base64 

Este secreto se usa para inyectar la variable SECRET_VALUE al contenedor de la aplicación, permitiendo gestionar credenciales sin exponerlas en texto plano.

🌐 Kubernetes Service
El archivo service.yaml define un recurso Service de tipo ClusterIP que expone la aplicación Node.js dentro del clúster.

Nombre del servicio: app-service

Namespace: devops-challenge

Selector: app: node-app

Puerto: 80 (externo)

targetPort: 3000 (puerto del contenedor)

Este servicio actúa como punto de entrada interno para acceder a los pods, y puede combinarse con un Ingress o cambiarse a tipo LoadBalancer o NodePort para exposición externa.

🚀 CI/CD con GitHub Actions
Este proyecto cuenta con un flujo CI/CD completamente automatizado usando GitHub Actions, que se ejecuta en cada push a la rama main.

Fases del pipeline:

Checkout del código fuente
Se clona el repositorio para ejecutar las acciones.

Login a GHCR
Se autentica en GitHub Container Registry utilizando los secretos GHCR_USERNAME y GHCR_TOKEN.

Construcción y publicación de la imagen
Se crea la imagen ghcr.io/carlosaortiz06/node-app:latest y se publica en GHCR.

Configuración del kubeconfig
Se utiliza el secreto KUBECONFIG_DATA (base64) para generar la configuración de acceso al clúster.

Despliegue en Kubernetes
Se aplican los manifiestos desde la carpeta k8s/ mediante kubectl apply.

Validación del estado del pod
Se ejecuta una espera activa de hasta 20 intentos verificando que el pod esté en estado Running.

Verificación del endpoint /health
Se realiza un kubectl port-forward para exponer temporalmente la app en localhost:8080. Se ejecuta una llamada HTTP al endpoint /health y se espera una respuesta con { status: "ok" }. Si es exitosa, se mantiene el puerto abierto por 60 segundos para validación manual.


#